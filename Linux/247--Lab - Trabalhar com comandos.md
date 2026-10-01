# 🔧 AWS re/Start — Trabalhando com Comandos (`tee`, `sort`, `grep`, `cut`, `sed` e pipe)

Laboratório prático do programa **AWS re/Start** em que, conectado por **SSH** a uma instância **Amazon Linux (EC2)**, pratiquei comandos de **manipulação de texto** e o **operador pipe (`|`)**, que permite combinar comandos simples para resolver tarefas maiores.

![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white)
![Linux](https://img.shields.io/badge/Amazon_Linux-CLI-232F3E?logo=linux&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-shell-4EAA25?logo=gnubash&logoColor=white)
![Status](https://img.shields.io/badge/status-concluído-success)

---

## 🎯 Objetivos do laboratório

- Usar o **`tee`** para direcionar a saída para um arquivo
- Usar o **`sort`** para reorganizar o conteúdo de um arquivo `.csv`
- Usar o **`cut`** para extrair partes do conteúdo de um arquivo
- Usar o **`sed`** para substituir texto
- Usar o **operador pipe (`|`)**

## 🧩 O que eu fiz

1. **Conectei por SSH** à instância Amazon Linux (`ec2-user`)
2. **Gravei a saída do `hostname`** na tela e em um arquivo com `tee`
3. **Criei o `test.csv`** com `cat >` e **ordenei** o conteúdo com `sort`
4. **Pesquisei "Paris"** no arquivo com `grep`
5. **Criei o `cities.csv`** e **extraí a primeira coluna** com `cut`
6. **Fiz o desafio adicional:** substituí vírgulas por pontos com `sed`

---

## 🧠 O que eu aprendi

### 1. Redirecionando a saída com `tee`
```bash
hostname | tee file1.txt
ls
```

O `tee` lê a **entrada padrão** e grava **na tela e em um arquivo ao mesmo tempo**. Aqui, o nome do host apareceu no terminal e também ficou salvo no `file1.txt`.

> **Lição:** diferente do `>`, que só manda a saída para o arquivo, o `tee` também a **mostra na tela**. Para **acrescentar** em vez de sobrescrever, existe o `tee -a`.

### 2. O operador pipe (`|`)
O pipe pega a **saída de um comando** e a entrega como **entrada do próximo**:

```
comando1 | comando2 | comando3
```

> **Lição:** é a ideia central da linha de comando do Linux: cada ferramenta faz **uma coisa bem feita**, e juntas resolvem problemas complexos.

### 3. Criando arquivos com `cat >`
```bash
cat > test.csv
Factory, 1, Paris
Store, 2, Dubai
Factory, 3, Brasilia
Store, 4, Algiers
Factory, 5, Tokyo
# CTRL+D para finalizar e salvar
```

> **Lição:** o `cat >` cria um arquivo a partir do que eu digito, e o **`CTRL+D`** sinaliza o fim da entrada.

### 4. Ordenando com `sort`
```bash
sort test.csv
```

Resultado:

```
Factory, 1, Paris
Factory, 3, Brasilia
Factory, 5, Tokyo
Store, 2, Dubai
Store, 4, Algiers
```

Sem opções, o `sort` ordena **linha a linha, em ordem alfabética**. Por isso as linhas `Factory` vieram antes das `Store`, e dentro de cada grupo os números seguiram a ordem.

> 💡 **Detalhe importante:** a ordenação padrão é **de texto**, e não numérica. Com números de um dígito o resultado parece numérico, mas com valores como `10` e `2` poderia sair fora do esperado. Para ordenar por número em uma coluna específica, o caminho é algo como `sort -t',' -k2 -n test.csv`.

### 5. Pesquisando com `grep`
O lab usou este comando para achar a fábrica de Paris:

```bash
find | grep Paris test.csv
# Factory, 1, Paris
```

> 💡 **Observação:** quando o `grep` recebe um **nome de arquivo** (`test.csv`), ele lê **o arquivo** e ignora o que vem pelo pipe. Ou seja, o resultado seria o mesmo com `grep Paris test.csv`. Para realmente usar o pipe, o `grep` precisa ficar **sem arquivo**, lendo da entrada: por exemplo, `cat test.csv | grep Paris` ou `find . | grep Paris` (que procura "Paris" em **nomes de arquivos**).

### 6. Extraindo colunas com `cut`
```bash
cut -d ',' -f 1 cities.csv
```

| Opção | Significado |
|---|---|
| `-d ','` | Define o **delimitador** (aqui, a vírgula) |
| `-f 1` | Seleciona o **campo** (coluna) número 1 |

Resultado: ficaram só os nomes das cidades (Dallas, Seattle, Los Angeles, Atlanta, New York), sem os estados.

> **Lição:** o `cut` é perfeito para extrair colunas de arquivos CSV e de logs.

### 7. Substituindo texto com `sed` (desafio adicional)
```bash
sed 's/,/./' cities.csv
sed 's/,/./' test.csv
```

Formato geral: `sed 's/texto-antigo/texto-novo/' arquivo`

Resultado de exemplo: `Dallas. Texas` e `Factory. 1, Paris`.

> **Lição:** sem a opção `g` (global), o `sed` troca só a **primeira ocorrência em cada linha**, e por isso a segunda vírgula de `Factory, 1, Paris` ficou intacta. Com `s/,/./g`, todas seriam trocadas.

> 💡 O `sed` **mostra** o resultado na tela, mas **não altera o arquivo original**. Para editar o arquivo direto, existe a opção `-i`, mas ela exige cuidado, pois não dá para desfazer.

---

## 🛠️ Comandos praticados

| Comando | Função |
|---|---|
| `hostname` | Mostra o nome do host |
| `tee <arquivo>` | Grava a saída na tela e em um arquivo |
| `\|` (pipe) | Passa a saída de um comando como entrada do próximo |
| `cat > <arquivo>` | Cria um arquivo a partir do que é digitado |
| `sort` | Ordena linhas |
| `grep <texto> <arquivo>` | Procura um texto |
| `find` | Lista arquivos e pastas |
| `cut -d ',' -f 1` | Extrai uma coluna de um arquivo delimitado |
| `sed 's/a/b/'` | Substitui a primeira ocorrência de um texto por linha |

## ✅ Boas práticas que levo daqui

- **Combinar comandos simples** com pipes em vez de procurar um comando "mágico"
- **Testar o comando no terminal primeiro** (`sed` e `cut` só mostram o resultado) antes de alterar arquivos de verdade
- Lembrar que o **`sort` padrão é alfabético**, e usar `-n` quando for numérico
- Entender **de onde cada comando lê** (arquivo ou entrada padrão), como no caso do `grep`
- **Encerrar o laboratório** ao final para liberar os recursos

---

## 📚 Próximos passos

- Explorar `awk` para processar colunas com mais poder
- Praticar `sort -u`, `uniq`, `wc` e `head`/`tail`
- Usar `grep -i`, `-r` e `-c` em arquivos de log
- Aprender expressões regulares básicas para usar no `sed` e no `grep`
- Documentar os próximos labs do re/Start neste repositório

---

## 👤 Autor

**Werther Crisley**
🔗 [GitHub](https://github.com/WertherCrisley)