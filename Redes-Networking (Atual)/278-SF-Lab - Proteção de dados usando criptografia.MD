# 🔐 AWS re/Start — Proteção de Dados com Criptografia (AWS KMS e Encryption CLI)

Laboratório prático do programa **AWS re/Start** em que criei uma **chave de criptografia no AWS KMS**, instalei a **AWS Encryption CLI** em uma instância EC2 e usei a chave para **criptografar** um arquivo com dados sigilosos e depois **descriptografá-lo**, vendo na prática a diferença entre **texto simples** e **texto cifrado**.

![AWS](https://img.shields.io/badge/AWS-KMS-FF9900?logo=amazonaws&logoColor=white)
![EC2](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white)
![Segurança](https://img.shields.io/badge/Seguran%C3%A7a-criptografia-D22128)
![Status](https://img.shields.io/badge/status-conclu%C3%ADdo-success)

---

## 🎯 Objetivos do laboratório

- Criar uma **chave de criptografia do AWS KMS**
- Instalar a **AWS Encryption CLI**
- **Criptografar** texto simples
- **Descriptografar** texto cifrado

## 🧠 Conceitos principais

| Conceito | O que é |
|---|---|
| **Criptografia** | Transformar dados legíveis em um formato ilegível, para que só quem tem autorização consiga ler |
| **Descriptografia** | O processo inverso: devolver os dados ao formato legível |
| **Texto simples (*plaintext*)** | O dado original, legível |
| **Texto cifrado (*ciphertext*)** | O dado depois de criptografado, ilegível sem a chave |
| **Chave simétrica** | A **mesma chave** criptografa e descriptografa (rápida e eficiente) |
| **Chave assimétrica** | **Chave pública** criptografa, **chave privada** descriptografa |
| **AWS KMS** | Serviço para criar e gerenciar chaves criptográficas e controlar o uso delas |

> A criptografia garante **privacidade** (só o destinatário lê), e também apoia **autenticação**, **integridade dos dados** e **não repúdio**.

---

## 🧩 O que eu fiz

### Tarefa 1: Criar a chave no AWS KMS
No console, em **Key Management Service → Create a key**:

| Opção | Valor |
|---|---|
| Tipo de chave | **Symmetric** (simétrica) |
| Alias | `MyKMSKey` |
| Descrição | `Key used to encrypt and decrypt data files` |
| Administradores da chave | perfil `voclabs` |
| Uso da chave (esta conta) | perfil `voclabs` |

Depois de criada, abri a chave e **copiei o ARN** (identificador único) para usar mais adiante.

> As chaves do KMS são protegidas por **módulos de segurança de hardware (HSMs)**.

### Tarefa 2: Preparar o servidor de arquivos
1. Conectei na instância **File Server** pelo **Session Manager** (do Systems Manager), **sem precisar de SSH nem de chave `.pem`**
2. Configurei as credenciais da AWS:

```bash
cd ~
aws configure
```

   Primeiro com valores temporários (`1`, `1`), a região do lab e o formato de saída padrão
3. Substituí o conteúdo do arquivo `~/.aws/credentials` pelas **credenciais temporárias** fornecidas pelo lab
4. Instalei a **AWS Encryption CLI** e ajustei o `PATH`:

```bash
pip3 install aws-encryption-sdk-cli
export PATH=$PATH:/home/ssm-user/.local/bin
```

> 🔐 **Segurança:** as credenciais do lab são **temporárias** e incluem um *session token*. **Nunca** publique chaves, tokens ou o arquivo `~/.aws/credentials` em repositórios ou prints.

### Tarefa 3: Criptografar e descriptografar

**1. Criei os arquivos e um diretório de saída:**
```bash
touch secret1.txt secret2.txt secret3.txt
echo 'TOP SECRET 1!!!' > secret1.txt
cat secret1.txt
mkdir output
```

**2. Guardei o ARN da chave em uma variável:**
```bash
keyArn=<ARN-da-minha-chave-KMS>
```

**3. Criptografei o `secret1.txt`:**
```bash
aws-encryption-cli --encrypt \
  --input secret1.txt \
  --wrapping-keys key=$keyArn \
  --metadata-output ~/metadata \
  --encryption-context purpose=test \
  --commitment-policy require-encrypt-require-decrypt \
  --output ~/output/.
```

| Parâmetro | Função |
|---|---|
| `--encrypt` | Define a operação |
| `--input` | Arquivo a ser criptografado |
| `--wrapping-keys key=$keyArn` | Chave do KMS que protege os dados |
| `--metadata-output` | Onde gravar os metadados da operação |
| `--encryption-context purpose=test` | Contexto de criptografia (boa prática) |
| `--commitment-policy ...` | Exige o recurso de **segurança com confirmação de chave** na criptografia e na descriptografia |
| `--output` | Pasta de saída |

**4. Verifiquei o resultado:**
```bash
echo $?        # 0 = sucesso; outro valor = falha
ls output      # secret1.txt.encrypted
cd output
cat secret1.txt.encrypted    # conteúdo ilegível (texto cifrado)
```

> Quando o comando de criptografia funciona, ele **não imprime nada**. O `echo $?` é o jeito de confirmar o sucesso.

**5. Descriptografei o arquivo:**
```bash
aws-encryption-cli --decrypt \
  --input secret1.txt.encrypted \
  --wrapping-keys key=$keyArn \
  --commitment-policy require-encrypt-require-decrypt \
  --encryption-context purpose=test \
  --metadata-output ~/metadata \
  --max-encrypted-data-keys 1 \
  --buffer \
  --output .
```

```bash
ls
cat secret1.txt.encrypted.decrypted    # TOP SECRET 1!!!
```

O arquivo `secret1.txt.encrypted.decrypted` trouxe de volta o **texto original**. ✅

---

## 💡 O que eu aprendi

- O **texto cifrado é ilegível**, e só volta a fazer sentido com a **chave certa**
- Em **criptografia simétrica**, a **mesma chave** faz os dois caminhos
- O **KMS guarda e controla a chave**, e eu **controlo quem pode usá-la** (administradores e usuários da chave)
- O **contexto de criptografia** (`purpose=test`) é um dado adicional **associado** à operação, e precisa ser o **mesmo na descriptografia**, o que dá uma camada extra de verificação
- O código de saída **`$?`** (0 = sucesso) é uma forma simples de checar comandos que não imprimem nada
- O **Session Manager** permite acessar a instância **sem abrir a porta SSH**, o que é mais seguro

> 💡 **Para ir além:** por trás, a Encryption CLI usa **criptografia de envelope**: o KMS gera uma **chave de dados** que criptografa o arquivo, e a **chave do KMS protege essa chave de dados**. Assim, os dados em si não precisam trafegar até o KMS.

> 💰 **Custos:** em uma conta própria, chaves do KMS geram cobrança mensal. Quando não precisar mais, é bom **agendar a exclusão da chave**.

> ⚠️ Os passos descritos no lab criptografam só o `secret1.txt`. Os outros dois arquivos (`secret2.txt` e `secret3.txt`) são criados, mas ficam sem uso.

## ✅ Boas práticas que levo daqui

- **Criptografar dados sensíveis**, tanto em repouso quanto em trânsito
- **Gerenciar chaves em um serviço próprio** (KMS), em vez de guardá-las junto dos dados
- **Controlar quem administra e quem usa** cada chave (menor privilégio)
- **Usar contexto de criptografia** nas operações
- **Nunca expor credenciais** em repositórios ou prints
- **Preferir o Session Manager** ao SSH quando possível
- **Encerrar o laboratório** ao final para liberar os recursos

---


## 👤 Autor

**Werther Crisley**
🔗 [GitHub](https://github.com/WertherCrisley)