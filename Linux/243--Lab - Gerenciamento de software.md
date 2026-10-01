# 📦 AWS re/Start — Gerenciamento de Software (`yum` e AWS CLI)

Laboratório prático do programa **AWS re/Start** em que, conectado por **SSH** a uma instância **Amazon Linux (EC2)**, usei o gerenciador de pacotes **`yum`** para atualizar o sistema, **reverter** uma instalação pelo histórico e, por fim, **instalar e configurar a AWS CLI** para conversar com a AWS direto do terminal.

![AWS](https://img.shields.io/badge/AWS-CLI-FF9900?logo=amazonaws&logoColor=white)
![Linux](https://img.shields.io/badge/Amazon_Linux-yum-232F3E?logo=linux&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-shell-4EAA25?logo=gnubash&logoColor=white)
![Status](https://img.shields.io/badge/status-concluído-success)

---

## 🎯 Objetivos do laboratório

- Atualizar a máquina Linux usando o gerenciador de pacotes
- Reverter (fazer *downgrade* de) um pacote usando o gerenciador de pacotes
- Instalar a **AWS Command Line Interface (AWS CLI)**

## 🧩 O que eu fiz

1. **Conectei por SSH** à instância Amazon Linux (`ec2-user`)
2. **Consultei e apliquei atualizações** (incluindo as de segurança) com `yum`
3. **Instalei o `httpd`** e consultei o histórico de transações
4. **Reverti uma transação** com `yum history undo`
5. **Verifiquei** os pré-requisitos (Python e pip)
6. **Baixei, descompactei e instalei** a AWS CLI
7. **Configurei** a AWS CLI com região e credenciais
8. **Testei** a CLI consultando o tipo da instância com `aws ec2 describe-instance-attribute`

---

## 🧠 O que eu aprendi

### 1. O que é um gerenciador de pacotes
Um **gerenciador de pacotes** automatiza a **instalação, atualização e remoção** de programas, resolvendo as dependências por conta própria. No Amazon Linux, usei o **`yum`**.

### 2. Atualizando o sistema
```bash
sudo yum -y check-update      # consulta os repositórios por atualizações disponíveis
sudo yum update --security    # aplica só as atualizações de segurança
sudo yum -y upgrade           # atualiza todos os pacotes
```

| Comando | O que faz |
|---|---|
| `check-update` | **Lista** o que tem atualização, sem instalar nada |
| `update --security` | Aplica apenas **correções de segurança** |
| `upgrade` | Atualiza **todos** os pacotes instalados |
| `-y` | Responde "sim" automaticamente às confirmações |

> **Lição:** manter o sistema atualizado, principalmente com patches de **segurança**, é uma das práticas mais importantes de administração de servidores. Separar "ver o que há de novo" de "aplicar" evita surpresas.

### 3. Instalando um pacote
```bash
sudo yum install httpd -y
```

Esse comando instala o servidor Apache (`httpd`) e suas dependências.

### 4. Histórico de transações e reversão (rollback)
O `yum` registra **cada operação** como uma transação. Isso permite auditar o que foi feito e **desfazer** mudanças.

```bash
sudo yum history list            # lista as transações (ID, usuário, data, ação, itens alterados)
sudo yum history info <ID>       # detalhes de uma transação (início, fim, usuário, comando)
sudo yum -y history undo <ID>    # desfaz a transação
```

No histórico, vi que as transações têm usuários diferentes (por exemplo, o `ec2-user` e o `System`) e mostram a **quantidade de itens alterados**.

> **Lição:** o `history undo` é uma **rede de segurança**: se uma instalação ou atualização causar problemas, dá para reverter com um comando, sem precisar lembrar tudo o que foi instalado.

### 5. Verificando pré-requisitos
```bash
python3 --version     # confere se o Python 3 está instalado
pip3 --version        # confere se o pip está instalado
```

Antes de instalar um software, vale **conferir os requisitos**. O lab também lembra que o `pip` é o gerenciador de pacotes do Python, usado como meio principal de distribuição da AWS CLI em algumas plataformas.

### 6. Instalando a AWS CLI v2
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws help
```

| Passo | O que faz |
|---|---|
| `curl ... -o awscliv2.zip` | Baixa o instalador e salva com o nome indicado em `-o` |
| `unzip awscliv2.zip` | Descompacta, criando a pasta `aws` |
| `sudo ./aws/install` | Executa o instalador (precisa de `sudo` para gravar em `/usr/local`) |
| `aws help` | Confirma que a CLI funciona (saio com `q`) |

> **Lição:** o instalador coloca os arquivos em `/usr/local/aws-cli` e cria um link simbólico em `/usr/local/bin`, por isso o comando `aws` fica disponível em qualquer pasta.

### 7. Configurando a AWS CLI
```bash
aws configure
```

O comando pergunta, em sequência:

| Pergunta | O que usei |
|---|---|
| Access Key ID | (em branco, as credenciais foram colocadas depois no arquivo) |
| Secret Access Key | (em branco) |
| Região padrão | `us-west-2` |
| Formato de saída | `json` |

Depois, editei o arquivo de credenciais para colar as credenciais fornecidas pelo lab:

```bash
sudo nano ~/.aws/credentials
```

```ini
[default]
aws_access_key_id=<sua-access-key>
aws_secret_access_key=<sua-secret-key>
aws_session_token=<seu-session-token>
```

> **Lição:** as credenciais do lab são **temporárias** e por isso incluem o **`aws_session_token`**. Credenciais de longa duração não têm esse campo.

> 🔐 **Segurança:** **nunca** publique chaves de acesso, chaves secretas ou tokens em repositórios. Neste README usei apenas *placeholders*, e o arquivo `~/.aws/credentials` não deve ir para o GitHub em hipótese alguma.

### 8. Testando a CLI na prática
```bash
aws ec2 describe-instance-attribute --instance-id <id-da-instancia> --attribute instanceType
```

O retorno em **JSON** mostrou o ID da instância e o tipo (`t3.micro`), comprovando que a CLI conseguiu se autenticar e consultar o serviço EC2.

> **Lição:** a AWS CLI permite fazer pelo terminal **tudo o que o console faz**, o que abre a porta para **automação e scripts**. O ID da instância pode ser pego no console, em EC2 → Instances.

---

## 🛠️ Comandos praticados

| Comando | Função |
|---|---|
| `yum check-update` | Lista atualizações disponíveis |
| `yum update --security` | Aplica atualizações de segurança |
| `yum upgrade` | Atualiza todos os pacotes |
| `yum install <pacote>` | Instala um pacote |
| `yum history list / info / undo` | Consulta e reverte transações |
| `python3 --version`, `pip3 --version` | Confere pré-requisitos |
| `curl -o` | Baixa um arquivo da internet |
| `unzip` | Descompacta um `.zip` |
| `sudo ./aws/install` | Instala a AWS CLI |
| `aws help` | Ajuda da AWS CLI |
| `aws configure` | Configura região, formato e credenciais |
| `aws ec2 describe-instance-attribute` | Consulta atributos de uma instância EC2 |

## ✅ Boas práticas que levo daqui

- **Manter o sistema atualizado**, priorizando atualizações de **segurança**
- **Conferir o que será atualizado** (`check-update`) antes de aplicar
- Usar o **histórico do gerenciador** para auditar e reverter mudanças
- **Verificar pré-requisitos** antes de instalar um software
- **Nunca expor credenciais** em código, repositórios ou prints
- Preferir **credenciais temporárias** e permissões mínimas
- **Encerrar o laboratório** ao final para liberar os recursos

---


## 📚 Próximos passos

- Estudar `yum remove`, `yum search`, `yum info` e `yum list installed`
- Explorar **perfis** da AWS CLI (`aws configure --profile`)
- Praticar outros comandos (`aws s3 ls`, `aws ec2 describe-instances`)
- Entender **IAM** e o princípio do menor privilégio para chaves de acesso
- Documentar os próximos labs do re/Start neste repositório

---

## 👤 Autor

**Werther Crisley**
🔗 [GitHub](https://github.com/WertherCrisley)