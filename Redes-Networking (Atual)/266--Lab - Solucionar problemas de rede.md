# 🔧 AWS re/Start — Solução de um Problema na Rede

Laboratório do programa **AWS re/Start** em que atuei como **engenheiro de suporte de nuvem** e investiguei, camada por camada, por que uma cliente **não conseguia fazer `ping` nem abrir a página do seu servidor Apache** em uma instância EC2.

![AWS](https://img.shields.io/badge/AWS-VPC-FF9900?logo=amazonaws&logoColor=white)
![EC2](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white)
![Apache](https://img.shields.io/badge/Apache-httpd-D22128?logo=apache&logoColor=white)
![Status](https://img.shields.io/badge/status-concluído-success)

---

## 📩 O cenário

A cliente (Ana), contratante de uma consultoria, escreveu:

> "Quando crio um servidor Apache pela linha de comando, **não consigo fazer um ping**. Também recebo **um erro quando insiro o endereço IP no navegador**. Poderia me ajudar a descobrir o que está bloqueando minha conexão?"

A arquitetura dela: **VPC + gateway de internet + sub-rede pública + instância EC2**. No lab, eu tinha uma **réplica exata** desse ambiente para investigar.

---

## 🧩 O que eu fiz

### 1. Reproduzi o problema
1. **Conectei por SSH** à instância Amazon Linux
2. **Verifiquei o serviço Apache** e o iniciei:

```bash
sudo systemctl status httpd.service   # inativo: instalado, mas ainda não iniciado
sudo systemctl start httpd.service
sudo systemctl status httpd.service   # ativo
```

3. **Tentei abrir** `http://<IP-público-da-instância>` no navegador. A página de teste do Apache **não carregou**, o que reproduziu o problema da cliente.

> **Conclusão parcial:** o **Apache está ativo**, então o problema **não é o serviço**. É algo no caminho da rede.

### 2. Investiguei a VPC, recurso por recurso
Percorri o menu da VPC conferindo cada peça:

| Recurso | O que conferir |
|---|---|
| **Sub-redes** | A tabela de rotas está associada à sub-rede correta? |
| **Tabelas de rotas** | Existe a rota `0.0.0.0/0` apontando para o gateway de internet? |
| **Gateway de internet** | Existe e está **anexado** à VPC? |
| **Grupos de segurança** | As regras de **entrada** liberam as portas necessárias? |
| **NACLs** | As regras de entrada **e de saída** permitem o tráfego? |

**Dicas do lab que usei:**
- Se a instância consegue fazer `ping` em `www.amazon.com`, **a saída para a internet funciona** (gateway de internet e rota estão certos)
- O Apache normalmente usa as portas **80 (HTTP)** e **443 (HTTPS)**

---

## 🧠 O raciocínio do diagnóstico

Em vez de mudar coisas aleatoriamente, fui **eliminando causas**:

| Pista | O que ela me diz |
|---|---|
| Consegui conectar por **SSH** (porta 22) no IP público | A instância tem IP público, o gateway de internet, a rota e a sub-rede **funcionam** |
| O **Apache está ativo** na instância | O serviço não é o problema |
| A página **não abre** (porta 80) e o **ping** não responde | Falta liberar **essas portas/protocolos** em um firewall |

➡️ **Suspeitos principais:** as regras de **entrada** do **grupo de segurança** (e, se estiver associada, da **NACL**) não liberam o **HTTP (80)**, o **HTTPS (443)** e o **ICMP** (usado pelo `ping`).

### Como corrigir
No **grupo de segurança** da instância, adicionar regras de **entrada**:

| Tipo | Porta | Origem |
|---|---|---|
| HTTP | 80 | Onde o acesso for necessário (no lab, qualquer lugar) |
| HTTPS | 443 | Idem |
| ICMP (ping) | — | Idem |

E, se a NACL da sub-rede estiver restringindo, conferir que ela **permite o tráfego nas duas direções** (a NACL é *stateless*).

### Teste final
Voltar ao navegador em `http://<IP-público>` e confirmar que aparece a **página de teste do Apache**. ✅

<!--
✏️ COMPLETE COM O QUE VOCÊ ENCONTROU NO SEU LAB:
- Qual recurso estava errado? (security group, NACL, rota, outro)
- O que você alterou? (qual regra, qual porta)
- O resultado depois da correção
Ajuste a seção "O raciocínio do diagnóstico" se a causa for diferente da hipótese acima.
-->

---

## 🧠 O que eu aprendi

- **Solucionar problemas por camadas**, validando cada recurso, em vez de testar ao acaso
- **Usar pistas do que já funciona** (como o SSH) para descartar causas
- O **security group** é o primeiro lugar para olhar quando uma **porta específica** não responde
- **Serviço ativo não significa serviço acessível**: a rede precisa liberar o tráfego
- Uma **NACL** é *stateless* e precisa de regras de ida e de volta, ao contrário do security group (*stateful*)
- Diferenciar **problema de aplicação** (Apache parado) de **problema de rede** (tráfego bloqueado)

## ✅ Boas práticas que levo daqui

- **Reproduzir o problema** antes de mexer em qualquer coisa
- Percorrer uma **checklist** fixa: sub-rede → rotas → gateway → security group → NACL
- **Liberar só o necessário** nos firewalls (evitar "qualquer lugar" em produção, sobretudo em portas administrativas)
- **Documentar** o que foi encontrado e alterado
- **Encerrar o laboratório** ao final para liberar os recursos

---

## 👤 Autor

**Werther Crisley**
🔗 [GitHub](https://github.com/WertherCrisley)