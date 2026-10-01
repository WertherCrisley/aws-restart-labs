# AWS re/Start — Lab: Introduction to Amazon EC2

> ⏱️ Duração aproximada: **45 minutos**

## Visão geral

O **Amazon EC2 (Elastic Compute Cloud)** é um serviço web que fornece capacidade computacional redimensionável na nuvem. Principais vantagens:

- Obtém e configura capacidade com o mínimo de esforço
- Controle completo dos recursos de computação
- Novas instâncias sobem em **minutos**
- Escala para mais ou para menos conforme a necessidade
- **Paga só pelo que usa**
- Ajuda a criar aplicações resistentes a falhas

## Objetivos do lab

- [x] Iniciar um servidor web com **proteção contra encerramento** ativada
- [x] Monitorar a instância EC2
- [x] Modificar o **security group** para permitir acesso HTTP
- [x] Redimensionar a instância conforme a necessidade
- [x] Testar a proteção contra encerramento
- [x] Terminar a instância

---

## Acessando o console da AWS

1. Clique em **Start Lab** (canto superior direito).
2. Aguarde o círculo ao lado de **AWS** ficar **verde** (🔴 não iniciado · 🟡 iniciando · 🟢 pronto).
3. Clique no círculo verde — abre o Console de Gerenciamento da AWS em nova aba (login automático).

> 💡 Erro **Access Denied**? Feche a caixa de erro e clique em **Start Lab** de novo.
> 💡 Aba não abriu? Provavelmente o navegador bloqueou pop-ups — permita.
> ⚠️ **Não altere a Região** do laboratório.

---

## Tarefa 1: Iniciar a instância EC2

**Caminho:** Serviços → **EC2** → Painel do EC2 → **Executar instância**

### Etapa 1 — Nome
- **Name:** `Web Server`

### Etapa 2 — AMI
- Manter **Amazon Linux 2023** (padrão do Quick Start).
- A AMI inclui: template do volume-raiz (SO/aplicações), permissões de execução e mapeamento de dispositivos de blocos.

### Etapa 3 — Tipo de instância
- **t3.micro** (2 vCPUs, 1 GiB de memória)

### Etapa 4 — Par de chaves
- **Proceed without a key pair (Not recommended)** — neste lab não vamos fazer login na instância.

### Etapa 5 — Rede
- Painel *Network settings* → **Editar**
- **VPC:** `Lab VPC`
- **Security group name:** `Web Server security group`
- **Descrição:** `Security group for my web server`
- **Regras de entrada:** clicar em **Remover** (sem SSH, para reforçar a segurança)

> 🔥 Um **security group** funciona como um **firewall virtual** que controla o tráfego de entrada e saída das instâncias. Mudanças nas regras são aplicadas automaticamente.

### Etapa 6 — Armazenamento
- Manter o padrão: volume raiz **EBS de 8 GiB**.

### Etapa 7 — Detalhes avançados
- **Termination protection:** `Enable`
- **User data** (cole o script abaixo):

```bash
#!/bin/bash
yum -y install httpd
systemctl enable httpd
systemctl start httpd
echo '<html><h1>Hello From Your Web Server!</h1></html>' > /var/www/html/index.html
```

**O que o script faz:**
1. Instala o servidor web **Apache (httpd)**
2. Configura para iniciar automaticamente no boot
3. Inicia o servidor web
4. Cria uma página web simples

### Etapa 8 — Executar
- **Executar instância** → **Visualizar todas as instâncias**
- Estados: `Pendente` → `Em execução`
- Aguardar: **Estado = Em execução** e **Verificações de status = 2/2 aprovadas**

---

## Tarefa 2: Monitorar a instância

- Aba **Status checks**: confira que *System reachability* e *Instance reachability* foram aprovadas.
- Aba **Monitoring**: métricas do **Amazon CloudWatch**.
  - Monitoramento **básico** (5 min) vem ativado por padrão.
  - Monitoramento **detalhado** (1 min) pode ser ativado.
- **Ações → Monitorar e solucionar problemas → Get Instance Screenshot**
  - Útil quando você não consegue acessar via SSH/RDP: mostra como seria a tela do console.
  - Depois, clique em **Cancelar**.

---

