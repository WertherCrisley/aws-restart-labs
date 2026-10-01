# 👥 AWS re/Start — Gerenciamento de Usuários e Grupos no Linux

Laboratório prático do programa **AWS re/Start** em que, conectado por **SSH** a uma instância **Amazon Linux (EC2)**, criei **usuários**, organizei-os em **grupos** conforme o cargo de cada um e testei **login com outros usuários**, vendo na prática o que é um **sudoer** e como ações bloqueadas ficam registradas em log.

![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white)
![Linux](https://img.shields.io/badge/Amazon_Linux-CLI-232F3E?logo=linux&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-shell-4EAA25?logo=gnubash&logoColor=white)
![Status](https://img.shields.io/badge/status-concluído-success)

---

## 🎯 Objetivos do laboratório

- Criar novos usuários com uma senha padrão
- Criar grupos e atribuir os usuários apropriados
- Fazer login como diferentes usuários

## 🧩 O que eu fiz

1. **Conectei por SSH** à instância Amazon Linux (`ec2-user`)
2. **Criei 10 usuários** (um para cada funcionário da empresa fictícia) e defini a senha inicial de cada um
3. **Validei** a criação lendo o arquivo `/etc/passwd`
4. **Criei 6 grupos** (`Sales`, `HR`, `Finance`, `Shipping`, `Managers` e `CEO`)
5. **Adicionei cada usuário** aos grupos certos, de acordo com o cargo
6. **Validei** as participações lendo o arquivo `/etc/group`
7. **Fiz login com outro usuário** (`su`) e testei suas permissões
8. **Consultei o log** `/var/log/secure` para ver o registro de uma ação não permitida

---

## 🧠 O que eu aprendi

### 1. Criando usuários: `useradd` e `passwd`
Cada usuário foi criado em dois passos:

```bash
sudo useradd arosalez     # cria o usuário
sudo passwd arosalez      # define a senha (digitada duas vezes, sem aparecer na tela)
```

Repeti o processo para todos os usuários da tabela de funcionários. Para conferir, listei só os nomes de usuário do sistema:

```bash
sudo cat /etc/passwd | cut -d: -f1
```

> **Lição:** o arquivo **`/etc/passwd`** guarda a lista de usuários do sistema. O `cut -d: -f1` pega só a primeira coluna (o nome), deixando a saída bem mais legível.

### 2. Criando grupos: `groupadd`
```bash
sudo groupadd Sales
sudo groupadd HR
sudo groupadd Finance
sudo groupadd Shipping
sudo groupadd Managers
sudo groupadd CEO
```

Para conferir:

```bash
cat /etc/group
```

> **Lição:** o arquivo **`/etc/group`** guarda todos os grupos e seus membros. Percebi que **já existia um grupo para cada usuário criado**, porque o Linux cria automaticamente um grupo pessoal para cada novo usuário.

### 3. Adicionando usuários aos grupos: `usermod -a -G`
```bash
sudo usermod -a -G Sales arosalez
```

- **`-G`** define o grupo (ou grupos) adicional
- **`-a`** (*append*) **adiciona** sem remover o usuário dos outros grupos em que ele já está

> **Lição importante:** sem o `-a`, o `usermod -G` **substitui** os grupos secundários do usuário. O `-a` evita perder participações existentes.

### 4. Um usuário pode estar em vários grupos
Nem todo funcionário é gerente, mas todo gerente é funcionário. Por isso alguns usuários ficaram em mais de um grupo (por exemplo, um gerente de vendas está em `Sales` **e** em `Managers`). O resultado final no `/etc/group` mostrou cada grupo com a lista de seus membros.

| Grupo | Quem entrou |
|---|---|
| **Sales** | Gerente de vendas e representante de vendas |
| **HR** | Gerente de RH e especialista de RH |
| **Finance** | Gerente financeira e especialista financeiro |
| **Shipping** | Os três funcionários de expedição |
| **Managers** | Os gerentes (vendas, RH e financeiro) |
| **CEO** | O CEO |

Também adicionei o **`ec2-user`** a todos os grupos.

### 5. Trocando de usuário: `su`
```bash
su arosalez
```

Depois de informar a senha, passei a operar como outro usuário. O prompt mudou para `[arosalez@ec2-user]`, mas o diretório continuou sendo `/home/ec2-user`.

### 6. Permissões na prática
Como `arosalez`, tentei criar um arquivo na pasta do `ec2-user`:

```bash
touch myFile.txt
# touch: cannot touch 'myFile.txt': Permission denied
```

> **Lição:** um usuário comum **não pode gravar na pasta pessoal de outro usuário**. As permissões protegem os arquivos de cada um.

### 7. O que é um *sudoer*
Em seguida, tentei usar privilégios de administrador:

```bash
sudo touch myFile.txt
# arosalez is not in the sudoers file. This incident will be reported.
```

> **Lição:** **sudoers** são os usuários autorizados a executar comandos com direitos de root. Essa permissão deve ser concedida só a quem realmente precisa (**princípio do menor privilégio**).

### 8. Auditoria: tudo fica registrado em log
De volta ao `ec2-user` (comando `exit`), li o arquivo de log de segurança:

```bash
sudo cat /var/log/secure
```

No final do arquivo apareceu o registro da tentativa bloqueada, informando o usuário, o diretório, o usuário alvo (root) e o comando tentado.

> **Lição:** o uso de `sudo`, inclusive as tentativas **negadas**, fica registrado em **`/var/log/secure`**. Isso é essencial para **auditoria e segurança**.

---

## 🛠️ Comandos e arquivos praticados

| Item | Função |
|---|---|
| `pwd` | Mostra o diretório atual |
| `useradd <usuário>` | Cria um usuário |
| `passwd <usuário>` | Define ou altera a senha do usuário |
| `groupadd <grupo>` | Cria um grupo |
| `usermod -a -G <grupo> <usuário>` | Adiciona o usuário a um grupo sem remover dos outros |
| `su <usuário>` | Troca para outro usuário |
| `touch <arquivo>` | Cria um arquivo vazio |
| `sudo` | Executa um comando com privilégios de administrador |
| `exit` | Volta ao usuário anterior |
| `cat` | Exibe o conteúdo de um arquivo |
| `cut -d: -f1` | Extrai a primeira coluna separada por `:` |
| `/etc/passwd` | Lista de usuários do sistema |
| `/etc/group` | Lista de grupos e membros |
| `/var/log/secure` | Log de segurança (inclui uso de `sudo`) |

## ✅ Boas práticas que levo daqui

- **Organizar acessos por grupos** em vez de gerenciar permissões usuário por usuário
- Usar **`usermod -a -G`** (com o `-a`) para não perder grupos existentes
- Aplicar o **menor privilégio**: nem todo usuário deve ser sudoer
- **Trocar a senha inicial** no primeiro acesso (senhas padrão são temporárias)
- **Consultar os logs** (`/var/log/secure`) para auditar ações e tentativas de acesso
- **Conferir a grafia** dos nomes de usuário e grupos antes de aplicar os comandos
- **Encerrar o laboratório** ao final para liberar os recursos

> 🔐 Neste README não incluí a senha usada no lab. Em um ambiente real, senhas nunca devem ficar em repositórios públicos.

---

## 📚 Próximos passos

-  Estudar **permissões de arquivos** (`chmod`, `chown`) e como elas se relacionam com grupos
-  Aprender a configurar o arquivo **sudoers** com segurança (`visudo`)
-  Explorar `userdel`, `groupdel` e `chage` (expiração de senha)
-  Documentar os próximos labs do re/Start neste repositório

---

## 👤 Autor

**Werther Crisley**
🔗 [GitHub](https://github.com/WertherCrisley)