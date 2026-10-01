# 📁 AWS re/Start — Trabalhando com o Sistema de Arquivos no Linux

Laboratório prático do programa **AWS re/Start** em que, conectado por **SSH** a uma instância **Amazon Linux (EC2)**, criei uma estrutura de pastas e arquivos de uma empresa fictícia e depois a **reorganizei** copiando, movendo e excluindo diretórios, tudo pela linha de comando.

![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white)
![Linux](https://img.shields.io/badge/Amazon_Linux-CLI-232F3E?logo=linux&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-shell-4EAA25?logo=gnubash&logoColor=white)
![Status](https://img.shields.io/badge/status-concluído-success)

---

## 🎯 Objetivos do laboratório

- Criar uma estrutura de pastas
- Criar arquivos
- Copiar e mover arquivos e diretórios
- Excluir arquivos e diretórios

## 🧩 O que eu fiz

1. **Conectei por SSH** à instância Amazon Linux (`ec2-user`)
2. **Criei a estrutura inicial** da `CompanyA`, com as pastas `Finance`, `HR` e `Management` e seus arquivos `.csv`
3. **Validei** cada etapa com `ls`, `pwd` e `ls -laR`
4. **Reorganizei a estrutura**: copiei `Finance` para dentro de `HR` e removi a original
5. **Movi** a pasta `Management` para dentro de `HR`
6. **Criei** a pasta `Employees` dentro de `HR` e movi para ela os arquivos `Assessments.csv` e `TrialPeriod.csv`

---

## 🗂️ Estrutura criada

**Antes (Tarefa 2):**

```
/home/ec2-user/CompanyA/
├── Finance/
│   ├── ProfitAndLossStatements.csv
│   └── Salary.csv
├── HR/
│   ├── Assessments.csv
│   └── TrialPeriod.csv
└── Management/
    ├── Managers.csv
    └── Schedule.csv
```

**Depois (Tarefa 3):**

```
/home/ec2-user/CompanyA/
└── HR/
    ├── Employees/
    │   ├── Assessments.csv
    │   └── TrialPeriod.csv
    ├── Finance/
    │   ├── ProfitAndLossStatements.csv
    │   └── Salary.csv
    └── Management/
        ├── Managers.csv
        └── Schedule.csv
```

---

## 🧠 O que eu aprendi

### 1. Criando pastas e arquivos: `mkdir` e `touch`
```bash
mkdir CompanyA                  # cria a pasta principal
cd CompanyA                     # entra nela
mkdir Finance HR Management     # cria várias pastas de uma vez
touch Assessments.csv TrialPeriod.csv   # cria arquivos vazios
```

> **Lição:** o `mkdir` e o `touch` aceitam **vários nomes de uma vez**, o que economiza muito tempo.

### 2. Verificando o que fiz: `pwd` e `ls`
```bash
pwd          # mostra onde estou
ls           # lista o conteúdo do diretório atual
ls Management   # lista o conteúdo de outra pasta sem entrar nela
```

> **Lição:** conferir o resultado depois de cada etapa ajuda a achar erros cedo, antes que se acumulem.

### 3. Caminhos relativos
Dá para trabalhar em outra pasta sem precisar entrar nela:

```bash
touch Management/Managers.csv Management/Schedule.csv   # a partir da CompanyA
cd ../Finance                                            # sobe um nível e entra em Finance
cd ..                                                    # sobe um nível
```

| Símbolo | Significado |
|---|---|
| `.` | Diretório atual |
| `..` | Diretório pai (um nível acima) |

### 4. Listagem completa e recursiva: `ls -laR`
```bash
ls -laR
```

- **`-l`**: formato detalhado (permissões, dono, tamanho, data)
- **`-a`**: inclui arquivos ocultos
- **`-R`**: percorre as subpastas recursivamente

> **Lição:** é ótimo para **conferir a árvore inteira** de uma só vez.

### 5. Copiando pastas: `cp -r`
```bash
cp -r Finance HR      # copia a pasta Finance e todo o conteúdo para dentro de HR
ls HR/Finance         # confere a cópia
```

> **Lição:** para copiar **diretórios**, é preciso usar **`-r`** (recursivo). Sem ele, o `cp` só copia arquivos.

### 6. Removendo: `rmdir` e `rm`
Ao tentar remover a pasta `Finance` original com `rmdir`, deu erro:

```bash
rmdir Finance
# rmdir: failed to remove 'Finance/': Directory not empty
```

> **Lição:** o **`rmdir` só remove diretórios vazios**.

Existem dois caminhos:

```bash
# Opção 1: apagar os arquivos e depois a pasta vazia
rm Finance/ProfitAndLossStatements.csv Finance/Salary.csv
rmdir Finance

# Opção 2: apagar tudo de uma vez, recursivamente
rm -r Finance
```

Segui a opção 1, mais cuidadosa, e depois confirmei com `ls` que a pasta tinha sido removida.

> ⚠️ **Cuidado:** o `rm -r` apaga a pasta e tudo dentro dela **sem pedir confirmação e sem lixeira**. Vale conferir o caminho antes de executar.

### 7. Movendo e reorganizando: `mv`
```bash
mv Management HR                          # move a pasta Management para dentro de HR
cd HR
mkdir Employees                           # cria a pasta Employees
mv Assessments.csv TrialPeriod.csv Employees   # move os arquivos para Employees
ls . Employees                            # confere o resultado
```

> **Lição:** o `mv` serve tanto para **mover** quanto para **renomear**. Quando o último argumento é uma pasta existente, os itens são movidos para dentro dela.

### 8. Sempre confirmar a pasta atual
Antes de comandos que alteram arquivos, usei `pwd` para ter certeza de que estava na pasta certa. Um comando executado no lugar errado pode mover ou apagar o que não devia.

---

## 🛠️ Comandos praticados

| Comando | Função |
|---|---|
| `pwd` | Mostra o diretório atual |
| `cd <pasta>` / `cd ..` | Entra em uma pasta / sobe um nível |
| `ls` | Lista o conteúdo do diretório |
| `ls -laR` | Lista tudo, em detalhes e recursivamente |
| `mkdir <pastas>` | Cria diretórios |
| `touch <arquivos>` | Cria arquivos vazios |
| `cp -r <origem> <destino>` | Copia diretórios recursivamente |
| `mv <origem> <destino>` | Move ou renomeia |
| `rm <arquivos>` | Remove arquivos |
| `rm -r <pasta>` | Remove pasta e conteúdo recursivamente |
| `rmdir <pasta>` | Remove pasta **vazia** |

## ✅ Boas práticas que levo daqui

- **Validar cada etapa** com `ls` e `pwd` antes de seguir para a próxima
- **Conferir a ortografia** de nomes de arquivos e pastas (um erro de digitação cria um arquivo com nome errado)
- **Copiar, conferir e só depois apagar** a versão antiga
- Usar `rm -r` com **muito cuidado**
- Planejar a **estrutura de pastas** antes de criar, para evitar reorganizações grandes depois
- **Encerrar o laboratório** ao final para liberar os recursos

---

## 📚 Próximos passos

- Estudar **permissões de arquivos** (`chmod`, `chown`) e ler com calma a saída do `ls -l`
- Aprender `find` e `grep` para localizar arquivos e conteúdo
- Praticar **curingas** (`*`, `?`) para operar em vários arquivos de uma vez
- Documentar os próximos labs do re/Start neste repositório

---

## 👤 Autor

**Werther Crisley**
🔗 [GitHub](https://github.com/WertherCrisley)