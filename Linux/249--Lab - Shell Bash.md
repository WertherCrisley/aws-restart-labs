# 🐚 AWS re/Start — O Shell Bash (Alias e Variável `PATH`)

Laboratório prático do programa **AWS re/Start** em que, conectado por **SSH** a uma instância **Amazon Linux (EC2)**, personalizei o **shell Bash**: criei um **alias** para automatizar backups com `tar` e entendi como a variável **`PATH`** define onde o sistema procura os comandos executáveis.

![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white)
![Linux](https://img.shields.io/badge/Amazon_Linux-CLI-232F3E?logo=linux&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-shell-4EAA25?logo=gnubash&logoColor=white)
![Status](https://img.shields.io/badge/status-concluído-success)

---

## 🎯 Objetivos do laboratório

- Criar e usar um **alias** para fazer o backup de uma pasta completa
- Trabalhar com a variável **`PATH`** e adicionar uma nova pasta a ela

## 🧩 O que eu fiz

1. **Conectei por SSH** à instância Amazon Linux (`ec2-user`)
2. **Criei um alias `backup`** que encapsula o comando `tar`
3. **Usei o alias** para fazer o backup da pasta `CompanyA`
4. **Executei um script** (`hello.sh`) de três jeitos diferentes e vi um deles falhar
5. **Descobri o motivo** inspecionando a variável `PATH`
6. **Adicionei a pasta `bin`** da `CompanyA` ao `PATH` e executei o script de qualquer lugar

---

## 🧠 O que eu aprendi

### 1. Alias: atalhos para comandos
Um **alias** é um apelido para um comando (ou conjunto de opções), que evita digitar tudo toda vez.

```bash
alias backup='tar -cvzf '
```

Agora, em vez do comando completo do `tar`, uso:

```bash
backup backup_companyA.tar.gz CompanyA
```

O **primeiro parâmetro** é o nome do arquivo de backup e o **segundo** é a pasta a ser salva.

**Opções do `tar` usadas:**

| Opção | Significado |
|---|---|
| `c` | **C**riar um novo arquivo |
| `v` | Modo **v**erboso: mostra o que está sendo adicionado |
| `z` | Compacta com **g**zip |
| `f` | Define o nome do arquivo (**f**ile) de saída |

O comando listou todo o conteúdo da `CompanyA`, e o `ls` confirmou que o `backup_companyA.tar.gz` foi criado.

> **Lição:** o alias transforma uma operação repetitiva em **um comando curto**, reduzindo erros e ganhando tempo.

> ⚠️ **Detalhe importante:** o alias criado no terminal vale **só para a sessão atual**. Ao fechar o terminal, ele some. Para torná-lo **permanente**, a linha do `alias` precisa ser adicionada ao arquivo **`~/.bashrc`**.

### 2. Três jeitos de executar um script
Dentro da pasta `CompanyA`, tentei rodar o script `hello.sh` de três formas:

| Comando | Resultado | Por quê |
|---|---|---|
| `./hello.sh` (dentro da pasta `bin`) | ✅ `hello ec2-user` | Caminho **relativo** à pasta atual |
| `./bin/hello.sh` (a partir da `CompanyA`) | ✅ `hello ec2-user` | Caminho **relativo** até o script |
| `hello.sh` | ❌ `command not found` | O Bash não sabe **onde** procurar esse nome |

> **Lição:** digitar só o nome de um programa funciona apenas se ele estiver em uma pasta listada no **`PATH`**. A pasta atual **não** entra nessa busca automaticamente, e é por isso que usamos o `./` para dizer "execute o que está **aqui**".

### 3. A variável `PATH`
```bash
echo $PATH
```

O `PATH` é uma **lista de pastas, separadas por `:`**, onde o sistema procura os executáveis quando eu digito um comando. Ela segue a ordem, e a busca para no primeiro resultado encontrado.

Exemplo de pastas típicas: `/usr/local/bin`, `/usr/bin`, `/usr/local/sbin`, `/usr/sbin`...

A pasta `/home/ec2-user/CompanyA/bin` **não estava** nessa lista, então o `hello.sh` não era encontrado pelo nome.

### 4. Adicionando uma pasta ao `PATH`
```bash
PATH=$PATH:/home/ec2-user/CompanyA/bin
hello.sh
# hello ec2-user
```

- **`$PATH`** traz o valor atual da variável
- **`:/home/ec2-user/CompanyA/bin`** acrescenta a nova pasta ao final

Depois disso, o `hello.sh` passou a funcionar **de qualquer pasta**, só pelo nome.

> **Lição:** o `PATH=$PATH:nova-pasta` **acrescenta** sem apagar o que já existia. Escrever `PATH=nova-pasta` (sem o `$PATH:`) **substituiria** tudo e quebraria comandos básicos como `ls`.

> ⚠️ **Detalhe importante:** assim como o alias, essa alteração é **temporária** (vale só para a sessão). Para deixá-la **permanente**, a linha `export PATH=$PATH:/home/ec2-user/CompanyA/bin` deve ir no `~/.bashrc`.

---

## 🛠️ Comandos praticados

| Comando | Função |
|---|---|
| `pwd` | Mostra o diretório atual |
| `alias nome='comando'` | Cria um atalho para um comando |
| `tar -cvzf` | Cria um backup compactado |
| `ls` | Lista o conteúdo do diretório |
| `cd` | Muda de diretório |
| `./script.sh` | Executa um script pelo caminho relativo |
| `echo $PATH` | Exibe as pastas onde o sistema procura executáveis |
| `PATH=$PATH:<pasta>` | Adiciona uma pasta ao `PATH` na sessão atual |

## ✅ Boas práticas que levo daqui

- **Criar aliases** para comandos longos ou repetitivos
- Lembrar que **alias e `PATH` alterados no terminal são temporários**; para persistir, usar o **`~/.bashrc`**
- **Acrescentar** ao `PATH` (`$PATH:nova-pasta`) em vez de substituí-lo
- Ter **cuidado com o que entra no `PATH`**, pois ele define quais programas são executados
- Organizar scripts próprios em uma pasta `bin` e incluí-la no `PATH`
- **Encerrar o laboratório** ao final para liberar os recursos

---

## 📚 Próximos passos

- Tornar o alias e o `PATH` permanentes editando o `~/.bashrc`
- Listar e remover aliases (`alias`, `unalias`)
- Estudar variáveis de ambiente (`export`, `env`, `printenv`)
- Escrever scripts Bash com variáveis, condicionais e laços
- Documentar os próximos labs do re/Start neste repositório

---

## 👤 Autor

**Werther Crisley**
🔗 [GitHub](https://github.com/WertherCrisley)