# ADR-005 — Quem responde o truco: só quem está na vez

**Data:** 2026-10-01 · **Estado:** VIVO

A fonte F2 diz: *"Qualquer um dos jogadores pode responder o pedido e vale a primeira
resposta dada, mesmo que eles se contradigam entre si."* Isso é uma regra de **mesa física**,
onde "primeiro" é quem falou mais alto meio segundo antes.

## Decisão

Na v1, o pedido de truco passa a vez para **um** jogador determinado: o próximo adversário na
ordem de assento. Só ele aceita, corre ou retruca.

## Por que

"Vale a primeira resposta" num servidor é uma corrida entre dois WebSockets, resolvida pela
latência de rede de cada jogador. Implementar isso fielmente significa: aceitar as duas
respostas, ordenar por chegada, e explicar ao jogador que o parceiro correu antes de ele
aceitar — porque o parceiro tem 20 ms menos de ping. Isso não reproduz a regra da mesa, **troca
a regra da mesa por uma corrida de infraestrutura**, e o jogador perde a mão por causa do
provedor dele.

## Custo aceito

O parceiro não pode socorrer quem está na vez. Numa mesa de verdade ele poderia. É um desvio
declarado de F2 — está marcado como `[DECISÃO]` dentro de R-21 na especificação, e não
escondido como se fosse a regra pesquisada.

## Cai se

O avaliador considerar a resposta do parceiro essencial ao truco paulista. A correção então é
desenhada, não improvisada: um prazo curto de resposta (p.ex. 3 s) em que **as duas** respostas
da dupla são aceitas e a de **maior compromisso** vence (retrucar > aceitar > correr) — o que
é determinístico e não depende de ping. Não implemento isso agora porque introduz um temporizador
no laço da partida para um caso que ninguém reclamou.
