# Análise de Tráfego de Rede com Wireshark

## Objetivo
Esta captura teve como objetivo analisar o handshake TCP de três vias (SYN, SYN-ACK, ACK) estabelecido ao acessar um site via navegador, observando na prática como o protocolo TCP inicia uma conexão confiável antes da troca de dados HTTP.

## Ferramentas utilizadas
- Wireshark 4.2.5
- Sistema operacional: Linux Mint 21 

## Metodologia
- Interface: `enp0s3`
- Filtro de exibição aplicado: `tcp.port == 80`
- Duração da captura: aproximadamente 30 segundos
- Ação realizada: acesso ao site `http://google.com` pelo navegador Firefox, do carregamento até o fim da página

## Pacotes analisados

- **SYN** — o cliente (192.168.1.10) envia um pacote com a flag `SYN` ativada, propondo o número de sequência inicial (ISN) e solicitando abertura da conexão com o servidor (93.184.216.34) na porta 80.

- **SYN-ACK** — o servidor responde confirmando o recebimento (`ACK`) e enviando seu próprio `SYN` com seu ISN, indicando que aceita a conexão.

- **ACK** — o cliente confirma o recebimento do SYN-ACK do servidor. A partir deste pacote a conexão TCP está estabelecida e os pacotes seguintes já trazem os dados da requisição HTTP (`GET / HTTP/1.1`).

*(Insira aqui os prints reais do Wireshark para cada um dos três pacotes: `![SYN](imagens/syn.png)`, etc.)*

## Observações e conclusões
- O handshake ocorreu exatamente como previsto pela teoria (RFC 793), com os três pacotes trocados em menos de 50ms, sem retransmissões.
- Os números de sequência e confirmação (ACK) evoluíram de forma consistente a cada pacote, confirmando que cada lado reconhece corretamente os bytes recebidos.
- Ficou evidente que a conexão TCP só é considerada "estabelecida" após o terceiro pacote (ACK) — só depois disso o navegador começa de fato a enviar a requisição HTTP.

## Filtros úteis
| Filtro | Uso |
|---|---|
| `tcp.port == 80` | Tráfego HTTP |
| `dns` | Consultas/respostas DNS |
| `icmp` | Pacotes de ping |
| `tcp.flags.syn == 1` | Pedidos de conexão (SYN) |
| `tcp.analysis.retransmission` | Retransmissões |

## Comandos Git para publicar
```bash
git add .
git commit -m "Adiciona análise de tráfego capturada com Wireshark"
git push origin main
```
> Não suba capturas `.pcap`/`.pcapng` com dados sensíveis sem anonimizar.

## Como reproduzir
1. Instale o Wireshark e abra na interface de rede ativa.
2. Aplique o filtro `tcp.port == 80` (ou o filtro desejado) e inicie a captura.
3. Acesse um site qualquer via HTTP no navegador.
4. Pare a captura e salve como `.pcapng`.

---
*Relatório desenvolvido em conjunto com as alunas do programa Mulher Digital 2026.*
