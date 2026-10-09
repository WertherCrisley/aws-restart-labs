# 📊 AWS re/Start — Monitorando uma Instância EC2 (CloudWatch e SNS)

Laboratório prático do programa **AWS re/Start** em que montei um sistema simples de **monitoramento e alerta**: criei um **tópico do Amazon SNS**, um **alarme do Amazon CloudWatch** que dispara quando a **CPU** de uma instância EC2 passa de **60%**, forcei um pico de CPU com um **teste de estresse** e confirmei que o **e-mail de alerta chegou**. Também criei um **painel (dashboard)** para acompanhar a métrica.

![AWS](https://img.shields.io/badge/AWS-CloudWatch-FF4F8B?logo=amazoncloudwatch&logoColor=white)
![SNS](https://img.shields.io/badge/AWS-SNS-FF9900?logo=amazonaws&logoColor=white)
![EC2](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white)
![Status](https://img.shields.io/badge/status-conclu%C3%ADdo-success)

---

## 🎯 Objetivos do laboratório

- Criar uma **notificação do Amazon SNS**
- Criar um **alarme do CloudWatch**
- **Testar por estresse** uma instância EC2
- Confirmar que um **e-mail do SNS foi enviado**
- Criar um **painel do CloudWatch**

## 🧠 Registro e monitoramento

| | O que é | Para que serve |
|---|---|---|
| **Registro (*logging*)** | Gravar eventos em arquivos de log | Visibilidade de baixo nível sobre o que o sistema fez; ajuda a achar sinais de alerta |
| **Monitoramento** | Coletar e analisar dados continuamente | Garantir desempenho e segurança, e detectar acessos ou comportamentos fora do normal |

**A ideia do lab:** um **pico de CPU** pode ter várias causas, e uma delas é **malware** (por exemplo, um minerador de criptomoedas rodando na instância sem autorização). O teste de estresse **simula** esse cenário, e o alarme mostra como seria detectá-lo.

---

## 🏗️ Como as peças se conectam

```
Instância EC2 ──(métrica CPUUtilization)──► CloudWatch ──(> 60%)──► Alarme
                                                                       │
                                                                       ▼
                                                      Tópico SNS (MyCwAlarm) ──► E-mail
```

| Serviço | Papel |
|---|---|
| **Amazon CloudWatch** | Coleta métricas, logs e eventos; mede a CPU e avalia o alarme |
| **Amazon SNS** | Serviço de mensagens que **envia a notificação** (aqui, por e-mail) |
| **Alarme** | Compara a métrica com um limite e **age** quando ele é ultrapassado |

---

## 🧩 O que eu fiz

### Tarefa 1: Configurar o Amazon SNS
1. Em **SNS → Topics → Create topic**:

| Campo | Valor |
|---|---|
| Tipo | **Standard** |
| Nome | `MyCwAlarm` |

2. Na aba **Subscriptions**, criei uma assinatura:

| Campo | Valor |
|---|---|
| Protocolo | **Email** |
| Endpoint | Meu endereço de e-mail |

3. O status ficou **Pending confirmation**. Abri o e-mail da AWS e cliquei em **Confirm subscription**
4. De volta ao console, o status passou para **Confirmed** ✅

> 💡 Sem confirmar a assinatura, o SNS **não entrega** as mensagens.

### Tarefa 2: Criar o alarme do CloudWatch
Primeiro olhei a métrica em **CloudWatch → Metrics → EC2 → Per-Instance Metrics → `CPUUtilization`** da instância **Stress Test**. Com a instância parada, ela ficava perto de **0%**.

Depois criei o alarme em **Alarms → Create alarm**:

| Configuração | Valor |
|---|---|
| Métrica | `CPUUtilization` (instância *Stress Test*) |
| Estatística | **Average** |
| Período | **1 minuto** |
| Tipo de limite | Estático |
| Condição | **Maior que** (`>`) **60** |
| Gatilho de estado | **In alarm** |
| Ação | Enviar notificação para o tópico **`MyCwAlarm`** |
| Nome | `LabCPUUtilizationAlarm` |
| Descrição | `CloudWatch alarm for Stress Test EC2 instance CPUUtilization` |

### Tarefa 3: Testar o alarme
1. Conectei na instância **Stress Test** pelo **Session Manager** (Systems Manager), sem SSH
2. Executei o teste de estresse:

```bash
sudo stress --cpu 10 -v --timeout 400s
```

| Parte | Significado |
|---|---|
| `stress` | Ferramenta que **gera carga** no sistema |
| `--cpu 10` | Cria **10 processos** pesando na CPU |
| `-v` | Modo detalhado (*verbose*) |
| `--timeout 400s` | Roda por **400 segundos** e para sozinho |

3. Abri um **segundo terminal** na mesma instância e rodei `top` para ver a CPU subir em **tempo real**
4. No CloudWatch, abri o `LabCPUUtilizationAlarm` e atualizei o gráfico a cada minuto, até o estado mudar para **In alarm** 🔴
5. No gráfico, vi a `CPUUtilization` **passar da linha de 60%**
6. Verifiquei o e-mail e encontrei a **notificação da AWS** avisando que o alarme disparou ✅

> ⏱️ **Por que demora alguns minutos?** As métricas padrão do EC2 (monitoramento básico) chegam ao CloudWatch em intervalos de **cerca de 5 minutos**. Por isso o alarme e o e-mail não são instantâneos. O **monitoramento detalhado** reduz esse intervalo para 1 minuto.

### Tarefa 4: Criar um painel do CloudWatch
Em **CloudWatch → Dashboards → Create dashboard**:

| Passo | Valor |
|---|---|
| Nome | `LabEC2Dashboard` |
| Tipo de widget | **Line** (linha) |
| Métrica | EC2 → Per-Instance Metrics → `CPUUtilization` da *Stress Test* |

Salvei o painel e passei a ter um **atalho para ver a CPU da instância** em uma única tela. Painéis podem reunir métricas e alarmes de **vários recursos**, inclusive de **regiões diferentes**.

---

## 💡 O que eu aprendi

- **Alarme = métrica + limite + ação:** o CloudWatch observa uma métrica e, ao cruzar o limite, executa uma ação, como notificar um tópico SNS
- **SNS** desacopla **quem detecta** de **quem recebe**: o mesmo tópico pode avisar várias pessoas por e-mail, SMS ou até acionar outros serviços
- **Monitorar é parte da segurança:** picos de CPU inesperados podem indicar **uso indevido da instância**
- **Testar o alarme** é essencial: um alerta que nunca foi testado não dá confiança
- **Dashboards** transformam números soltos em uma **visão rápida da saúde** do ambiente
- Existe **atraso** entre o evento e o alerta, o que é importante considerar ao definir períodos e limites

## ✅ Boas práticas que levo daqui

- **Criar alarmes** para métricas importantes (CPU, memória, disco, erros), e não só olhar painéis
- **Definir limites realistas** para evitar tanto alertas falsos quanto alertas que nunca disparam
- **Confirmar as assinaturas** do SNS e **testar** o fluxo de ponta a ponta
- **Combinar** monitoramento (CloudWatch) com **registro** (logs) para investigar incidentes
- **Usar o Session Manager** em vez de abrir portas de acesso remoto
- **Encerrar o laboratório** ao final para liberar os recursos

---


## 👤 Autor

**Werther Crisley**
🔗 [GitHub](https://github.com/WertherCrisley)