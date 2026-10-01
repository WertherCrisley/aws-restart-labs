# ☁️ AWS re/Start — Introduction to Amazon EC2

Laboratório prático do programa **AWS re/Start** em que lancei, monitorei, protegi, redimensionei e encerrei uma instância **Amazon EC2** rodando um servidor web Apache.

![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white)
![Linux](https://img.shields.io/badge/Amazon_Linux-2023-232F3E?logo=linux&logoColor=white)
![Apache](https://img.shields.io/badge/Apache-httpd-D22128?logo=apache&logoColor=white)
![Status](https://img.shields.io/badge/status-concluído-success)

---

## 🎯 Objetivo do laboratório

Entender, na prática, o ciclo de vida de um servidor na nuvem: do lançamento ao encerramento, passando por rede, monitoramento, escalabilidade e proteção contra erros humanos.

## 🧩 O que eu fiz

1. **Lancei** uma instância EC2 (`t3.micro`, Amazon Linux 2023) dentro de uma VPC dedicada
2. **Automatizei** a instalação do Apache com um script de *user data*
3. **Monitorei** a instância com status checks, métricas do CloudWatch e captura de tela do console
4. **Configurei** o security group para liberar tráfego HTTP (porta 80)
5. **Redimensionei** a instância (`t3.micro` → `t3.small`) e o disco (8 GiB → 10 GiB)
6. **Testei** a proteção contra encerramento e depois **terminei** a instância

---

## 🧠 O que eu aprendi

### 1. Como funciona o EC2 e a ideia de "pagar pelo que usa"
O EC2 entrega servidores virtuais que sobem em minutos e podem ser aumentados ou diminuídos conforme a necessidade. Em vez de comprar hardware, eu pago só pela capacidade que uso, e isso muda bastante a forma de pensar em infraestrutura.

### 2. AMI, tipo de instância e volume EBS
- **AMI**: o "molde" da instância (sistema operacional, permissões de execução e volumes). Usei a Amazon Linux 2023.
- **Tipo de instância**: combinação de CPU, memória e rede. A `t3.micro` tem 2 vCPUs e 1 GiB de RAM.
- **EBS**: o disco virtual conectado pela rede, onde fica o volume raiz.

### 3. Automação com *user data*
Em vez de entrar na máquina e instalar tudo na mão, passei um script que roda no primeiro boot:

```bash
#!/bin/bash
yum -y install httpd
systemctl enable httpd
systemctl start httpd
echo '<html><h1>Hello From Your Web Server!</h1></html>' > /var/www/html/index.html
```

Esse script instala o Apache, configura para iniciar com o sistema, sobe o serviço e cria a página inicial. Entendi que **infraestrutura pode ser reproduzível**: a mesma instância pode ser recriada com o mesmo resultado.

### 4. Security group é um firewall (e ele bloqueia tudo por padrão)
Depois de subir o servidor, **não consegui acessá-lo pelo navegador**. O motivo: o security group não tinha regra de entrada para a **porta 80**. Ao adicionar uma regra `HTTP` com origem `IPv4 em qualquer lugar`, a página "Hello From Your Web Server!" apareceu.

> **Lição:** na AWS o padrão é *negar*. Só trafega o que eu libero explicitamente. Também removi o acesso SSH de propósito, o que reduz a superfície de ataque.

*(Em um ambiente real, eu restringiria a origem a IPs específicos em vez de liberar para qualquer lugar, sempre que possível.)*

### 5. Monitoramento
- **Status checks** (acessibilidade do sistema e da instância) mostram se a AWS detectou problemas de hardware ou software
- **CloudWatch** coleta métricas: monitoramento básico a cada 5 minutos e detalhado a cada 1 minuto
- A **captura de tela da instância** ajuda a diagnosticar quando não dá para acessar via SSH/RDP

### 6. Redimensionamento (escalar para cima)
Para trocar o tipo da instância, é preciso **interromper** primeiro. Depois alterei para `t3.small` (o dobro de memória) e aumentei o volume EBS de 8 para 10 GiB.

> **Detalhe de custo:** instância interrompida não cobra computação, mas o **armazenamento EBS continua sendo cobrado**.

### 7. Proteção contra encerramento
Com a *termination protection* ativa, a tentativa de terminar a instância **falhou com erro**, o que evita exclusões acidentais. Para terminar de verdade, tive que desativar a proteção primeiro. Instância terminada não pode ser reiniciada, e o volume raiz é excluído por padrão.

---

## 🛠️ Conceitos e serviços praticados

| Conceito | O que significa |
|---|---|
| EC2 | Servidores virtuais na nuvem |
| AMI | Template para iniciar instâncias |
| Tipos de instância | CPU, memória e rede (`t3.micro`, `t3.small`) |
| EBS | Disco virtual da instância |
| VPC | Rede privada virtual onde a instância roda |
| Security Group | Firewall virtual de entrada e saída |
| User data | Script de configuração no primeiro boot |
| CloudWatch | Métricas e monitoramento |
| Termination protection | Proteção contra encerramento acidental |

## ✅ Boas práticas que levo daqui

- Dar **nomes e tags** claros aos recursos (`Web Server`)
- Aplicar **menor privilégio** na rede (liberar só o necessário)
- **Automatizar** configurações em vez de fazer tudo manualmente
- Ativar **proteção contra encerramento** em recursos importantes
- Ficar atento a **custos** de recursos parados (EBS)
- **Encerrar** os recursos ao final para não gerar cobrança

---

## 📸 Evidências

<!-- Adicione aqui seus prints do lab. Exemplos:
![Instância em execução](./img/instancia-running.png)
![Página do servidor web](./img/hello-web-server.png)
![Erro da proteção contra encerramento](./img/termination-protection.png)
-->

---

## 📚 Próximos passos

- [ ] Praticar conexão segura com **par de chaves / Session Manager**
- [ ] Explorar **Elastic IP** e **Load Balancer**
- [ ] Estudar **Auto Scaling**
- [ ] Documentar os próximos labs do re/Start neste repositório

---

## 👤 Autor

**Werther Crisley**
🔗 [GitHub](https://github.com/WertherCrisley)