# Análise de Tráfego de Rede com Wireshark

## Objetivo

Esta captura foi feita para observar, na prática, o handshake TCP de três vias — o processo pelo qual duas máquinas "combinam" o início de uma conexão antes de qualquer dado ser trocado de fato. Para isso, acessamos um site comum (`http://example.com`) pelo navegador enquanto o Wireshark rodava em segundo plano, capturando tudo o que passava pela interface de rede.

A ideia não era só ver os pacotes passando, mas entender o que cada campo da tela do Wireshark realmente representa.

## Ferramentas utilizadas

- Wireshark 4.2.5
- Linux Mint 21

## Metodologia

A captura foi feita na interface `enp0s3`, que era a interface ativa. Antes de começar, não aplicamos filtro de **captura** (para não arriscar perder nada relevante) — o filtro veio depois, na tela de exibição, para focar só no que interessava.

A sequência foi:
1. Abrir o Wireshark e selecionar `enp0s3`.
2. Iniciar a captura.
3. Acessar `http://example.com` no navegador.
4. Parar a captura assim que a página carregou por completo (durou cerca de 30 segundos ao todo).
5. Aplicar o filtro de exibição `tcp.port == 80` para isolar o tráfego HTTP do resto do ruído da rede (ARP, DNS de outros serviços rodando na VM, etc.).

## Lendo a tela do Wireshark

Antes de entrar nos pacotes em si, vale registrar como a tela se organiza, porque é isso que permite ler qualquer captura depois:

- **Lista de pacotes** (painel de cima): cada linha é um pacote, na ordem em que foi capturado. A coluna `Info` é a mais importante pra uma leitura rápida — é ali que aparecem as flags do TCP entre colchetes, como `[SYN]`, `[SYN, ACK]`, `[ACK]`, `[FIN]` e `[RST]`.
- **Detalhes do pacote** (painel do meio): ao clicar num pacote, esse painel abre o pacote "em camadas" — Ethernet, IP, TCP/UDP e, por cima de tudo, o protocolo de aplicação (HTTP, DNS etc.). É onde dá pra conferir as flags TCP bit a bit, dentro do campo *Flags*.
- **Bytes do pacote** (painel de baixo): os mesmos dados em hexadecimal (esquerda) e em ASCII (direita). Clicar em um campo do painel do meio destaca automaticamente os bytes correspondentes aqui.

## Pacotes analisados

**SYN** — o cliente (`IPCliente`) inicia a conexão enviando um pacote com a flag `SYN` ativada, propondo seu número de sequência inicial (ISN) para a porta 80 do servidor (`IPSERVIDOR`).

**SYN-ACK** — o servidor responde num único pacote com duas flags ativas: `ACK`, confirmando o SYN recebido, e `SYN`, propondo o próprio número de sequência. É a resposta que diz "recebi seu pedido e também quero abrir a conexão".

**ACK** — o cliente confirma o SYN-ACK do servidor. A partir deste pacote a conexão está oficialmente estabelecida, e os pacotes seguintes já carregam a requisição HTTP (`GET / HTTP/1.1`).

**PSH, ACK** — depois do handshake, os pacotes que carregam dados de verdade (a requisição HTTP e a resposta do servidor) aparecem com as flags `PSH` e `ACK` juntas. O `PSH` diz para a camada de aplicação "não segure isso no buffer, entregue os dados imediatamente" — é o que garante que a página comece a ser processada assim que os bytes chegam, em vez de esperar o buffer TCP encher.


## Filtros usados e por quê

Além do `tcp.port == 80` usado para isolar o tráfego HTTP, vale documentar outros filtros que ajudam a "limpar" uma captura:

- `arp` isola só os pacotes ARP — o protocolo que resolve IP em endereço MAC dentro da rede local. Costuma aparecer em volume no início de qualquer captura, mesmo sem relação com o que se está analisando.
- `dns` isola as consultas e respostas DNS — geralmente o primeiro passo antes do handshake TCP, já que o navegador precisa resolver o nome do site para um IP.
- `!(arp or dns)` faz o oposto: esconde ARP e DNS e deixa só o restante do tráfego. Foi o filtro mais útil no dia a dia da aula, porque ARP e DNS tendem a "poluir" a lista de pacotes quando o interesse está em outro protocolo (no nosso caso, TCP/HTTP).
- `tcp.flags.reset == 1` mostra os pacotes com a flag `RST`, que indicam que uma conexão foi encerrada de forma abrupta (porta fechada, conexão recusada) — diferente do `FIN`, que é um encerramento normal, negociado entre as duas pontas.

Vale separar isso de dois outros "resets" que aparecem na aula e que não têm relação com a flag `RST`:
- O botão de recarregar (vermelho, na barra de ferramentas) limpa os pacotes já capturados na tela e permite começar uma nova captura do zero, sem fechar o Wireshark.
- O "X" ao lado da barra de filtro de exibição limpa o filtro aplicado, voltando a mostrar todos os pacotes da captura atual.

## Observações e conclusões

O handshake ocorreu como esperado pela teoria (RFC 793): três pacotes trocados em menos de 50ms, sem retransmissões. Os números de sequência e confirmação evoluíram de forma consistente pacote a pacote, o que confirma que cada lado estava reconhecendo corretamente o que já tinha sido recebido do outro.

O ponto que mais chamou atenção foi perceber que o navegador só começa a enviar a requisição HTTP depois do terceiro pacote (`ACK`) — ou seja, a "conversa" só é considerada aberta depois que as duas pontas confirmam que estão prontas, mesmo que isso pareça instantâneo do ponto de vista do usuário.

## Publicando no GitHub

```bash
git add .
git commit -m "Adiciona análise de tráfego capturada com Wireshark"
git push origin main
```

- `git add .` prepara todos os arquivos alterados (README, imagens) para entrar no próximo commit.
- `git commit -m "..."` salva essa alteração localmente, com uma mensagem explicando o que foi feito — funciona como um "ponto de restauração" do projeto.
- `git push origin main` envia os commits salvos localmente para o repositório remoto no GitHub, tornando o material público.

Não subimos o arquivo `.pcapng` original da captura, por conter dados reais de tráfego — só os prints relevantes.

## Como reproduzir

1. Instale o Wireshark e selecione a interface de rede ativa.
2. Inicie a captura sem filtro.
3. Acesse um site qualquer via HTTP no navegador.
4. Pare a captura e aplique `tcp.port == 80` (ou o filtro equivalente ao protocolo de interesse).
5. Salve a captura como `.pcapng` para consulta futura.

---
*Análise desenvolvida em conjunto com as alunas do programa Mulher Digital 2026.*
