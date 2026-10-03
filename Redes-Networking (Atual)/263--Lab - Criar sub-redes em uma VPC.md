# 🏗️ AWS re/Start — Criando uma VPC, Sub-redes e Alocando Endereços IP

Laboratório do programa **AWS re/Start** em que atuei como **engenheiro de suporte de nuvem** e montei a **primeira VPC** de um cliente iniciante na AWS, calculando os blocos **CIDR** para atender à quantidade de endereços IP que ele precisava.

![AWS](https://img.shields.io/badge/AWS-VPC-FF9900?logo=amazonaws&logoColor=white)
![Status](https://img.shields.io/badge/status-concluído-success)

---

## 📩 O cenário

O cliente (Paulo), dono de uma startup, pediu ajuda para montar uma **VPC** com estes requisitos:

| Requisito | Detalhe |
|---|---|
| Endereços IP privados na VPC | **Cerca de 15.000** |
| Faixa do CIDR da VPC | Começando com **192.x.x.x** (ele não lembrava qual era o intervalo privado) |
| Sub-rede pública | **Pelo menos 50** endereços IP |
| Arquitetura | VPC + gateway de internet + **1 sub-rede pública** |

---

## 🧮 Calculando o CIDR

O número depois da barra (`/`) define quantos bits são da rede. Quanto **menor** o número, **mais** endereços:

> Total de endereços = **2^(32 − prefixo)**

### VPC: precisa de ~15.000 endereços

| CIDR | Endereços | Atende? |
|---|---|---|
| `/19` | 8.192 | ❌ Pouco |
| **`/18`** | **16.384** | ✅ **O menor que atende** |
| `/17` | 32.768 | ✅ Mas desperdiça espaço |

➡️ **Escolha: `192.168.0.0/18`** (de 192.168.0.0 até 192.168.63.255)

### Sub-rede pública: precisa de pelo menos 50 endereços

A AWS **reserva 5 endereços em cada sub-rede**, então a conta é: endereços − 5.

| CIDR | Endereços | Utilizáveis (−5) | Atende? |
|---|---|---|---|
| `/27` | 32 | 27 | ❌ Pouco |
| **`/26`** | **64** | **59** | ✅ **O menor que atende** |

➡️ **Escolha: `192.168.1.0/26`** (dentro do intervalo da VPC)

### Por que 192.168.x.x?
Segundo a **RFC 1918**, o intervalo privado que começa com 192 é o **`192.168.0.0/16`**. Por isso o cliente pode usar `192.168.0.0/18` sem problemas.

---

## 🛠️ Montando a VPC no console

1. No console da AWS, pesquisei **VPC** e cliquei em **Criar VPC**
2. Configurei:

| Opção | Valor |
|---|---|
| Recursos a serem criados | **VPC e muito mais** |
| Geração automática de etiqueta de nome | `First` |
| CIDR IPv4 | `192.168.0.0/18` |
| Bloco CIDR IPv6 | Nenhum |
| Tenancy | Padrão |
| Zonas de disponibilidade (AZs) | 1 |
| Sub-redes públicas | 1 |
| Sub-redes privadas | 0 |
| CIDR da sub-rede pública (us-west-2a) | `192.168.1.0/26` |
| Endpoints da VPC | Nenhum |

3. Cliquei em **Criar VPC**, vi a mensagem de sucesso e abri **Visualizar VPC** para conferir os detalhes da `First-vpc`

> O assistente **"VPC e muito mais"** costuma criar, além da VPC e da sub-rede, os recursos de rede que a sub-rede pública precisa, como o **gateway de internet** e a **tabela de rotas**.

---

## 🧠 O que eu aprendi

**O que é uma VPC?** Um "data center na nuvem", **isolado logicamente** de outras redes, onde se iniciam recursos da AWS.

**Por que existem sub-redes públicas e privadas?**
- **Públicas:** para recursos que precisam ser **acessados pela internet**. Usam IP público e um **gateway de internet**
- **Privadas:** mantêm os recursos **fora do alcance da internet**. Para que eles saiam para a internet, precisam de um **NAT gateway**

**Por que usar IPs privados dentro da VPC?** Eles **não são acessíveis pela internet**, o que mantém os recursos e a comunicação entre eles **protegidos dentro da VPC**.

**Principais lições:**
- Calcular o CIDR a partir da **necessidade real** de endereços, sem exagerar nem faltar
- A AWS **reserva 5 IPs por sub-rede**, o que muda a conta
- A sub-rede precisa estar **dentro do bloco CIDR** da VPC
- Usar sempre os intervalos **privados da RFC 1918**

---

## 📝 Passo a passo curto para o cliente

> Olá, Paulo! Para o seu cenário, use **`192.168.0.0/18`** na VPC (16.384 endereços, atendendo os ~15.000 que você precisa) e **`192.168.1.0/26`** na sub-rede pública (59 endereços utilizáveis, mais que os 50 necessários).
>
> No console: **VPC → Criar VPC → "VPC e muito mais"**, informe os CIDRs acima, escolha **1 AZ**, **1 sub-rede pública** e **0 privadas**, e clique em **Criar VPC**.
> O intervalo `192.168.0.0/16` é privado (RFC 1918), então você pode usá-lo com segurança.

---

## ✅ Boas práticas que levo daqui

- **Dimensionar o CIDR** pela necessidade, deixando uma margem para crescimento
- **Conferir a sub-rede** dentro do bloco da VPC
- **Lembrar dos 5 IPs reservados** por sub-rede
- **Usar a calculadora de sub-redes** e a RFC 1918 como apoio
- **Explicar o passo a passo** de forma simples para quem está começando
- **Encerrar o laboratório** ao final para liberar os recursos

---


## 👤 Autor

**Werther Crisley**
🔗 [GitHub](https://github.com/WertherCrisley)