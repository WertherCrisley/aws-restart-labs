# 🔐 AWS re/Start — Gerenciando Permissões de Arquivos no Linux

Laboratório prático do programa **AWS re/Start** em que, conectado por **SSH** a uma instância **Amazon Linux (EC2)**, ajustei a **propriedade** (dono e grupo) de pastas de uma empresa fictícia com `chown` e aprendi a controlar quem pode **ler, gravar e executar** arquivos com `chmod`, nos modos **simbólico** e **absoluto**.

![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white)
![Linux](https://img.shields.io/badge/Amazon_Linux-CLI-232F3E?logo=linux&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-shell-4EAA25?logo=gnubash&logoColor=white)
![Status](https://img.shields.io/badge/status-concluído-success)

---

## 🎯 Objetivos do laboratório

- Alterar as permissões de pastas e arquivos para corresponder à estrutura de grupos
- Modificar as permissões de arquivos para um usuário
- Atualizar a estrutura de pastas da empresa

## 🧩 O que eu fiz

1. **Conectei por SSH** à instância Amazon Linux (`ec2-user`)
2. **Alterei a propriedade** das pastas da empresa (dono e grupo) com `chown`
3. **Validei** o resultado com `ls -laR`
4. **Criei dois arquivos** no Vim e alterei suas permissões com `chmod` (modo simbólico e modo absoluto)
5. **Atribuí a propriedade** das pastas `Shipping` e `Sales` aos responsáveis e aos grupos correspondentes

---

## 🧠 O que eu aprendi

### 1. Todo arquivo tem dono, grupo e permissões
Ao rodar `ls -l`, cada linha mostra algo assim:

```
-rwxrw-r-- 1 root root 0 Aug 10 13:37 absolute_mode_file
```

| Parte | Significado |
|---|---|
| `-` ou `d` | Tipo: arquivo comum ou diretório |
| `rwx` | Permissões do **dono** (user) |
| `rw-` | Permissões do **grupo** (group) |
| `r--` | Permissões dos **outros** (others) |
| 1º `root` | **Dono** do arquivo |
| 2º `root` | **Grupo** do arquivo |

E as letras significam: **r** (read, ler), **w** (write, gravar) e **x** (execute, executar).

### 2. Alterando dono e grupo: `chown`
```bash
sudo chown -R mjackson:Personnel /home/ec2-user/companyA   # CEO como dono, grupo Personnel
sudo chown -R ljuan:HR HR                                  # gerente de RH como dono, grupo HR
sudo chown -R mmajor:Finance HR/Finance                    # gerente financeira como dona, grupo Finance
```

- **Formato:** `chown <dono>:<grupo> <caminho>`
- **`-R`**: aplica de forma **recursiva**, ou seja, a todo o conteúdo da pasta

> **Lição:** a **ordem importa**. Primeiro apliquei a propriedade geral na pasta principal (`companyA`) e depois ajustei as subpastas mais específicas, que **sobrescreveram** a regra mais ampla. Vale ir **do geral para o específico**.

Para validar:

```bash
ls -laR
```

### 3. Alterando permissões: `chmod`
O `chmod` tem dois modos.

#### Modo simbólico (letras e símbolos)
```bash
sudo vi symbolic_mode_file          # cria o arquivo (ESC, depois :wq para salvar e sair)
sudo chmod g+w symbolic_mode_file   # dá permissão de gravação ao grupo
```

| Parte | Significado |
|---|---|
| `u` / `g` / `o` / `a` | **u**ser (dono), **g**roup (grupo), **o**thers (outros), **a**ll (todos) |
| `+` / `-` / `=` | Adiciona / remove / define exatamente |
| `r` / `w` / `x` | Leitura / gravação / execução |

> `g+w` = "dar permissão de **gravação (w)** ao **grupo (g)**".

#### Modo absoluto (números)
```bash
sudo vi absolute_mode_file
sudo chmod 764 absolute_mode_file
```

Cada permissão tem um valor, e eles se somam:

| Permissão | Valor |
|---|---|
| Leitura (`r`) | 4 |
| Gravação (`w`) | 2 |
| Execução (`x`) | 1 |

O `764` fica assim:

| | Dono | Grupo | Outros |
|---|---|---|---|
| **Número** | 7 | 6 | 4 |
| **Cálculo** | 4+2+1 | 4+2 | 4 |
| **Permissões** | `rwx` | `rw-` | `r--` |

> **Lição:** em `764`, o **dono** pode ler, gravar e executar, o **grupo** pode ler e gravar, e **os outros** só podem ler.

Para confirmar:

```bash
ls -l
```

### 4. Atribuindo responsáveis às pastas
```bash
sudo chown -R eowusu:Shipping Shipping
sudo chown -R nwolf:Sales Sales

ls -laR Shipping
ls -laR Sales
```

> **Lição:** combinar **dono + grupo** em cada pasta é uma forma simples de organizar o acesso por equipe, ligando o que aprendi neste lab com a criação de usuários e grupos do lab anterior.

### 5. Detalhe sobre o `sudo`
Criei os arquivos com `sudo vi`, então eles ficam com **dono `root`**. Em um cenário real, vale prestar atenção em **quem é o dono** de arquivos criados com `sudo` para não gerar problemas de acesso depois.

---

## 🛠️ Comandos praticados

| Comando | Função |
|---|---|
| `pwd` | Confirma o diretório atual |
| `cd <pasta>` | Muda de diretório |
| `chown -R <dono>:<grupo> <caminho>` | Altera dono e grupo recursivamente |
| `chmod g+w <arquivo>` | Modo simbólico: concede gravação ao grupo |
| `chmod 764 <arquivo>` | Modo absoluto: define permissões numéricas |
| `ls -l` | Lista com permissões, dono e grupo |
| `ls -laR` | Lista tudo, em detalhes e recursivamente |
| `sudo vi <arquivo>` | Cria/edita um arquivo como administrador |

## ✅ Boas práticas que levo daqui

- Aplicar o **princípio do menor privilégio**: dar só as permissões necessárias
- **Organizar acessos por grupos** em vez de usuário por usuário
- Usar **`-R` com cuidado**, pois afeta tudo dentro da pasta
- Ir **do geral para o específico** ao aplicar `chown -R` em estruturas aninhadas
- **Validar sempre** com `ls -l` ou `ls -laR` depois de alterar permissões
- Evitar permissões amplas demais, como dar escrita para "outros"
- **Encerrar o laboratório** ao final para liberar os recursos

---


## 📚 Próximos passos

- Praticar mais combinações do `chmod` (`u+x`, `o-r`, `755`, `644`, `600`)
- Entender permissões em **diretórios** (o `x` permite entrar na pasta)
- Estudar `chgrp`, `umask` e permissões especiais (SUID, SGID e *sticky bit*)
- Documentar os próximos labs do re/Start neste repositório

---

## 👤 Autor

**Werther Crisley**
🔗 [GitHub](https://github.com/WertherCrisley)