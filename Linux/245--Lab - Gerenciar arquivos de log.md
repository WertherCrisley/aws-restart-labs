# 🧾 AWS re/Start — Gerenciando Arquivos de Log no Linux (`less` e `lastlog`)

Laboratório prático do programa **AWS re/Start** em que, conectado por **SSH** a uma instância **Amazon Linux (EC2)**, revisei **logs de segurança** e o histórico de **últimos logins** dos usuários, entendendo como esses registros ajudam a **auditar e proteger** um servidor.

![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white)
![Linux](https://img.shields.io/badge/Amazon_Linux-CLI-232F3E?logo=linux&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-shell-4EAA25?logo=gnubash&logoColor=white)
![Status](https://img.shields.io/badge/status-concluído-success)

---

## 🎯 Objetivo do laboratório

Revisar o **`lastlog`** e as saídas do **log de segurança** da máquina Linux.

## 🧩 O que eu fiz

1. **Conectei por SSH** à instância Amazon Linux (`ec2-user`)
2. **Abri o arquivo de log de segurança** de amostra com `less`
3. **Analisei** as tentativas de acesso registradas (origem, resultado e porta)
4. **Saí** do visualizador com `q`
5. **Consultei o histórico de logins** de todos os usuários com `lastlog`

---

## 🧠 O que eu aprendi

### 1. Logs são o "diário" do sistema
O Linux registra eventos importantes em **arquivos de log**: quem tentou acessar, o que funcionou, o que falhou e quando. São essenciais para **segurança**, **auditoria** e **troubleshooting**.

### 2. O log de segurança (`secure`)
```bash
sudo less /tmp/log/secure
```

> **Observação:** em um sistema real, esse arquivo fica em **`/var/log/secure`**. O lab usa uma **cópia de amostra** em `/tmp/log/secure` para a atividade.

O arquivo lista eventos de autenticação. Nele, consegui identificar informações como:

- **De onde** veio a tentativa de acesso (**endereço IP**)
- Se a **autenticação falhou ou foi bem-sucedida**
- Qual **porta** foi usada

> **Lição:** muitas falhas de autenticação seguidas vindas do mesmo IP podem indicar uma tentativa de **ataque de força bruta**. O log é o primeiro lugar para investigar esse tipo de suspeita.

### 3. Navegando com `less`
O **`less`** é um visualizador de arquivos que carrega o conteúdo **aos poucos**, ideal para arquivos grandes como logs:

| Tecla | Ação |
|---|---|
| ⬆️ ⬇️ | Rolar linha por linha |
| `Espaço` | Avançar uma página |
| `/termo` | Pesquisar um texto |
| `n` | Ir para o próximo resultado da pesquisa |
| `q` | Sair |

> **Lição:** diferente do `cat`, o `less` não despeja o arquivo inteiro na tela, então dá para **navegar e pesquisar** com calma.

### 4. Últimos logins com `lastlog`
```bash
sudo lastlog
```

O comando lista **todos os usuários do sistema** e a **data e hora do login mais recente** de cada um. Vi que muitas contas de sistema (como `root`, `bin` e `daemon`) aparecem como **nunca logadas**.

> **Lição:** contas de sistema normalmente nunca fazem login interativo, e isso é esperado. Já uma conta de **pessoa** que nunca logou, ou que logou em um horário estranho, merece atenção.

### 5. Desafio adicional: que informações extrair para objetivos de negócio?
Pensando em como esses logs poderiam apoiar uma empresa:

| Informação do log | Utilidade para o negócio |
|---|---|
| IPs e falhas de autenticação | **Detectar ataques** e bloquear origens suspeitas |
| Horários e frequência de logins | Identificar **acessos fora do padrão** (por exemplo, de madrugada) |
| Usuários que nunca logaram ou estão inativos | **Limpar contas** desnecessárias e reduzir riscos |
| Histórico de quem acessou e quando | **Auditoria e conformidade**, comprovando quem fez o quê |
| Uso de `sudo` e ações negadas (visto no lab de usuários e grupos) | Controlar **privilégios** e rastrear ações administrativas |

> **Lição:** logs não servem só para corrigir problemas. Eles também ajudam a **prevenir incidentes**, comprovar conformidade e embasar **decisões de segurança**.

---

## 🛠️ Comandos praticados

| Comando | Função |
|---|---|
| `pwd` / `cd` | Confirma e muda o diretório atual |
| `sudo less <arquivo>` | Visualiza um arquivo de forma paginada |
| `q` | Sai do `less` |
| `sudo lastlog` | Mostra o último login de cada usuário |

## ✅ Boas práticas que levo daqui

- **Revisar logs regularmente**, e não só quando algo dá errado
- Ficar atento a **falhas repetidas de autenticação** e a IPs suspeitos
- Usar `less` para **explorar arquivos grandes** com segurança, sem alterá-los
- **Proteger os logs**: só administradores devem ter acesso
- **Desativar ou remover contas** que não são mais usadas
- **Encerrar o laboratório** ao final para liberar os recursos

---


## 📚 Próximos passos

- Praticar `tail -f`, `head` e `grep` para filtrar logs
- Explorar o `journalctl` (logs do systemd)
- Conhecer outros arquivos em `/var/log` (`messages`, `dmesg`, `httpd/`)
- Estudar o **Amazon CloudWatch Logs** para centralizar logs na nuvem
- Documentar os próximos labs do re/Start neste repositório

---

## 👤 Autor

**Werther Crisley**
🔗 [GitHub](https://github.com/WertherCrisley)