## Tarefa 3: Atualizar o security group e acessar o servidor web

1. Aba **Details** → copie o **Public IPv4 address**.
2. Cole em uma nova aba do navegador e dê Enter.

**❓ Você consegue acessar o servidor web? Por quê?**
**Não.** O security group não permite tráfego de entrada na **porta 80** (HTTP). Isso demonstra o security group funcionando como firewall.

**Corrigindo:**

1. Menu esquerdo → **Network & Security → Security Groups**
2. Selecione **Web Server security group**
3. Aba **Inbound rules** → **Editar regras de entrada** → **Adicionar regra**
   - **Tipo:** `HTTP`
   - **Origem:** `IPv4 em qualquer lugar` (Anywhere-IPv4)
4. **Salvar regras**
5. Atualize a aba do navegador → deve aparecer: **Hello From Your Web Server!** ✅

---

## Tarefa 4: Redimensionar a instância (tipo + volume EBS)

> Instância **subutilizada** (grande demais) ou **superutilizada** (pequena demais) → dá para mudar o tipo. O disco também pode crescer.

### 4.1 Interromper a instância
- **Instances** → **Estado da instância → Interromper instância** → **Interromper**
- Aguarde `Stopped`.

> 💰 Instância interrompida **não gera cobrança de computação**, mas o **armazenamento EBS** conectado continua sendo cobrado.

### 4.2 Alterar o tipo
- **Ações → Configurações de instância → Alterar tipo de instância**
- **Tipo:** `t3.small` (o dobro de memória do t3.micro)

### 4.3 Redimensionar o volume EBS
- Menu esquerdo → **Elastic Block Store → Volumes**
- Selecione o volume → **Ações → Modificar volume**
- **Tamanho:** de `8` para `10` GiB → **Modificar** → confirmar

### 4.4 Iniciar de novo
- **Instances** → selecione `Web Server` → **Estado da instância → Iniciar instâncias**

**Resultado:** `t3.micro` → `t3.small` e disco `8 GiB` → `10 GiB` ✅

> ⚠️ O lab pode restringir outros tipos de instância e volumes grandes.

---

## Tarefa 5: Testar a proteção contra encerramento

1. **Instances** → selecione `Web Server` → **Estado da instância → Encerrar (excluir) instância** → **Encerrar**
2. **Resultado:** aparece um erro vermelho — *"Falha ao terminar uma instância"* — porque a **proteção contra encerramento** está ativa. 🛡️
3. Desativar a proteção:
   - **Ações → Configurações de instância → Change termination protection**
   - Desmarque **Enable** → **Save**
4. Agora sim: **Ações → Estado da instância → Terminate instance** → **Encerrar**

> Em instâncias baseadas em EBS, o volume raiz é **excluído por padrão** ao terminar a instância. Instância terminada **não pode** ser reconectada nem reiniciada.

---

## Encerrando o lab

1. **End Lab** (topo da página) → **Yes**
2. Aparece *"DELETE has been initiated..."* e depois *"Ended AWS Lab Successfully"*.

---

## 📌 Resumo rápido / cola

| Conceito | O que é |
|---|---|
| **EC2** | Servidores virtuais redimensionáveis na nuvem |
| **AMI** | Template para iniciar a instância (SO + config) |
| **Tipo de instância** | Combinação de CPU, memória, armazenamento e rede (ex.: `t3.micro`) |
| **Par de chaves** | Chave pública/privada para login seguro (SSH/RDP) |
| **Security group** | Firewall virtual da instância (regras de entrada/saída) |
| **EBS** | Disco virtual anexado pela rede (volume raiz de 8 GiB no lab) |
| **User data** | Script executado no primeiro boot da instância |
| **Termination protection** | Impede encerramento acidental da instância |
| **CloudWatch** | Métricas e monitoramento (básico 5 min / detalhado 1 min) |
| **Porta 80** | HTTP — precisa estar liberada no security group |

## ✅ O que eu aprendi

- Como lançar uma instância EC2 com script de **user data**
- Como o **security group** bloqueia/libera tráfego
- Como monitorar com **status checks** e **CloudWatch**
- Como **redimensionar** tipo de instância e volume EBS (precisa **parar** antes)
- Como a **proteção contra encerramento** evita exclusões acidentais