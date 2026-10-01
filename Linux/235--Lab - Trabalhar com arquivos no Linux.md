# 💾 AWS re/Start — Trabalhando com Arquivos: Backup com `tar`, Log e Transferência

Laboratório prático do programa **AWS re/Start** em que, conectado por **SSH** a uma instância **Amazon Linux (EC2)**, criei um **backup compactado** de toda uma estrutura de pastas, **registrei em um arquivo de log** a data, a hora e o nome do backup, e **movi o backup** para outra pasta.

![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white)
![Linux](https://img.shields.io/badge/Amazon_Linux-CLI-232F3E?logo=linux&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-shell-4EAA25?logo=gnubash&logoColor=white)
![Status](https://img.shields.io/badge/status-concluído-success)

---

## 🎯 Objetivos do laboratório

- Criar um arquivo de backup de toda a estrutura de pastas usando **`tar`**
- Registrar em log a criação do backup (data, hora e nome do arquivo)
- Transferir o arquivo de backup para outra pasta

## 🧩 O que eu fiz

1. **Conectei por SSH** à instância Amazon Linux (`ec2-user`)
2. **Verifiquei** a estrutura da pasta `CompanyA` com `ls -R`
3. **Criei o backup** compactado da pasta inteira com `tar`
4. **Registrei o backup em log** em um arquivo `backups.csv`, usando `echo` e `tee`
5. **Conferi** o conteúdo do log com `cat`
6. **Movi o arquivo de backup** para a pasta `IA` e validei com `ls`

---

## 🗂️ Estrutura trabalhada

```
/home/ec2-user/CompanyA/
├── Employees/
│   └── Schedules.csv
├── Finance/
│   └── Salary.csv
├── HR/
│   ├── Assessments.csv
│   └── Managers.csv
├── IA/
├── Management/
│   ├── Promotions.csv
│   └── Sections.csv
└── SharedFolders/
```

---

## 🧠 O que eu aprendi

### 1. Criando um backup compactado com `tar`
```bash
tar -csvpzf backup.CompanyA.tar.gz CompanyA
```

O `tar` junta uma pasta inteira (com todas as subpastas e arquivos) em **um único arquivo**, e aqui ele já saiu compactado. As opções principais:

| Opção | Significado |
|---|---|
| `c` | **C**riar um novo arquivo de backup |
| `v` | Modo **v**erboso: mostra cada arquivo adicionado |
| `p` | Preserva as **p**ermissões dos arquivos |
| `z` | Compacta com **g**zip (`.gz`) |
| `f` | Define o nome do arquivo (**f**ile) de saída |

> **Lição:** o arquivo `backup.CompanyA.tar.gz` guarda toda a estrutura e pode ser copiado e descompactado em **outro local ou outro servidor**, o que o torna ótimo para backup e transporte de dados.

Confirmei a criação com `ls`, que mostrou o backup ao lado da pasta original.

### 2. Registrando o backup em um log
Primeiro criei o arquivo de log, depois escrevi nele a data, a hora e o nome do backup:

```bash
cd /home/ec2-user/CompanyA
touch SharedFolders/backups.csv
echo "25 Aug 25 2021, 16:59, backup.CompanyA.tar.gz" | sudo tee SharedFolders/backups.csv
cat SharedFolders/backups.csv
```

- **`echo`** imprime o texto
- **`|` (pipe)** envia a saída de um comando para o próximo
- **`tee`** grava a entrada **no terminal e no arquivo** ao mesmo tempo
- **`cat`** exibe o conteúdo do arquivo para conferir

> **Lição:** manter um **log dos backups** mostra quando cada um foi feito e ajuda a **evitar backups desnecessários**.

> ⚠️ **Detalhe importante:** o `tee` **sobrescreve** o arquivo. Para **acrescentar** uma nova linha sem apagar as anteriores, o certo é usar `tee -a`.

### 3. Movendo o backup para outra pasta
```bash
mv ../backup.CompanyA.tar.gz IA/
ls . IA
```

O `..` indica a pasta acima (onde o backup foi criado), e o `IA/` é o destino. Com o `ls . IA`, confirmei que o arquivo **saiu da pasta de origem** e **apareceu na pasta `IA`**.

> **Lição:** em um cenário real, mover o backup é o passo para deixá-lo acessível a **outra equipe ou usuário** que não tem acesso ao local onde foi criado.

### 4. Caminhos absolutos e relativos
- **Absoluto**: `cd /home/ec2-user/CompanyA` funciona de qualquer lugar
- **Relativo**: `mv ../backup... IA/` depende da pasta em que estou, por isso uso `pwd` antes para confirmar onde estou

---

## 🛠️ Comandos praticados

| Comando | Função |
|---|---|
| `pwd` | Mostra o diretório atual |
| `ls -R <pasta>` | Lista o conteúdo recursivamente |
| `tar -czf` (e variações) | Cria um arquivo de backup compactado |
| `touch <arquivo>` | Cria um arquivo vazio |
| `echo "<texto>"` | Imprime um texto |
| `\|` (pipe) | Passa a saída de um comando para outro |
| `tee <arquivo>` | Grava a saída no terminal e no arquivo |
| `cat <arquivo>` | Exibe o conteúdo de um arquivo |
| `mv <origem> <destino>` | Move arquivos e pastas |
| `cd` | Muda de diretório |

## ✅ Boas práticas que levo daqui

- **Fazer backup antes de mudanças grandes** e guardá-lo em local separado
- **Registrar cada backup** em um log com data, hora e nome do arquivo
- **Conferir o resultado** de cada etapa com `ls` e `cat`
- Usar **nomes de arquivo descritivos** (`backup.CompanyA.tar.gz`) para saber o que há dentro
- Lembrar que `tee` sobrescreve, e **`tee -a`** acrescenta
- **Encerrar o laboratório** ao final para liberar os recursos

---

## 📚 Próximos passos

- Aprender a **restaurar** um backup (`tar -xzf`) e a listar o conteúdo sem extrair (`tar -tzf`)
- Gerar a data automaticamente no log com o comando `date`, em vez de digitar manualmente
- Estudar **agendamento de tarefas** (`cron`) para automatizar backups
- Explorar backups na nuvem com **Amazon S3**
- Documentar os próximos labs do re/Start neste repositório

---

## 👤 Autor

**Werther Crisley**
🔗 [GitHub](https://github.com/WertherCrisley)