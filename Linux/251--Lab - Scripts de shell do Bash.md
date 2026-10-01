# 📜 AWS re/Start — Shell Scripts do Bash (Backup Automatizado)

Laboratório prático do programa **AWS re/Start** em que, conectado por **SSH** a uma instância **Amazon Linux (EC2)**, escrevi meu primeiro **shell script em Bash** para **automatizar o backup** de uma pasta, gerando um arquivo compactado com a **data do dia no nome**.

![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white)
![Linux](https://img.shields.io/badge/Amazon_Linux-CLI-232F3E?logo=linux&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-script-4EAA25?logo=gnubash&logoColor=white)
![Status](https://img.shields.io/badge/status-concluído-success)

---

## 🎯 Objetivo do laboratório

Criar um **script de Bash** que automatiza o backup de uma pasta.

## 🧩 O que eu fiz

1. **Conectei por SSH** à instância Amazon Linux (`ec2-user`)
2. **Criei o arquivo** `backup.sh` e o tornei **executável**
3. **Escrevi o script** no Vim, com shebang, variáveis e o comando `tar`
4. **Executei o script** e vi o `tar` listando os arquivos incluídos
5. **Confirmei** que o backup foi criado na pasta `backups`

---

## 🧠 O que eu aprendi

### 1. O que é um shell script
Um **shell script** é um arquivo de texto com uma **sequência de comandos** que o Bash executa em ordem. Tudo o que eu faria digitando um a um no terminal pode virar um script reutilizável.

### 2. Preparando o arquivo
```bash
touch backup.sh          # cria o arquivo vazio
sudo chmod 755 backup.sh # torna o arquivo executável
vi backup.sh             # abre o editor para escrever o script
```

O `755` dá permissões assim (assunto do lab de permissões):

| | Dono | Grupo | Outros |
|---|---|---|---|
| **755** | `rwx` (ler, gravar, executar) | `r-x` (ler, executar) | `r-x` (ler, executar) |

> **Lição:** um script **só roda como programa** se tiver a permissão de **execução (`x`)**.

### 3. O script `backup.sh`
```bash
#!/bin/bash
DAY="$(date +%Y_%m_%d)"
BACKUP="/home/$USER/backups/$DAY-backup-CompanyA.tar.gz"
tar -csvpzf $BACKUP /home/$USER/CompanyA
```

**Linha por linha:**

| Linha | O que faz |
|---|---|
| `#!/bin/bash` | **Shebang**: diz ao sistema qual interpretador executa o script |
| `DAY="$(date +%Y_%m_%d)"` | Guarda a **data do dia** (ex.: `2026_10_01`) em uma variável |
| `BACKUP="..."` | Monta o **caminho e o nome** do arquivo de backup, usando a data |
| `tar -csvpzf $BACKUP ...` | Cria o backup **compactado** da pasta `CompanyA` |

**Conceitos importantes:**

- **Variáveis:** criadas sem espaços ao redor do `=` (`DAY="..."`) e usadas com `$` (`$DAY`)
- **Substituição de comando `$(...)`:** executa o comando de dentro e **usa o resultado** (aqui, a data)
- **`$USER`:** variável do sistema com o **usuário atual** (`ec2-user`), equivalente ao `whoami`
- **`date +%Y_%m_%d`:** formata a data como **ano_mês_dia**

> **Lição:** colocar a **data no nome** do arquivo evita sobrescrever o backup anterior e deixa o histórico organizado.

> 💡 **Observação sobre o lab:** o texto cita um formato de data mais longo (com hora e minuto), mas o script final mostrado usa só `%Y_%m_%d`. Dá para incluir a hora com `%H_%M` se quiser mais de um backup por dia.

### 4. Executando o script
```bash
./backup.sh
```

O `tar` listou todos os arquivos adicionados ao backup, e apareceu o aviso *"Removing leading `/' from member names"*.

> **Lição:** esse aviso é normal. O `tar` **remove a barra inicial** dos caminhos absolutos para que, ao extrair o backup, os arquivos não sobrescrevam pastas do sistema por acidente.

### 5. Conferindo o resultado
```bash
ls backups/
```

O arquivo `AAAA_MM_DD-backup-CompanyA.tar.gz` apareceu na pasta `backups`.

> ⚠️ **Atenção:** o script grava o backup em `/home/$USER/backups/`. Se essa pasta **não existir**, o `tar` falha ao criar o arquivo. Nesse caso, basta criá-la antes com `mkdir backups` (ou `mkdir -p` dentro do próprio script).

### 6. Automação completa com cron
O lab sugere que o script pode ser **agendado com o `cron`** para gerar um backup **diário** automaticamente. Juntando com o lab de processos, uma linha de crontab para isso poderia ser, por exemplo:

```cron
0 0 * * * /home/ec2-user/backup.sh
```

> **Lição:** script + cron = **rotina automatizada**, sem depender de eu lembrar de rodar o backup.

---

## 🛠️ Comandos e conceitos praticados

| Item | Função |
|---|---|
| `touch` | Cria um arquivo vazio |
| `chmod 755` | Torna o arquivo executável |
| `vi` | Edita o arquivo |
| `#!/bin/bash` | Shebang: define o interpretador |
| `VAR="valor"` | Cria uma variável |
| `$(comando)` | Usa a saída de um comando como valor |
| `$USER` | Usuário atual |
| `date +%Y_%m_%d` | Data formatada |
| `tar -csvpzf` | Cria um backup compactado |
| `./script.sh` | Executa o script |
| `ls` | Confere o resultado |

## ✅ Boas práticas que levo daqui

- **Automatizar tarefas repetitivas** com scripts
- Sempre começar o script com o **shebang**
- Dar ao script **só as permissões necessárias**
- **Incluir a data no nome** dos backups
- **Testar o script** manualmente antes de agendá-lo no cron
- Usar **caminhos absolutos** em scripts agendados
- **Entre aspas** as variáveis (`"$BACKUP"`) para evitar problemas com espaços nos nomes
- **Encerrar o laboratório** ao final para liberar os recursos

---

## 📚 Próximos passos

- Agendar o `backup.sh` no **cron** para rodar todo dia
- Receber a pasta de origem como **parâmetro** (`$1`) em vez de fixá-la no script
- Adicionar **verificações** (`if`) para checar se a pasta de backup existe
- Enviar o backup para o **Amazon S3** com a AWS CLI (`aws s3 cp`)
- Documentar os próximos labs do re/Start neste repositório

---

## 👤 Autor

**Werther Crisley**
🔗 [GitHub](https://github.com/WertherCrisley)