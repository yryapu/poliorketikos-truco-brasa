# R02 — Eu li a mão de onze como evento único. É estado permanente.

**Quando:** 2026-10-01, construindo. **Como achei:** um teste meu falhou — o
`r25_na_mao_de_onze_a_dupla_ve_as_cartas_entre_si` assumia que, depois de recusar a mão de
onze, a mão seguinte era normal. Não é.

## O que eu tinha entendido

"Quando uma dupla chega a onze pontos, ela está com a mão de onze" — li como: *a mão em que a
dupla chega aos 11 é especial, e depois o jogo volta ao normal*.

## O que a fonte diz

> F2: "Quando uma dupla chega a onze pontos, ela está com a mão de onze. **A partir deste
> momento, no começo de cada mão** esta dupla pode escolher se continua com a mão ou não."

É **estado**, não evento. Uma dupla em 11 pontos tem mão de onze em **toda** mão seguinte, até
a partida acabar. E acaba rápido: aceitar e ganhar dá 11+3 = 14; aceitar e perder dá 3 ao
adversário; recusar dá 1. Mas o adversário pode ficar em 11 recusas sucessivas enquanto só
sobe 1 por vez, e cada uma dessas mãos é de onze.

## Por que isso importa no código

O tipo da mão tem de ser **derivado do placar a cada distribuição**, não um sinalizador que se
liga uma vez e se apaga. É o que `Mao::do_baralho` faz: recalcula `TipoMao` a partir de
`placar` toda vez. Se eu tivesse implementado como evento, a segunda mão depois de uma recusa
viria normal — e o truco voltaria a ser permitido numa mão em que a dupla está a um ponto de
ganhar. Seria um bug de regra invisível, porque só aparece em placar alto.

## A lição de método

O teste que eu escrevi para provar uma regra (R-25) refutou **a minha leitura de outra** (R-22).
Isso só aconteceu porque o teste afirmava o estado da mão *seguinte* em vez de só a visão. Vale
como argumento a favor de testes que verificam uma transição a mais do que o estritamente
necessário.

## Cai se

Alguma casa jogar a mão de onze como evento único. Nenhuma das quatro fontes sugere isso.
