# ✏️ AWS re/Start — Edição de Arquivos no Linux (Vim e nano)

Laboratório prático do programa **AWS re/Start** em que, conectado por **SSH** a uma instância **Amazon Linux (EC2)**, aprendi a criar, editar e salvar arquivos pela linha de comando usando dois editores de texto: o **Vim** e o **nano**.

![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white)
![Linux](https://img.shields.io/badge/Amazon_Linux-CLI-232F3E?logo=linux&logoColor=white)
![Vim](https://img.shields.io/badge/Vim-editor-019733?logo=vim&logoColor=white)
![Status](https://img.shields.io/badge/status-concluído-success)

---

## 🎯 Objetivos do laboratório

- Usar o **vimtutor** para aprender os fundamentos do Vim
- Criar e editar arquivos no **Vim**
- Criar e editar arquivos no **nano**

## 🧩 O que eu fiz

1. **Conectei por SSH** à instância "Command Host" (Amazon Linux)
2. **Rodei o `vimtutor`**, o tutorial interativo do Vim, e completei as lições
3. **Criei e editei um arquivo no Vim** (`helloworld`), inserindo texto, salvando e saindo
4. **Reabri o arquivo**, adicionei uma linha e saí **sem salvar**, para entender a diferença entre os comandos de saída
5. **Testei comandos extras do Vim** (`dd`, `u` e `:w`)
6. **Criei e editei um arquivo no nano** (`cloudworld`) e confirmei que foi salvo

---

## 🧠 O que eu aprendi

### 1. Vimtutor: aprendendo Vim praticando
O `vimtutor` abre um tutorial interativo dentro do próprio Vim, com lições que ensinam desde mover o cursor até editar texto:

```bash
vimtutor
```

Se o comando não funcionar, o Vim pode não estar instalado:

```bash
sudo yum install vim
```

Para sair do tutorial sem salvar nada:

```
:q!
```

### 2. O Vim trabalha com modos
A grande diferença do Vim para outros editores é que ele tem **modos**. Os que usei:

| Modo | Como entrar | Para que serve |
|---|---|---|
| **Normal** | `ESC` | Navegar e executar comandos |
| **Inserção** | `i` | Digitar texto |
| **Comando** | `:` | Salvar, sair e outras ações |

> **Lição:** no Vim, **não dá para simplesmente começar a digitar**. É preciso entrar no modo de inserção com `i` primeiro. O canto inferior esquerdo do terminal mostra em qual modo estou.

### 3. Criando, salvando e saindo no Vim
```bash
vim helloworld
```

1. Digitei `i` para entrar no modo de inserção
2. Escrevi duas linhas de texto
3. Pressionei `ESC` para voltar ao modo normal
4. Digitei `:wq` para **salvar e sair**

### 4. A diferença entre `:wq` e `:q!`
Depois, abri o arquivo de novo, adicionei uma terceira linha e saí com **`:q!`** (sair **sem salvar**). Ao reabrir o `helloworld`, a linha nova **não estava lá**: só o conteúdo salvo antes tinha sido mantido.

| Comando | O que faz |
|---|---|
| `:wq` | Salva e sai |
| `:w` | Salva sem sair |
| `:q!` | Sai **descartando** as alterações |

> **Lição:** `:q!` é útil para desistir de mudanças, mas é perigoso se eu esquecer de salvar algo importante.

### 5. Comandos extras do Vim (desafio adicional)
| Comando | O que faz |
|---|---|
| `dd` | Exclui a linha inteira |
| `u` | Desfaz o último comando |
| `:w` | Salva as alterações sem encerrar o editor |

### 6. nano: um editor mais direto
```bash
nano cloudworld
```

No **nano não existe modo de inserção**: é só começar a digitar. Os comandos aparecem na parte de baixo da tela, e o `^` significa a tecla **CTRL**.

| Atalho | O que faz |
|---|---|
| `CTRL+O` | Salva o arquivo (Enter confirma o nome) |
| `CTRL+X` | Sai do nano |

Depois reabri o arquivo com `nano cloudworld` para confirmar que o texto tinha sido salvo corretamente.

### 7. Vim ou nano?
| | Vim | nano |
|---|---|---|
| Curva de aprendizado | Maior | Menor |
| Modos | Sim (normal, inserção, comando) | Não |
| Atalhos visíveis na tela | Não | Sim |
| Ideal para | Edição rápida e eficiente, depois de aprender | Edições simples e iniciantes |

> **Lição:** saber usar pelo menos um editor de terminal é essencial, porque em servidores na nuvem normalmente **não há interface gráfica**. Aprender o básico dos dois me deixa preparado para qualquer servidor.

---

## 🛠️ Comandos e atalhos praticados

| Item | Função |
|---|---|
| `vimtutor` | Tutorial interativo do Vim |
| `sudo yum install vim` | Instala o Vim |
| `vim <arquivo>` | Abre ou cria um arquivo no Vim |
| `i` | Entra no modo de inserção (Vim) |
| `ESC` | Volta ao modo normal (Vim) |
| `:wq` | Salva e sai (Vim) |
| `:w` | Salva sem sair (Vim) |
| `:q!` | Sai sem salvar (Vim) |
| `dd` | Exclui a linha atual (Vim) |
| `u` | Desfaz a última ação (Vim) |
| `nano <arquivo>` | Abre ou cria um arquivo no nano |
| `CTRL+O` | Salva (nano) |
| `CTRL+X` | Sai (nano) |

## ✅ Boas práticas que levo daqui

- **Salvar com frequência** (`:w` no Vim, `CTRL+O` no nano) em vez de deixar para o final
- **Conferir o modo atual** no Vim antes de digitar
- Usar `:q!` com **cuidado**, só quando eu realmente quiser descartar as mudanças
- **Reabrir o arquivo** depois de editar para confirmar que o conteúdo foi salvo
- Praticar com o **vimtutor** antes de editar arquivos importantes
- **Encerrar o laboratório** ao final para liberar os recursos

---

## 📚 Próximos passos

- Praticar mais no `vimtutor` até ficar natural
- Aprender buscas e substituições no Vim (`/termo`, `:%s/antigo/novo/g`)
- Editar arquivos de configuração do sistema com `sudo`
- Documentar os próximos labs do re/Start neste repositório

---

## 👤 Autor

**Werther Crisley**
🔗 [GitHub](https://github.com/WertherCrisley)