# ADR-010 — O histórico de mãos é montado no cliente, e a tela diz isso

**Data:** 2026-10-01 · **Estado:** VIVO

Depois de o operador jogar: "deve ter o historico pra acompanhar/filtrar".

## O que foi para o servidor, e o que não foi

**Foi para o servidor** o detalhe da mão corrente e o da mão que acabou:

- `estado.rodadas` passou a trazer, para cada rodada resolvida desta mão, as cartas na ordem
  jogada, a dupla que levou e o assento que pôs a carta vencedora;
- um aviso `mao_resolvida` carrega a mão inteira ao terminar — porque o `estado` seguinte já é
  da mão nova, e sem isso a última rodada de cada mão se perdia.

**Não foi para o servidor** o histórico acumulado da partida. O cliente o monta a partir dos
avisos que já recebe.

## Por que essa fronteira

O princípio do protocolo é que o `estado` é suficiente: o cliente redesenha a mesa inteira a
cada estado, sem memória. A rodada resolvida **tinha** de entrar nele, porque faz parte da
mesa que está em tela — era exatamente o defeito de R05.

O histórico de mãos passadas é outra coisa: não é a mesa, é um registro. Mandá-lo em toda
mensagem de estado custaria alguns KB por ação, numa partida de poucos minutos, para um painel
que fica fechado por padrão.

**Alternativa descartada:** guardar o histórico no servidor e enviá-lo no estado. Compra uma
coisa real — o histórico sobrevive a recarregar a página e a uma reconexão. Descartada nesta
versão porque **não existe reconexão** (está nos riscos conhecidos): quem cai perde a partida,
então o histórico persistente não teria a quem servir.

**Alternativa descartada:** uma rota `GET /api/partidas/{id}/maos`. É a resposta certa quando
houver reconexão ou quando alguém quiser rever uma partida antiga. Não agora, porque hoje
nenhuma dessas duas coisas existe.

## O custo central que esta decisão aceita

**Recarregar a página apaga o histórico.** Aceito — e a consequência é que a tela **diz isso**,
numa linha discreta no painel, em vez de deixar o jogador descobrir ao recarregar.

Isto é a regra geral que eu quero registrada: *quando uma decisão de desenho tem um limite
que o usuário vai encontrar, o limite vai para a interface.* Não para o README, não para um
comentário no código — para o lugar onde a pessoa está quando o limite a atinge.

**Gate de reversão:** no dia em que houver reconexão, o histórico sobe para o servidor junto,
porque aí ele passa a ter uma função que o cliente não consegue cumprir.

## Uma coisa que o construtor da tela me ensinou

Ele estava reconstruindo "esta mão teve truco?" a partir do aviso `pediu`, e me disse que
aquilo falharia se o aviso se perdesse ou chegasse fora de ordem. Estava certo: **dado
derivado de narração mente em silêncio.** Por isso `mao_resolvida` leva `valor` explícito, e o
filtro usa `valor > 1`.

A mesma objeção vale para `assento_vencedor`, que ele também pediu. Num protocolo, o que a
interface precisa afirmar tem de vir dito, não inferido.
