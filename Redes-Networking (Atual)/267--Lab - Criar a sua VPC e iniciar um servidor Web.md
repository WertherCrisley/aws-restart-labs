# 🚀 AWS re/Start — Criando uma VPC e Iniciando um Servidor Web

Laboratório prático do programa **AWS re/Start** em que construí uma **VPC personalizada** com **sub-redes públicas e privadas em duas Zonas de Disponibilidade**, configurei um **grupo de segurança** e iniciei uma **instância EC2** rodando um **servidor web Apache**, tudo automatizado com *user data*.

![AWS](https://img.shields.io/badge/AWS-VPC-FF9900?logo=amazonaws&logoColor=white)
![EC2](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white)
![Apache](https://img.shields.io/badge/Apache-httpd-D22128?logo=apache&logoColor=white)
![Status](https://img.shields.io/badge/status-concluído-success)

---

## 🎯 Objetivos do laboratório

- Criar uma **VPC**
- Criar **sub-redes**
- Configurar um **grupo de segurança**
- Iniciar uma **instância EC2** dentro da nova VPC

## 📩 O cenário

Montar uma rede personalizada para um cliente de grande porte, com uma VPC, sub-redes públicas e privadas, um grupo de segurança e uma instância EC2 executando um servidor web.

---

## 🏗️ A arquitetura final

```
Lab VPC (10.0.0.0/16)
│
├── Zona de Disponibilidade 1
│   ├── Public Subnet 1   (10.0.0.0/24)  → Public Route Table  → Internet Gateway
│   └── Private Subnet 1  (10.0.1.0/24)  → Private Route Table → NAT Gateway
│
└── Zona de Disponibilidade 2
    ├── Public Subnet 2   (10.0.2.0/24)  → Public Route Table
    └── Private Subnet 2  (10.0.3.0/24)  → Private Route Table
```

| Componente | Função |
|---|---|
| **VPC** | A rede privada virtual isolada |
| **Gateway de internet (IGW)** | Permite a comunicação entre a VPC e a internet |
| **Sub-redes públicas** | Têm rota para o gateway de internet |
| **Sub-redes privadas** | Não têm rota direta para o gateway de internet |
| **Gateway NAT** | Dá acesso de **saída** à internet para as sub-redes privadas |
| **Tabelas de rotas** | Direcionam o tráfego de cada sub-rede |
| **Grupo de segurança** | Firewall virtual da instância |

---

## 🛠️ O que eu fiz

### Tarefa 1: Criar a VPC com o assistente
Em **VPC → Criar VPC**, usei a opção **"VPC e muito mais"**:

| Opção | Valor |
|---|---|
| Geração automática de nome | Desmarcada |
| CIDR IPv4 | `10.0.0.0/16` |
| IPv6 | Nenhum |
| Tenancy | Padrão |
| Zonas de disponibilidade | 1 |
| Sub-redes públicas | 1 (`10.0.0.0/24`) |
| Sub-redes privadas | 1 (`10.0.1.0/24`) |
| Gateways NAT | Em 1 AZ |
| Endpoints da VPC | Nenhum |

Depois, dei nomes aos recursos no painel de visualização: `Lab VPC`, `Public Subnet 1`, `Private Subnet 1`, `Public Route Table` e `Private Route Table`.

> O assistente cria a VPC, o **gateway de internet**, as sub-redes, as tabelas de rotas e o **gateway NAT** de uma vez.

### Tarefa 2: Criar sub-redes adicionais (segunda AZ)
Para ter **alta disponibilidade**, criei mais duas sub-redes na `Lab VPC`:

| Sub-rede | CIDR |
|---|---|
| `Public Subnet 2` | `10.0.2.0/24` |
| `Private Subnet 2` | `10.0.3.0/24` |

> 💡 O lab deixa a zona de disponibilidade em "sem preferência", mas, para realmente ter alta disponibilidade, a segunda sub-rede precisa ficar em uma **AZ diferente** da primeira. Vale conferir isso na coluna "Zona de disponibilidade".

### Tarefa 3: Associar as sub-redes às tabelas de rotas

| Tabela de rotas | Sub-rede associada |
|---|---|
| `Public Route Table` | `Public Subnet 2` |
| `Private Route Table` | `Private Subnet 2` |

(As sub-redes 1 já vieram associadas pelo assistente.)

### Tarefa 4: Criar o grupo de segurança

| Campo | Valor |
|---|---|
| Nome | `Web Security Group` |
| Descrição | `Enable HTTP access` |
| VPC | `Lab VPC` |
| Regra de entrada | **HTTP**, origem **Anywhere-IPv4** (`Permit web requests`) |

### Tarefa 5: Iniciar o servidor web

| Opção | Valor |
|---|---|
| Nome | `Web Server 1` |
| AMI | Amazon Linux 2 |
| Tipo | `t3.micro` |
| Par de chaves | `vockey` |
| VPC / Sub-rede | `Lab VPC` / `Public Subnet 1` |
| IP público automático | **Habilitado** |
| Grupo de segurança | `Web Security Group` |

**Dados do usuário (*user data*):** o script que roda no primeiro boot da instância:

```bash
#!/bin/bash
# Instala o Apache, MySQL (cliente) e PHP
yum install -y httpd mysql php
# Baixa os arquivos da aplicação do lab e descompacta na pasta do servidor web
wget <URL-do-arquivo-lab-app.zip>
unzip lab-app.zip -d /var/www/html/
# Liga o servidor web e o inicia
chkconfig httpd on
service httpd start
```

Depois de executar, aguardei o status **2/2 verificações aprovadas**, copiei o **DNS IPv4 público** da instância e abri no navegador. A página da aplicação carregou, confirmando que o servidor web estava funcionando. ✅

---

## 🧠 O que eu aprendi

**Sub-rede pública x privada:** a diferença está na **tabela de rotas**.
- Se o tráfego da sub-rede é roteado para o **gateway de internet**, ela é **pública**
- Se não tem essa rota, é **privada**

**Gateway NAT:** permite que instâncias em sub-redes **privadas** acessem a internet (por exemplo, para baixar atualizações) **sem ficarem expostas** a conexões vindas de fora.

**Cada sub-rede vive em uma única AZ.** Para ter alta disponibilidade, o ideal é distribuir os recursos em **sub-redes de AZs diferentes**.

**Associação de sub-redes às tabelas de rotas:** criar a sub-rede não basta, é preciso associá-la à tabela de rotas certa.

**Automação com user data:** em vez de entrar na instância e instalar tudo na mão, o script já deixa o servidor pronto no primeiro boot, e o resultado é **repetível**.

**Menor privilégio no firewall:** o grupo de segurança libera **apenas HTTP**, a porta que o servidor web precisa.

> 💰 **Atenção a custos:** o **gateway NAT** gera cobrança por hora e por dados processados. Em um ambiente real, vale remover recursos que não estão em uso. No lab, tudo é encerrado em **End Lab**.

---

## ✅ Boas práticas que levo daqui

- **Planejar os blocos CIDR** antes de criar a rede, deixando espaço para crescimento
- **Separar recursos públicos e privados** em sub-redes diferentes
- **Distribuir sub-redes em mais de uma AZ** para alta disponibilidade
- **Dar nomes claros** aos recursos (`Public Subnet 1`, `Private Route Table`...)
- **Liberar só as portas necessárias** no grupo de segurança
- **Automatizar a configuração** das instâncias com *user data*
- **Encerrar o laboratório** ao final para liberar os recursos

---


## 📚 Próximos passos

- [ ] Colocar um **Application Load Balancer** distribuindo tráfego entre duas instâncias em AZs diferentes
- [ ] Usar o **Auto Scaling** para ajustar o número de instâncias
- [ ] Colocar um **banco de dados** em uma sub-rede privada
- [ ] Explorar **NACLs** e regras mais restritivas de grupo de segurança
- [ ] Documentar os próximos labs do re/Start neste repositório

---

## 👤 Autor

**Werther Crisley**
🔗 [GitHub](https://github.com/WertherCrisley)