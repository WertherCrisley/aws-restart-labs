# 🌐 AWS re/Start — Endereços IP Públicos e Privados

Laboratório do programa **AWS re/Start** em que atuei como **engenheiro de suporte de nuvem** para resolver o problema de rede de uma cliente: duas instâncias EC2 na mesma VPC, mas só uma delas acessava a internet.

![AWS](https://img.shields.io/badge/AWS-VPC-FF9900?logo=amazonaws&logoColor=white)
![EC2](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white)
![Status](https://img.shields.io/badge/status-concluído-success)

---

## 📩 O cenário

A cliente (Jess) tem uma **VPC com CIDR `10.0.0.0/16`** e duas instâncias EC2, a **A** e a **B**, na mesma configuração:

- A **instância A não acessa a internet**
- A **instância B acessa a internet**
- Ela também perguntou se poderia usar um **intervalo de IP público** (como `12.0.0.0/16`) em uma nova VPC

> Neste lab, a arquitetura (gateway de internet, rotas e sub-redes) já estava correta. O foco era **investigar e explicar**, não construir.

---

## 🔍 Investigação

1. No console da AWS, fui em **EC2 → Instâncias** e abri a guia **Redes** de cada instância
2. Comparei os endereços **IPv4 público** e **IPv4 privado** das duas
3. Tentei me conectar às duas por **SSH**

### O que descobri

| | Instância A | Instância B |
|---|---|---|
| IP privado | ✅ Tem | ✅ Tem |
| IP público | ❌ **Não tem** | ✅ **Tem** |
| Acesso à internet | ❌ | ✅ |
| SSH de fora da VPC | ❌ Não conecta | ✅ Conecta |

---

## ✅ Conclusão: o problema era o IP público

A instância A tinha **apenas um IP privado**. IPs privados só funcionam **dentro da VPC** e não são alcançáveis pela internet. Como a instância B tinha um **IP público**, ela conseguia sair para a internet e receber conexões de fora.

**Solução:** dar à instância A um **endereço IP público** (por exemplo, com um **Elastic IP**).

---

## 🧠 O que eu aprendi

### IP público x IP privado

| | IP privado | IP público |
|---|---|---|
| **Funciona** | Dentro da VPC/rede interna | Na internet |
| **Acessível de fora?** | ❌ Não | ✅ Sim |
| **Exemplo** | `10.0.x.x` | Atribuído pela AWS à instância |
| **Uso típico** | Comunicação entre recursos internos | Acesso externo e saída para a internet |

### Por que usar intervalos privados na VPC?
Para a pergunta da cliente sobre usar `12.0.0.0/16`, a recomendação é **não usar**. Para redes internas, a norma (**RFC 1918**) reserva estes intervalos privados:

| Intervalo | CIDR |
|---|---|
| 10.0.0.0 – 10.255.255.255 | `10.0.0.0/8` |
| 172.16.0.0 – 172.31.255.255 | `172.16.0.0/12` |
| 192.168.0.0 – 192.168.255.255 | `192.168.0.0/16` |

**Riscos de usar um intervalo público em uma VPC:**
- Esses endereços **pertencem a outras organizações** na internet
- O tráfego destinado a eles pode ser tratado como **interno à VPC**, e a instância **não consegue alcançar os servidores reais** desse endereço
- Pode causar **conflitos e problemas** ao receber respostas de outros recursos não relacionados

> **Lição:** `10.0.0.0/16` (o CIDR da cliente) já está dentro de um intervalo privado, então está **correto**.

### Solução de problemas em camadas
Aprendi a investigar **de baixo para cima**, começando pela instância e subindo pelas camadas (como no modelo OSI): instância → grupo de segurança → rotas e sub-rede → gateway de internet. Aqui, o problema estava logo na base, no **endereço da instância**.

---

## 📝 Resposta para a cliente (resumo)

> Olá, Jess! A instância A não acessa a internet porque possui apenas um **IP privado**, que só funciona dentro da VPC. A instância B tem também um **IP público**, por isso consegue sair para a internet e receber conexões externas. Para resolver, associe um IP público (como um Elastic IP) à instância A.
>
> Sobre a nova VPC, **não recomendamos** usar um intervalo público como `12.0.0.0/16`, pois esses endereços pertencem a outras redes na internet e podem causar conflitos de roteamento. Use um intervalo **privado** (RFC 1918), como o `10.0.0.0/16` que você já usa.

---

## ✅ Boas práticas que levo daqui

- **Usar intervalos privados (RFC 1918)** para VPCs
- **Conferir o IP público** de uma instância quando ela precisa ser alcançada de fora
- **Investigar em camadas**, de baixo para cima
- **Explicar o problema de forma clara** para quem não é da área técnica
- **Encerrar o laboratório** ao final para liberar os recursos

---

## 👤 Autor

**Werther Crisley**
🔗 [GitHub](https://github.com/WertherCrisley)