# 🌍 AWS re/Start — Criando Recursos de Rede em uma VPC

Laboratório do programa **AWS re/Start** em que atuei como **engenheiro de suporte de nuvem** e montei, do zero, uma **rede roteável para a internet** em uma VPC: **sub-rede, tabela de rotas, gateway de internet, NACL e grupo de segurança**, para depois iniciar uma instância EC2 e testar a conectividade com o `ping`.

![AWS](https://img.shields.io/badge/AWS-VPC-FF9900?logo=amazonaws&logoColor=white)
![EC2](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white)
![Status](https://img.shields.io/badge/status-concluído-success)

---

## 📩 O cenário

O cliente (Brock), dono de uma startup, montou uma VPC mas **não conseguia nem fazer `ping` para fora dela**. Ele pediu ajuda para configurar a rede e ter conectividade com a internet.

| Item | Valor mantido do cliente |
|---|---|
| CIDR da VPC | `192.168.0.0/18` |
| CIDR da sub-rede pública | `192.168.1.0/26` |
| Arquitetura | VPC + gateway de internet + sub-rede pública + grupo de segurança + instância EC2 |

**Objetivo final:** conseguir fazer `ping` da instância EC2 para fora da VPC, provando que a rede está funcionando.

---

## 🧱 Os componentes de uma rede roteável

Para a instância chegar na internet, **todas** estas peças precisam estar conectadas:

| Componente | Função |
|---|---|
| **VPC** | A rede privada virtual: o "data center na nuvem" |
| **Sub-rede** | Um intervalo de IPs dentro da VPC onde ficam os recursos |
| **Gateway de internet (IGW)** | A "porta" da VPC para a internet; faz NAT para instâncias com IP público |
| **Tabela de rotas** | Define para onde o tráfego da sub-rede vai |
| **NACL** | Firewall no nível da **sub-rede** (stateless) |
| **Grupo de segurança** | Firewall no nível da **instância** (stateful) |
| **IP público** | Necessário para a instância se comunicar fora da VPC |

---

## 🛠️ O que eu construí, na ordem

Segui uma abordagem **de cima para baixo**, começando pela VPC:

| # | Recurso | Configuração |
|---|---|---|
| 1 | **VPC** | Nome `Test VPC`, CIDR `192.168.0.0/18` |
| 2 | **Sub-rede** | `Sub-rede pública`, CIDR `192.168.1.0/26`, dentro da Test VPC |
| 3 | **Tabela de rotas** | `Tabela de rotas públicas`, na Test VPC |
| 4 | **Gateway de internet** | `IGW test VPC`, criado e **anexado à VPC** (*Ações → Attach to VPC*) |
| 5 | **Rota para a internet** | Na tabela de rotas: destino `0.0.0.0/0`, alvo = o IGW |
| 6 | **Associação** | Associei a sub-rede pública à tabela de rotas públicas |
| 7 | **NACL** | `Public Subnet NACL`, com regra 100 permitindo todo o tráfego na **entrada** e na **saída** |
| 8 | **Grupo de segurança** | `public security group`, na Test VPC (regras abaixo) |
| 9 | **Instância EC2** | Na sub-rede pública, com IP público automático e o grupo de segurança criado |

### Regras do grupo de segurança

| Direção | Tipo | Origem/Destino |
|---|---|---|
| Entrada | SSH, HTTP e HTTPS | Qualquer lugar |
| Saída | Todo o tráfego | Qualquer lugar |

### A instância EC2

| Opção | Valor |
|---|---|
| AMI | Amazon Linux 2023 |
| Tipo | `t3.micro` |
| Par de chaves | `vockey` |
| VPC / Sub-rede | Test VPC / Sub-rede pública |
| IP público automático | **Habilitado** |
| Grupo de segurança | `public security group` |

---

## 🧠 O que eu aprendi

**A rota `0.0.0.0/0` é o que "liga" a sub-rede à internet.** Ela significa "qualquer destino que não seja local" e aponta para o IGW. Sem ela, a sub-rede não sai da VPC, mesmo com o IGW criado. A rota **local** (dentro da VPC) já vem pronta.

**Criar o IGW não basta: é preciso anexá-lo à VPC** e depois criar a rota e associar a sub-rede à tabela.

**Security group x NACL:**

| | Grupo de segurança | NACL |
|---|---|---|
| Nível | **Instância** | **Sub-rede** |
| Estado | **Stateful**: a resposta de uma conexão permitida volta automaticamente | **Stateless**: entrada e saída precisam ser liberadas separadamente |
| Por isso | Basta liberar o tráfego de saída para o `ping` funcionar | Precisei criar regras **nas duas direções** |

**Uma NACL nova nega tudo até receber regras.** Por isso criei a regra 100 liberando todo o tráfego na entrada e na saída. O asterisco (`*`) na lista indica a regra padrão que **nega** o que não combinar com nenhuma outra.

**Convenção de nomes ajuda:** `Test VPC`, `Sub-rede pública`, `Tabela de rotas públicas`... Em redes maiores, nomes consistentes evitam confusão sobre qual recurso fica onde.

**Dá para solucionar problemas em ordem:** VPC → sub-rede → rota → IGW → NACL → grupo de segurança → instância. Se o `ping` falhar, é só percorrer essa lista.

---

## ✅ Teste final

O lab se conclui quando a instância consegue fazer `ping` para a internet. Depois de conectar na instância por SSH, o teste é algo como:

```bash
ping -c 4 8.8.8.8
```

Se houver respostas, a VPC tem conectividade de rede. ✅

> **Por que o `ping` de dentro para fora funciona** mesmo sem regra de entrada para ICMP? Porque o grupo de segurança é **stateful**: a resposta a uma conexão iniciada pela instância é permitida automaticamente.

---

## ✅ Boas práticas que levo daqui

- **Construir a rede em uma ordem lógica** para não esquecer nenhuma peça
- **Nomear os recursos de forma consistente**
- **Liberar só o necessário** nos grupos de segurança, em vez de permitir tudo
- **Lembrar que a NACL é stateless** e precisa de regras de ida e de volta
- **Solucionar problemas em camadas**, do recurso mais interno para o mais externo
- **Encerrar o laboratório** ao final para liberar os recursos

---


## 👤 Autor

**Werther Crisley**
🔗 [GitHub](https://github.com/WertherCrisley)