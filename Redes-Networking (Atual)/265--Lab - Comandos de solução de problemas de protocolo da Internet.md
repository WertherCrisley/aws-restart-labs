# 🛠️ AWS re/Start — Comandos de Solução de Problemas de Rede (`ping`, `traceroute`, `netstat`, `telnet` e `curl`)

Laboratório prático do programa **AWS re/Start** em que, conectado por **SSH** a uma instância **Amazon Linux (EC2)**, atuei como **administrador de rede** e pratiquei os comandos mais usados para **diagnosticar problemas de conectividade**, relacionando cada um à sua **camada do modelo OSI**.

![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white)
![Linux](https://img.shields.io/badge/Amazon_Linux-CLI-232F3E?logo=linux&logoColor=white)
![Redes](https://img.shields.io/badge/Redes-troubleshooting-0A66C2)
![Status](https://img.shields.io/badge/status-concluído-success)

---

## 🎯 Objetivos do laboratório

- Praticar comandos de solução de problemas de rede
- Identificar como usá-los em **cenários reais de clientes**

## 🧩 O que eu fiz

1. **Conectei por SSH** à instância Amazon Linux (`ec2-user`)
2. **Testei a conectividade** com `ping` e tracei o caminho dos pacotes com `traceroute` (camada 3)
3. **Inspecionei conexões e portas** com `netstat` e testei portas com `telnet` (camada 4)
4. **Testei uma requisição web** completa com `curl` (camada 7)

---

## 🧠 A ideia central: solucionar problemas por camadas

Em vez de tentar adivinhar, uso o **modelo OSI** para investigar **de baixo para cima**. Cada camada tem comandos próprios:

| Camada OSI | O que verifica | Comandos |
|---|---|---|
| **3 – Rede** | Endereços IP e rotas | `ping`, `traceroute` |
| **4 – Transporte** | Portas e conexões TCP | `netstat`, `telnet` |
| **7 – Aplicação** | A aplicação (HTTP/HTTPS) | `curl` |

> **Lição:** começar pela camada de baixo ajuda a **eliminar causas** uma de cada vez e economiza tempo.

---

## 🔎 Camada 3 (Rede): `ping` e `traceroute`

### `ping`: "o destino responde?"
```bash
ping 8.8.8.8 -c 5
```

- Envia pedidos **ICMP echo** ao destino e mede o **tempo de ida e volta**
- **`-c 5`**: envia 5 pedidos e para (**c**ount)
- Aceita **IP ou nome** (por exemplo, `amazon.com`)

**Quando usar:** para testar **conectividade e alcançabilidade**, inclusive para verificar se grupos de segurança e NACLs permitem **ICMP**.

### `traceroute`: "por onde o pacote passa?"
```bash
traceroute 8.8.8.8
```

Mostra o **caminho** (cada servidor intermediário é um **hop**) e a **latência** em cada etapa.

**Cenário do cliente:** a conexão está lenta e há perda de pacotes. É problema da AWS ou do provedor (ISP)?
- Perda que aparece **cedo** na rota costuma indicar a **rede local/ISP**
- Perda que aparece **no final** da rota indica problema **no servidor de destino**
- Três asteriscos (`* * *`) significam que **aquele hop não respondeu**

> 💡 **Nuance:** `* * *` pode ser uma falha real, mas também um roteador configurado para **não responder** a esse tipo de teste. Vale olhar o conjunto da rota, e não só um hop.

---

## 🔎 Camada 4 (Transporte): `netstat` e `telnet`

### `netstat`: "o que está escutando e conectado na minha máquina?"
```bash
netstat -tp      # conexões TCP estabelecidas
netstat -tlp     # serviços escutando (listening)
netstat -ntlp    # idem, mostrando portas em número (sem resolver nomes)
```

| Opção | Significado |
|---|---|
| `-t` | Apenas **TCP** |
| `-l` | Apenas portas **escutando** (*listening*) |
| `-p` | Mostra o **programa** dono da conexão |
| `-n` | Mostra **números** de portas/IPs, sem resolver nomes |

**Cenário do cliente:** numa verificação de segurança, descobriram que uma porta de uma sub-rede pode estar comprometida. Com o `netstat`, vejo no próprio host **se a porta está escutando quando não deveria**.

> **Lição:** como o `netstat` dá um "retrato" da camada 4, ele reduz rápido um problema grande de rede.

### `telnet`: "consigo abrir conexão nessa porta?"
```bash
sudo yum install telnet -y     # instala o telnet
telnet www.google.com 80       # testa a porta 80 do destino
```

**Cenário do cliente:** um servidor web seguro com regras de **security group e NACL** personalizadas. O cliente quer garantir que a **porta 80 esteja realmente bloqueada** (ou aberta). O `telnet <ip> 80` mostra o que acontece de verdade.

**Como interpretar o resultado:**

| Resultado | O que costuma indicar |
|---|---|
| **Conectou** | Nada está bloqueando a porta, e há um serviço respondendo |
| **Connection refused** (recusada) | O destino foi alcançado, mas **recusou** a conexão (porta fechada ou sem serviço) |
| **Timeout / sem resposta** | Pacotes **descartados no caminho**, comum com firewall, security group, NACL ou problema de rota |

> 💡 O lab associa "recusada" a firewall/security group e "interrompida" a problema de rota. Na prática, firewalls costumam **descartar** pacotes (resultando em *timeout*), enquanto "recusada" costuma vir do próprio destino. Em todo caso, o `telnet` ajuda a **separar** problema de rede de problema de aplicação.

---

## 🔎 Camada 7 (Aplicação): `curl`

```bash
curl -vLo /dev/null https://aws.com
```

| Opção | Significado |
|---|---|
| `-v` | **Verbose**: mostra os detalhes da conexão (DNS, TLS, cabeçalhos) |
| `-L` | **Segue redirecionamentos** |
| `-o /dev/null` | Descarta o conteúdo da página (só importa o diálogo da conexão) |
| `-I` | Faz uma requisição **HEAD** e mostra só os cabeçalhos |
| `-i` | Inclui os **cabeçalhos** da resposta na saída |
| `-k` | **Ignora erros de certificado** SSL (só para testes!) |

**Cenário do cliente:** um servidor Apache em execução e a dúvida: ele responde **`200 OK`**? Esse código indica que o site está funcionando corretamente.

> **Lição:** o `curl` testa a **comunicação completa até a aplicação**. Se o `ping` e o `telnet` funcionam mas o `curl` falha, o problema provavelmente está **na aplicação**, e não na rede.

> ⚠️ O `-k` desativa a verificação do certificado e deve ser usado **apenas em testes**, nunca em produção.

---

## 🧭 Qual comando usar quando?

| Sintoma | Primeiro comando |
|---|---|
| "Não consigo alcançar o servidor" | `ping` |
| "Está lento / perdendo pacotes" | `traceroute` |
| "Quais portas estão abertas no meu servidor?" | `netstat -tlp` |
| "A porta X está acessível?" | `telnet <host> <porta>` |
| "O site responde direito (200 OK)?" | `curl -v` |

---

## ✅ Boas práticas que levo daqui

- **Solucionar problemas por camadas**, de baixo para cima
- **Começar pelo host local** (`netstat`) e ir ampliando para fora
- **Usar `traceroute`** para saber se o problema está na AWS, no ISP ou no destino
- **Testar portas com `telnet`** em vez de confiar apenas nas regras de firewall
- **Não usar `curl -k`** fora de testes
- **Borrar IPs** ao publicar prints de comandos de rede
- **Encerrar o laboratório** ao final para liberar os recursos

---


## 📚 Próximos passos

-Explorar `ss` (substituto moderno do `netstat`), `dig`/`nslookup` (DNS) e `tcpdump`
-Praticar o diagnóstico **de ponta a ponta** em uma VPC com security group e NACL
-Usar o **VPC Reachability Analyzer** da AWS para analisar caminhos de rede
-Documentar os próximos labs do re/Start neste repositório

---

## 👤 Autor

**Werther Crisley**
🔗 [GitHub](https://github.com/WertherCrisley)