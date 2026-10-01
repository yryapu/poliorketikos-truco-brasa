# ADR-006 — Mão de ferro aberta (cada um vê as próprias cartas)

**Data:** 2026-10-01 · **Estado:** VIVO

F2 oferece as duas variantes explicitamente: *"Dependendo da regra usada, a mão de ferro é
aberta, isto é, cada um pode ver suas cartas normalmente, ou é fechada."* F1 e F4 descrevem a
fechada ("no escuro").

## Decisão

**Aberta.** Em 11×11 joga-se a mão valendo 1 ponto, sem truco (R-26), com cada jogador vendo
a própria mão e **não** a do parceiro.

## Por que

A mão de ferro decide a partida inteira. Na variante fechada, a partida — e a aposta em
moedas — é resolvida por sorteio puro: ninguém tem decisão a tomar, só ordem de clique. Num
produto web isso tem duas consequências piores que a perda de folclore: o jogador atribui a
derrota a azar em vez de jogo, e a tela "você não pode ver a sua própria carta" é lida como
defeito, não como regra.

## Alternativa descartada

A fechada, que é a mais citada (F1, F4). Descartada pelo acima — e a escolha é **barata de
reverter**: é um `bool` na configuração da mesa, porque a única diferença no código é se a
mão do jogador vai ou não na mensagem de início da mão.

## Quem paga se eu estiver errado

O jogador tradicionalista, que vai dizer que mão de ferro é no escuro. Pagamento: um sinalizador
de configuração. Está em `riscos_conhecidos`.
