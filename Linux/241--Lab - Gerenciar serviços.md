# 📈 AWS re/Start — Gerenciamento de Serviços e Monitoramento (`systemctl`, `top` e CloudWatch)

Laboratório prático do programa **AWS re/Start** em que, conectado por **SSH** a uma instância **Amazon Linux (EC2)**, gerenciei o serviço do servidor web **Apache (`httpd`)** com o `systemctl` e depois **monitorei a instância** de duas formas: pelo terminal, com o `top`, e pelo console da AWS, com o **Amazon CloudWatch**.

![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white)
![CloudWatch](https://img.shields.io/badge/AWS-CloudWatch-FF4F8B?logo=amazoncloudwatch&logoColor=white)
![Linux](https://img.shields.io/badge/Amazon_Linux-CLI-232F3E?logo=linux&logoColor=white)
![Apache](https://img.shields.io/badge/Apache-httpd-D22128?logo=apache&logoColor=white)
![Status](https://img.shields.io/badge/status-concluído-success)

---

## 🎯 Objetivos do laboratório

- Verificar o status do serviço **`httpd`**, iniciá-lo e confirmar que o servidor responde por HTTP
- Monitorar a instância EC2 usando o comando **`top`**
- Monitorar a instância EC2 usando o **AWS CloudWatch**

## 🧩 O que eu fiz

1. **Conectei por SSH** à instância Amazon Linux (`ec2-user`)
2. **Verifiquei o status** do serviço `httpd` (estava inativo)
3. **Iniciei o serviço** e confirmei que passou a ficar ativo
4. **Testei no navegador** acessando o IP público da instância e vi a página de teste do Apache
5. **Parei o serviço** com `systemctl stop`
6. **Observei o `top`** com a instância em uso normal
7. **Rodei um script de estresse** (`stress.sh`) para simular uma carga pesada de CPU e acompanhei o efeito no `top`
8. **Abri o CloudWatch** e analisei o painel automático do EC2, onde apareceu o pico de CPU

---

## 🧠 O que eu aprendi

### 1. O que é o `httpd` e o que é um serviço
O **`httpd`** é o serviço do **servidor web Apache**. Um **serviço** (ou *daemon*) é um programa que roda em segundo plano, e no Linux moderno ele é gerenciado pelo **`systemctl`**.

### 2. Gerenciando um serviço com `systemctl`
```bash
sudo systemctl status httpd.service   # mostra o estado do serviço
sudo systemctl start httpd.service    # inicia o serviço
sudo systemctl stop httpd.service     # para o serviço
```

| Comando | O que faz |
|---|---|
| `status` | Mostra se o serviço está ativo, inativo ou com falha |
| `start` | Inicia o serviço agora |
| `stop` | Para o serviço agora |

Na primeira verificação, o status mostrou o serviço como **carregado (instalado), mas inativo (dead)**. Depois do `start`, o status passou a **ativo (running)**.

> **Lição:** "instalado" e "em execução" são coisas diferentes. O `status` é o primeiro passo de qualquer diagnóstico de serviço.

> 💡 **Para ir além:** o `start` só inicia o serviço **agora**. Para ele subir **automaticamente no boot**, é preciso o **`systemctl enable`**, que foi o que o script de *user data* fez no primeiro lab de EC2.

### 3. Testando o servidor no navegador
Com o serviço ativo, acessei `http://<ip-público>` no navegador e apareceu a **página de teste do Apache**, confirmando que o servidor web estava respondendo.

> **Lição:** para o navegador conseguir acessar, o tráfego **HTTP (porta 80)** precisa estar liberado no **security group** da instância, assunto que vi no lab de introdução ao EC2.

### 4. Monitorando pelo terminal com `top`
```bash
top
```

O `top` mostra, em tempo real, os processos em execução e o uso de **CPU** e **memória**. Com a instância parada, o uso de CPU ficou baixo. Para sair, usei **`q`**.

### 5. Simulando uma carga pesada
```bash
./stress.sh & top
```

- **`./stress.sh`** executa o script que simula uma carga pesada na CPU (ele roda por cerca de **seis minutos** e depois é interrompido)
- **`&`** coloca o script em **segundo plano**, liberando o terminal
- **`top`** abre o monitor logo em seguida

No `top`, o processo `stress` apareceu **rodando como `ec2-user`**, com **uso de CPU bem mais alto** que antes.

> **Lição:** o `&` é ótimo para rodar algo em segundo plano e continuar usando o terminal ao mesmo tempo.

### 6. Monitorando pelo console com CloudWatch
O **Amazon CloudWatch** coleta métricas dos recursos da AWS e permite visualizá-las em painéis.

Caminho que segui:

1. No console da AWS, pesquisei **CloudWatch**
2. No menu à esquerda: **Dashboards → Automatic dashboards**
3. Selecionei **Elastic Compute Cloud (EC2)**

O painel automático mostra, entre outras, estas métricas:

| Métrica | O que mede |
|---|---|
| **CPUUtilization** | Percentual de uso de CPU |
| **DiskReadBytes / DiskWriteBytes** | Volume de leitura e gravação em disco |
| **DiskReadOps / DiskWriteOps** | Quantidade de operações de disco |
| **NetworkIn** | Tráfego de rede de entrada |

No gráfico de **CPU**, vi um **pico** que correspondia ao momento em que rodei o script de estresse. Cerca de **5 minutos depois**, a utilização caiu.

> **Lição:** o CloudWatch por padrão **agrega os dados em intervalos de 5 minutos**, então o gráfico não reage instantaneamente. Para análises mais finas, existe o monitoramento detalhado, de 1 minuto.

> 💡 O CloudWatch também permite **personalizar painéis**, criar **alarmes** e **acionar eventos**, o que o torna essencial para monitorar aplicações em tempo real.

### 7. Duas formas de monitorar, dois usos
| | `top` | CloudWatch |
|---|---|---|
| Onde | Dentro da instância (terminal) | No console da AWS |
| Visão | Momento atual, em tempo real | Histórico e tendências |
| Bom para | Ver **qual processo** consome recursos | Ver **quando** e **quanto** o recurso foi usado |

---

## 🛠️ Comandos e ferramentas praticados

| Item | Função |
|---|---|
| `systemctl status <serviço>` | Mostra o estado de um serviço |
| `systemctl start <serviço>` | Inicia um serviço |
| `systemctl stop <serviço>` | Para um serviço |
| `top` | Monitora processos e recursos em tempo real |
| `./stress.sh &` | Executa um script em segundo plano |
| Navegador (`http://<ip>`) | Testa se o servidor web responde |
| AWS CloudWatch | Métricas, painéis e alarmes dos recursos da AWS |

## ✅ Boas práticas que levo daqui

- **Verificar o status** do serviço antes de tentar corrigi-lo
- **Testar de fora** (navegador) além de olhar o status
- **Combinar monitoramento local** (`top`) e **monitoramento na nuvem** (CloudWatch)
- Lembrar que **dados do CloudWatch têm atraso** (agregação de 5 minutos por padrão)
- Usar **alarmes** para ser avisado de problemas em vez de depender só de olhar painéis
- **Encerrar o laboratório** ao final para liberar os recursos

---

## 📚 Próximos passos

- Praticar `systemctl enable`, `disable` e `restart`
- Ler logs de serviços com `journalctl`
- Criar um **alarme** no CloudWatch para alta utilização de CPU
- Explorar o **monitoramento detalhado** (1 minuto) e o **CloudWatch Logs**
- Documentar os próximos labs do re/Start neste repositório

---

## 👤 Autor

**Werther Crisley**
🔗 [GitHub](https://github.com/WertherCrisley)