# 🐧 AWS re/Start — Introdução a uma AMI no Amazon Linux

Laboratório prático do programa **AWS re/Start** em que me conectei por **SSH** a uma instância **Amazon EC2** com **Amazon Linux** e explorei o sistema de ajuda do Linux, as **man pages**.

![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white)
![Linux](https://img.shields.io/badge/Amazon_Linux-CLI-232F3E?logo=linux&logoColor=white)
![SSH](https://img.shields.io/badge/SSH-acesso_remoto-4EAA25)
![Status](https://img.shields.io/badge/status-concluído-success)

---

## 🎯 Objetivo do laboratório

Reforçar o uso básico da **linha de comando (CLI)** e criar uma base sólida para continuar aprendendo comandos e recursos do shell Linux.

## 🧩 O que eu fiz

1. **Conectei por SSH** a uma instância Amazon Linux (o "host de comando") que estava em uma sub-rede pública
2. **Usei o comando `man`** para abrir as páginas do manual
3. **Naveguei** pelas man pages e identifiquei os principais cabeçalhos
4. **Analisei a seção DESCRIPTION**, prestando atenção nos números de seção
5. **Saí** das man pages e **encerrei** o laboratório

## 🏗️ Ambiente criado pelo lab

- **Amazon EC2** (host de comando) em uma **sub-rede pública**
- **Amazon VPC** com a sub-rede pública
- Instância do tipo **t3.micro**

---

## 🧠 O que eu aprendi

### 1. Como se conectar a uma instância por SSH
O **SSH (Secure Shell)** permite acessar o terminal de um servidor remoto de forma segura. A autenticação foi feita com um **par de chaves** (arquivo `.pem` no macOS/Linux ou `.ppk` no Windows com PuTTY), então **não precisei digitar senha**.

No macOS/Linux, o passo a passo foi:

```bash
# 1. Ir até a pasta onde a chave foi baixada
cd ~/Downloads

# 2. Deixar a chave somente leitura (exigência do SSH)
chmod 400 labsuser.pem

# 3. Conectar na instância usando o IP público
ssh -i labsuser.pem ec2-user@<public-ip>
```

No Windows, usei o **PuTTY** com o arquivo `labsuser.ppk` e o **Public IP** da instância.

> **Lição:** o SSH recusa chaves privadas com permissões abertas demais. O `chmod 400` garante que só o dono consiga ler a chave. Também aprendi que, na primeira conexão, preciso confirmar com `yes` a identidade do servidor.

### 2. O usuário padrão do Amazon Linux é o `ec2-user`
Em instâncias Amazon Linux, o login é feito com o usuário **`ec2-user`**, não com root, o que já reforça a ideia de usar o menor privilégio possível.

### 3. O comando `man` é o manual embutido do Linux
Em vez de procurar na internet toda hora, o próprio sistema tem documentação de cada comando:

```bash
man man      # abre o manual do próprio comando man
```

- **Setas para cima e para baixo**: navegar pelo texto
- **`q`**: sair das man pages

### 4. Como ler uma man page
Cada man page é organizada em cabeçalhos padronizados. Os principais que eu vi:

| Cabeçalho | Para que serve |
|---|---|
| **NAME** | Nome do comando e uma descrição curta |
| **SYNOPSIS** | Como o comando deve ser escrito (sintaxe) |
| **DESCRIPTION** | Visão geral do que o comando faz |
| **OVERVIEW** | Resumo geral |
| **OPTIONS** | Opções e flags disponíveis |
| **EXAMPLES** | Exemplos de uso |
| **FILES** | Arquivos relacionados ao comando |
| **SEE ALSO** | Comandos e páginas relacionados |

Na seção **DESCRIPTION**, reparei nos **números de seção** do manual, que indicam a categoria da página (comandos, chamadas de sistema, arquivos de configuração etc.).

### 5. Busca dentro das man pages
Um dos objetivos do lab era demonstrar o **atributo de busca** das man pages. Dentro do `man`, dá para pesquisar um termo digitando `/` seguido da palavra e apertando Enter (e `n` para ir ao próximo resultado). Isso economiza muito tempo em manuais grandes.

---

## 🛠️ Conceitos e ferramentas praticados

| Conceito | O que significa |
|---|---|
| EC2 | Servidor virtual na nuvem |
| AMI | Template usado para iniciar a instância (aqui, Amazon Linux) |
| t3.micro | Tipo de instância com 1 vCPU e 1 GiB de memória neste lab |
| VPC e sub-rede pública | Rede virtual onde a instância roda, com acesso à internet |
| SSH | Protocolo de acesso remoto seguro |
| Par de chaves (.pem / .ppk) | Autenticação sem senha |
| PuTTY | Cliente SSH para Windows |
| `chmod 400` | Permissão somente leitura para o dono do arquivo |
| `man` | Manual de comandos do Linux |

## ✅ Boas práticas que levo daqui

- **Proteger a chave privada** com permissões restritas e nunca compartilhá-la
- **Consultar a man page** antes de buscar soluções externas
- Ler **SYNOPSIS** e **EXAMPLES** primeiro para entender rápido como usar um comando
- **Encerrar o laboratório** ao final para liberar os recursos

---

## 📚 Próximos passos

- Praticar comandos de navegação e arquivos (`ls`, `cd`, `pwd`, `cat`)
- Estudar permissões de arquivos no Linux
- Explorar o **AWS Systems Manager Session Manager** como alternativa ao SSH
- Documentar os próximos labs do re/Start neste repositório

---

## 👤 Autor

**Werther Crisley**
🔗 [GitHub](https://github.com/WertherCrisley)