# ⌨️ AWS re/Start — Linha de Comando do Linux

Laboratório prático do programa **AWS re/Start** em que me conectei por **SSH** a uma instância **Amazon Linux (EC2)** e pratiquei comandos para conhecer o sistema e a sessão atual, além de técnicas para **pesquisar e reutilizar comandos** do histórico do bash.

![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white)
![Linux](https://img.shields.io/badge/Amazon_Linux-CLI-232F3E?logo=linux&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-shell-4EAA25?logo=gnubash&logoColor=white)
![Status](https://img.shields.io/badge/status-concluído-success)

---

## 🎯 Objetivos do laboratório

- Executar comandos para obter conhecimento do **sistema e da sessão atuais**
- **Pesquisar e executar** comandos bash anteriores

## 🧩 O que eu fiz

1. **Conectei por SSH** a uma instância Amazon Linux (`ec2-user`)
2. **Usei o autocompletar** com a tecla Tab
3. **Consultei informações** de usuário, host, tempo de atividade e usuários logados
4. **Verifiquei data e hora** de outros fusos horários
5. **Explorei o comando `cal`** em diferentes formatos
6. **Consultei o ID e os grupos** do usuário
7. **Usei o histórico**, a pesquisa reversa (CTRL+R) e o `!!` para reaproveitar comandos

---

## 🧠 O que eu aprendi

### 1. Conexão por SSH com par de chaves
A conexão foi feita com a chave do laboratório (`.pem` no macOS/Linux ou `.ppk` no Windows com PuTTY), sem precisar de senha:

```bash
cd ~/Downloads
chmod 400 labsuser.pem
ssh -i labsuser.pem ec2-user@<public-ip>
```

> **Lição:** o `chmod 400` deixa a chave somente leitura para o dono, e o SSH exige isso para aceitar o arquivo.

### 2. Autocompletar com Tab
Digitando `whoa` e apertando **Tab**, o terminal completa para `whoami`. Parece pouco, mas **economiza digitação e evita erros** de escrita em comandos longos.

### 3. Comandos para conhecer o sistema e a sessão

| Comando | O que mostra |
|---|---|
| `whoami` | Usuário atual (`ec2-user`) |
| `hostname -s` | Nome curto do host da máquina |
| `uptime -p` | Há quanto tempo o sistema está ligado, em formato legível |
| `who -H -a` | Usuários logados, com colunas como nome, linha, hora, ociosidade, PID, comentário e saída |
| `id ec2-user` | ID do usuário, ID do grupo e grupos aos quais ele pertence |

> **Para que serve na prática:** esses comandos ajudam a descobrir rapidamente **quem sou, em qual máquina estou e há quanto tempo ela está ativa**, o que é útil em **troubleshooting**.

### 4. Data e hora de outros fusos horários
Usando a variável de ambiente `TZ` antes do comando `date`, dá para ver a hora de outro lugar sem alterar a configuração do sistema:

```bash
TZ=America/New_York date
TZ=America/Los_Angeles date
```

> **Lição:** a variável vale **só para aquele comando**. Se a hora do sistema estiver errada, a saída também ficará errada.

### 5. Calendário no terminal
```bash
cal -j    # calendário com dias numerados em sequência (data juliana)
cal -s    # semana começando no domingo
cal -m    # semana começando na segunda-feira
```

O formato **juliano** conta os dias de forma contínua, em vez de reiniciar em 1 a cada mês. E, como em qualquer comando, as outras opções estão na **man page** (`man cal`).

### 6. Histórico de comandos
```bash
history    # lista os comandos digitados anteriormente
```

### 7. Pesquisa reversa com CTRL+R
Pressionando **CTRL+R** e digitando parte de um comando (por exemplo, `TZ`), o bash localiza um uso antigo. Com **Tab**, ele passa para a linha de comando, onde dá para **editar com as setas** e executar de novo.

### 8. Repetir o último comando com `!!`
```bash
date
!!        # executa o último comando novamente
```

> **Lição:** `!!` é um atalho muito útil, por exemplo, quando esqueço de usar `sudo` e preciso repetir o comando anterior.

---

## 🛠️ Comandos e atalhos praticados

| Item | Função |
|---|---|
| `Tab` | Autocompleta comandos |
| `whoami` | Mostra o usuário atual |
| `hostname -s` | Mostra o nome curto do host |
| `uptime -p` | Mostra o tempo de atividade |
| `who -H -a` | Mostra usuários logados e detalhes |
| `TZ=... date` | Mostra data e hora em outro fuso |
| `cal -j / -s / -m` | Exibe o calendário em formatos diferentes |
| `id` | Mostra ID de usuário e grupos |
| `history` | Lista o histórico de comandos |
| `CTRL+R` | Pesquisa reversa no histórico |
| `!!` | Repete o último comando |

## ✅ Boas práticas que levo daqui

- **Usar Tab** para ganhar velocidade e reduzir erros
- **Reaproveitar o histórico** em vez de redigitar comandos longos
- Conferir **usuário, host e uptime** no início de um troubleshooting
- **Consultar a man page** quando precisar de mais opções de um comando
- **Encerrar o laboratório** ao final para liberar os recursos

---

## 📚 Próximos passos

- Praticar navegação e manipulação de arquivos (`ls`, `cd`, `pwd`, `cat`, `cp`, `mv`)
- Estudar permissões de arquivos e usuários no Linux
- Aprender a encadear comandos com pipes (`|`) e redirecionamentos
- Documentar os próximos labs do re/Start neste repositório

---

## 👤 Autor

**Werther Crisley**
🔗 [GitHub](https://github.com/WertherCrisley)