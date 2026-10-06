# 🛡️ AWS re/Start — Amazon Inspector: Avaliação e Correção de Vulnerabilidades

Laboratório prático do programa **AWS re/Start** em que usei o **Amazon Inspector** para **encontrar vulnerabilidades** em funções **AWS Lambda**, entendi o relatório e **corrigi** o problema, confirmando depois que a descoberta foi fechada.

![AWS](https://img.shields.io/badge/AWS-Inspector-FF9900?logo=amazonaws&logoColor=white)
![Lambda](https://img.shields.io/badge/AWS-Lambda-FF9900?logo=awslambda&logoColor=white)
![Segurança](https://img.shields.io/badge/Seguran%C3%A7a-vulnerabilidades-D22128)
![Status](https://img.shields.io/badge/status-conclu%C3%ADdo-success)

---

## 🎯 Objetivos do laboratório

- Ativar o **Amazon Inspector**
- Analisar e interpretar as **descobertas de vulnerabilidades**
- **Corrigir** as vulnerabilidades encontradas

## 📩 O cenário

Os desenvolvedores de uma empresa fictícia estão criando uma aplicação baseada principalmente em **AWS Lambda** e precisam de uma ferramenta de segurança **automatizada**, que verifique **pacotes de software vulneráveis** (e também o código). A escolha foi o **Amazon Inspector**, que:

- Responde rápido a **novas implantações** de funções Lambda
- Também verifica automaticamente **instâncias EC2** e **repositórios do Amazon ECR**

---

## 🧩 O que eu fiz

### Tarefa 1: Ativar o Amazon Inspector
1. No console da AWS, pesquisei **Inspector**
2. Cliquei em **Ativar o Inspector** (necessário só na primeira vez)
3. Vi a mensagem de boas-vindas informando que a **primeira verificação estava em andamento**
4. Atualizei a página até o painel mostrar **Cobertura do ambiente → Funções do Lambda em 100%**

> Por padrão, a verificação fica ativa para **EC2, ECR e Lambda**.

### Tarefa 2: Analisar as descobertas
1. Em **Descobertas → Todas as descobertas**, vi **3 descobertas de severidade Média** na função Lambda
2. Cada linha mostra: a **severidade**, o **recurso afetado** (a função Lambda) e o **título** (o motivo)
3. Abri a descoberta **CVE-2023-32681 – requests** e analisei:
   - **Detalhes da vulnerabilidade**, com o ID e um link para a **NVD** (*National Vulnerability Database*, mantida pelo NIST)
   - A seção **Correção**, com a recomendação

**Diagnóstico:** o pacote Python **`requests`** usado pela função estava **desatualizado e vulnerável**, e a recomendação era **atualizá-lo**.

### Tarefa 3: Corrigir a vulnerabilidade
1. No **Lambda**, abri a função **`get-request`**
2. No editor de código, abri o **`requirements.txt`**
3. Troquei a linha que fixava a versão antiga:

```diff
- requests==2.20.0
+ requests
```

4. Cliquei em **Implantar** (*Deploy*). O console confirmou que a função foi atualizada
5. A nova implantação **acionou automaticamente** uma nova verificação do Inspector
6. De volta ao Inspector, em **Todas as descobertas**, mudei o filtro de **Ativo** para **Fechado** e vi a descoberta **CVE-2023-32681 – requests** ali ✅
7. Em **Cobertura de recursos → Funções do Lambda**, conferi que a **última verificação** da função tinha o carimbo de data e hora atualizado

---

## 🧠 O que eu aprendi

### Vulnerabilidades e CVE
Uma **CVE** (*Common Vulnerabilities and Exposures*) é o identificador público de uma falha de segurança conhecida. O Inspector cruza os **pacotes usados** pelos recursos com bancos como a **NVD** e informa quais estão vulneráveis.

### O fluxo de trabalho de segurança
| Etapa | O que fiz |
|---|---|
| **Detectar** | O Inspector verificou as funções Lambda automaticamente |
| **Analisar** | Li a severidade, o recurso afetado e os detalhes da CVE |
| **Corrigir** | Atualizei o pacote vulnerável e implantei a função |
| **Validar** | Confirmei que a descoberta passou para **Fechada** |

### Verificação contínua
O Inspector **reage a novas implantações**: ao atualizar a função, ele **reverificou sozinho**, sem eu precisar iniciar nada. Isso ajuda a pegar problemas **durante o desenvolvimento**.

### Sobre o `requirements.txt`
É o arquivo que lista os **pacotes Python** de que a função precisa. Fixar uma versão (`requests==2.20.0`) garante previsibilidade, mas **pode prender a aplicação em uma versão vulnerável**.

> 💡 **Ponto de atenção:** no lab, eu removi a versão para pegar a mais recente. Em projetos reais, uma prática comum é **fixar uma versão corrigida** (por exemplo, a mínima que resolve a CVE, conforme a NVD), para manter builds **reproduzíveis**, e atualizar de forma planejada.

---

## ✅ Boas práticas que levo daqui

- **Ativar verificação automática** de vulnerabilidades desde o início do desenvolvimento
- **Priorizar pela severidade** e ler os detalhes antes de corrigir
- **Manter as dependências atualizadas**
- **Confirmar a correção** (descoberta fechada), em vez de presumir que deu certo
- **Consultar fontes oficiais** (NVD/NIST) para entender o impacto da CVE
- **Encerrar o laboratório** ao final para liberar os recursos

---

## 📚 Próximos passos

- Explorar a verificação de **código** do Inspector para Lambda
- Ver as descobertas de **EC2** e **ECR**
- Integrar o Inspector com **Security Hub** e **EventBridge** para alertas
- Estudar o **CVSS** e como a severidade é calculada
-Documentar os próximos labs do re/Start neste repositório

---

## 👤 Autor

**Werther Crisley**
🔗 [GitHub](https://github.com/WertherCrisley)