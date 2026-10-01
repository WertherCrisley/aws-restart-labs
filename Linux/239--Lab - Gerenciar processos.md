# ⚙️ AWS re/Start — Gerenciando Processos no Linux (`ps`, `top` e `cron`)

Laboratório prático do programa **AWS re/Start** em que, conectado por **SSH** a uma instância **Amazon Linux (EC2)**, listei e filtrei **processos** em execução, monitorei o sistema com o **`top`** e agendei uma tarefa automática com o **`cron`**.

![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white)
![Linux](https://img.shields.io/badge/Amazon_Linux-CLI-232F3E?logo=linux&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-shell-4EAA25?logo=gnubash&logoColor=white)
![Status](https://img.shields.io/badge/status-concluído-success)

---

## 🎯 Objetivos do laboratório

- Criar um arquivo de log com a listagem de processos
- Usar o comando `top`
- Estabelecer uma tarefa repetitiva (cron) que gera um arquivo de auditoria

## 🧩 O que eu fiz

1. **Conectei por SSH** à instância Amazon Linux (`ec2-user`)
2. **Listei os processos** com `ps -aux`, filtrei os do `root` e salvei o resultado em `processes.csv`
3. **Monitorei o sistema** em tempo real com `top`
4. **Criei um trabalho cron** que gera automaticamente um arquivo de auditoria dos arquivos `.csv`
5. **Validei** o agendamento com `crontab -l`

---

## 🧠 O que eu aprendi

### 1. O que é um processo
Um **processo** é um programa em execução. O Linux acompanha todos eles, e é possível listá-los, monitorá-los e analisá-los, o que é essencial para **troubleshooting** e **auditoria**.

### 2. Listando processos com `ps`
```bash
ps -aux
```

| Opção | Significado |
|---|---|
| `a` | Mostra processos de todos os usuários |
| `u` | Formato detalhado (usuário, CPU, memória etc.) |
| `x` | Inclui processos sem terminal associado |

### 3. Filtrando e salvando a saída: `grep -v` e `tee`
```bash
sudo ps -aux | grep -v root | sudo tee SharedFolders/processes.csv
cat SharedFolders/processes.csv
```

Passo a passo do comando:

| Parte | O que faz |
|---|---|
| `ps -aux` | Lista todos os processos |
| `\|` (pipe) | Passa a saída para o próximo comando |
| `grep -v root` | **Exclui** (`-v` = inverter) as linhas que contêm `root` |
| `tee <arquivo>` | Grava o resultado **no arquivo e também no terminal** |
| `cat` | Exibe o arquivo para conferir |

> **Lição:** combinar comandos com **pipes** deixa o terminal muito poderoso. Cada comando faz uma coisa simples, e juntos resolvem tarefas maiores.

> **Observação:** o lab também pede para omitir processos com `[` ou `]` na coluna COMMAND. Esses colchetes costumam indicar processos do kernel, que normalmente já pertencem ao `root` e, por isso, são removidos pelo filtro do `grep -v root`.

### 4. Monitorando o sistema com `top`
```bash
top
```

O `top` mostra, **em tempo real**, o desempenho do sistema e os processos e threads ativos. Na linha **Tasks**, vi o total de tarefas e como elas estão divididas:

| Estado | Significado |
|---|---|
| **running** | Em execução |
| **sleeping** | Aguardando um evento (a maioria fica aqui) |
| **stopped** | Interrompidas |
| **zombie** | Terminaram, mas ainda constam na tabela de processos |

Também mostra o uso de **CPU**, de **memória** e de **swap (troca)**.

- **`q`**: sai do `top`
- **`top -hv`**: mostra ajuda e versão

> **Lição:** a maioria dos processos fica em estado *sleeping*, e poucos estão realmente rodando ao mesmo tempo. Ver muitos processos *zombie* ou o uso de CPU e memória nas alturas é um sinal de que algo precisa de atenção.

### 5. Automatizando tarefas com `cron`
O **cron** é o agendador de tarefas do Linux: um serviço (daemon) que executa comandos em horários definidos. A lista de tarefas fica em um arquivo **crontab**.

```bash
sudo crontab -e    # edita o crontab (abre o editor padrão, o Vim)
sudo crontab -l    # lista o crontab para conferir
```

Conteúdo que configurei:

```cron
SHELL=/bin/bash
PATH=/usr/bin:/bin:/usr/local/bin
MAILTO=root
0 * * * * ls -la $(find .) | sed -e 's/..csv/#####.csv/g' > /home/ec2-user/companyA/SharedFolders/filteredAudit.csv
```

**Variáveis do crontab:**

| Variável | Função |
|---|---|
| `SHELL` | Qual shell executa os comandos |
| `PATH` | Onde procurar os programas |
| `MAILTO` | Para quem enviar a saída das tarefas |

**Os 5 campos de horário, seguidos do comando:**

```
┌───────── minuto (0-59)
│ ┌─────── hora (0-23)
│ │ ┌───── dia do mês (1-31)
│ │ │ ┌─── mês (1-12)
│ │ │ │ ┌─ dia da semana (0-7)
│ │ │ │ │
0 * * * *  comando
```

> O asterisco (`*`) significa "qualquer valor".

**O que o comando agendado faz:**
- `find .` lista os arquivos a partir da pasta atual
- `ls -la` mostra os detalhes
- `sed` substitui o trecho do nome dos arquivos `.csv` por `#####`, **mascarando** os nomes na auditoria
- `>` grava tudo no arquivo `filteredAudit.csv`

> ⚠️ **Detalhe de agendamento:** `0 * * * *` executa **a cada hora** (no minuto 0). O objetivo do lab fala em uma execução **por dia**, e para isso o certo seria algo como `0 0 * * *` (todo dia à meia-noite). É uma boa lição de **ler cada campo do cron com atenção**.

> 💡 **Outro ponto:** jobs do cron não rodam, por padrão, na pasta em que o comando foi escrito. Em ambiente real, o ideal é usar **caminhos absolutos** em vez de `find .`.

---

## 🛠️ Comandos praticados

| Comando | Função |
|---|---|
| `pwd` / `cd` | Confirma e muda o diretório atual |
| `ps -aux` | Lista todos os processos |
| `grep -v <texto>` | Exclui linhas que contêm o texto |
| `tee <arquivo>` | Grava a saída no terminal e em um arquivo |
| `cat <arquivo>` | Exibe o conteúdo de um arquivo |
| `top` | Monitora processos e desempenho em tempo real |
| `crontab -e` | Edita as tarefas agendadas |
| `crontab -l` | Lista as tarefas agendadas |
| `find`, `ls -la`, `sed` | Busca, lista em detalhes e substitui texto |

## ✅ Boas práticas que levo daqui

- **Usar pipes** para combinar comandos pequenos em tarefas maiores
- **Registrar a saída em arquivos** para auditoria e histórico
- **Monitorar processos e recursos** com `top` ao investigar lentidão
- **Automatizar tarefas repetitivas** com cron em vez de executar manualmente
- **Ler com atenção os campos do cron**: um erro troca "uma vez por dia" por "a cada hora"
- Usar **caminhos absolutos** em tarefas agendadas
- **Validar** sempre com `cat` e `crontab -l`
- **Encerrar o laboratório** ao final para liberar os recursos

---

## 📚 Próximos passos

- Praticar `kill`, `pkill` e `killall` para encerrar processos
- Explorar `htop`, `pgrep` e `pstree`
- Estudar jobs em segundo plano (`&`, `jobs`, `fg`, `bg`) e `nohup`
- Praticar expressões do cron (por exemplo, `*/15 * * * *` e `0 0 * * 1`)
- Documentar os próximos labs do re/Start neste repositório

---

## 👤 Autor

**Werther Crisley**
🔗 [GitHub](https://github.com/WertherCrisley)