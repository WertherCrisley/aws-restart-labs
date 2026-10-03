# 📍 AWS re/Start — Endereços IP Dinâmicos e Estáticos (Elastic IP)

Laboratório do programa **AWS re/Start** em que atuei como **engenheiro de suporte de nuvem** para resolver o problema de um cliente: o **IP público** da instância EC2 mudava toda vez que ela era parada e iniciada, quebrando os recursos que dependiam desse endereço.

![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white)
![Elastic IP](https://img.shields.io/badge/AWS-Elastic_IP-FF9900?logo=amazonaws&logoColor=white)
![Status](https://img.shields.io/badge/status-concluído-success)

---

## 📩 O cenário

O cliente (Bob) tem uma **VPC** com uma sub-rede pública e uma instância EC2:

- O **IP muda** toda vez que ele **para e inicia** a instância
- Ele **não pode deixá-la ligada o tempo todo** porque seria caro
- Ele **precisa de um IP fixo**; caso contrário, os outros recursos ligados a esse endereço deixam de funcionar

---

## 🔍 Investigação: reproduzindo o problema

Para confirmar a teoria, criei uma instância de teste com a mesma configuração do cliente:

| Configuração | Valor |
|---|---|
| Nome | `test instance` |
| AMI | Amazon Linux 2023 |
| Tipo | `t2.micro` (ou `t3.micro`) |
| Par de chaves | `vockey` |
| VPC / Sub-rede | VPC do laboratório / Sub-rede pública 1 |
| IP público automático | **Ativado** |
| Grupo de segurança | `Linux Instance SG` |

Depois:

1. Anotei os IPs **público e privado** na guia **Redes**
2. **Parei** a instância e olhei os IPs de novo
3. **Iniciei** a instância e comparei os endereços

### O que observei

| | Antes de parar | Depois de parar e iniciar |
|---|---|---|
| **IP público** | um endereço | **mudou** 🔄 |
| **IP privado** | um endereço | **permaneceu o mesmo** ✅ |

**Conclusão da investigação:** o IP público é **dinâmico**. Ele é emprestado de um conjunto da AWS e **devolvido** quando a instância é parada. O IP privado dentro da VPC fica **estável**. Assim, o problema do cliente foi **reproduzido**.

---

## ✅ A solução: Elastic IP (EIP)

Um **Elastic IP** é um endereço IPv4 público **estático**, que fica reservado na minha conta e **não muda** quando a instância é parada e iniciada.

### Passo a passo

1. No EC2, fui em **Rede e segurança → IPs elásticos**
2. Cliquei em **Alocar endereço IP elástico** (configurações padrão) e anotei o endereço
3. Selecionei o EIP e fui em **Ações → Associar endereço IP elástico**
4. Escolhi o tipo de recurso **Instância**, selecionei a `test instance`, o IP privado dela e cliquei em **Associar**
5. Voltei em **Instâncias → Redes** e confirmei que o **IP público agora era o EIP**
6. **Parei e iniciei** a instância novamente

### Resultado

| | Sem EIP | Com EIP |
|---|---|---|
| IP público após parar/iniciar | Muda 🔄 | **Permanece o mesmo** ✅ |
| Tipo | Dinâmico | **Estático** |

**Problema do cliente resolvido:** o IP público passou a ser fixo, mesmo com a instância sendo parada e iniciada.

---

## 🧠 O que eu aprendi

| | IP privado | IP público (padrão) | Elastic IP |
|---|---|---|---|
| **Tipo** | Estável na VPC | **Dinâmico** | **Estático** |
| **Muda ao parar/iniciar?** | Não | **Sim** | Não |
| **Alcançável pela internet?** | Não | Sim | Sim |

- O **IP público automático** não é permanente: ele muda quando a instância é **parada e iniciada**
- O **Elastic IP** é a forma de ter um endereço público **fixo** na AWS
- Reproduzir o problema **antes** de aplicar a correção confirma a causa e valida a solução
- Na prática, se as coisas dependem de um endereço, o ideal é um **IP estático** (ou um **nome DNS**)

> 💰 **Atenção a custos:** a AWS cobra por endereços IPv4 públicos, e um **Elastic IP que não está em uso** também pode gerar cobrança. Quando não precisar mais, é bom **desassociar e liberar** (*release*) o EIP.

---

## 📝 Resposta para o cliente (resumo)

> Olá, Bob! O IP público da sua instância muda porque, por padrão, a AWS atribui um **IP público dinâmico**, que é devolvido ao conjunto da AWS quando a instância é parada. Reproduzimos o problema em um ambiente de teste e confirmamos isso. Para resolver, **aloque um Elastic IP** e associe à instância: ele é um **IP público estático** que continua o mesmo ao parar e iniciar. Lembre-se de **liberar o EIP** quando não precisar mais dele, para evitar cobranças.

---

## ✅ Boas práticas que levo daqui

- **Reproduzir o problema do cliente** antes de propor a solução
- Usar **Elastic IP** quando um serviço depende de um **IP público fixo**
- **Liberar EIPs** que não estão em uso para evitar custos
- **Explicar a causa e a solução** de forma clara para o cliente
- **Encerrar o laboratório** ao final para liberar os recursos

---

## 👤 Autor

**Werther Crisley**
🔗 [GitHub](https://github.com/WertherCrisley)