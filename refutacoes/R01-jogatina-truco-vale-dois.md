# R01 — O PDF oficial do Jogatina diz que o truco vale **dois** pontos. Está errado.

**Quando:** 2026-10-01. **Onde:** `fontes/fonte-01-jogatina-regras-truco-paulista.txt`,
seção "Definições", verbete *Truco*.

## O que a fonte diz

> "**Truco** - É um pedido de aumento de aposta que só pode ser feito na vez do jogador. Se
> nenhum jogador pedir Truco, a partida valerá um ponto. Se o Truco for pedido, e o adversário
> aceitar, **a partida passa a valer dois pontos**. Se o adversário não aceitar, o desafiador
> ganha um ponto."

E, três parágrafos antes, **o mesmo documento** diz:

> "**Mão** – Por definição cada mão vale 1 ponto, mas esse valor pode ser aumentado para
> **3, 6, 9 e até 12** pontos em função do truco."

## Por que eu considero "dois" o erro, e não "três"

1. **A fonte se contradiz internamente**, e o lado "3" aparece duas vezes nela (verbete *Mão*
   e verbete *Seis*: "a partida que inicialmente valia um ponto e foi trucada, passando a
   valer dois, passará a valer seis" — onde o "dois" reaparece, mas o destino `6` só faz
   sentido na escada `1→3→6`).
2. **Nenhuma outra fonte sustenta o 2.** F2: "Um pedido de truco aumenta o valor da mão para
   3." F3: "'Truco' (first request): raises to 3 points." F4: "O Truco é pedido para elevar a
   aposta a três *tentos*."
3. **O 2 não fecha a aritmética da própria escada.** Com truco=2, correr de um *seis* deveria
   entregar 2, mas F1 descreve a entrega do *nove* recusado como 6 e do *doze* recusado como
   9 — valores que só existem na escada `1,3,6,9,12`.

## O que eu implemento

`1 → 3 → 6 → 9 → 12` (R-15), e quem corre entrega o valor anterior ao pedido (R-18).

## Cai se

Alguém mostrar uma casa de truco paulista, ou regulamento de torneio, em que o truco aceito
vale 2 — aí isto deixa de ser erro da fonte e passa a ser **variante regional**, e a v1 passa
a precisar de uma chave de regra em vez de uma constante.

## Como eu achei

Lendo a fonte inteira em vez de confiar na leitura resumida que a ferramenta de fetch me
devolveu primeiro. O resumo automático do PDF não mencionou o "dois" — ele tinha normalizado
para a escada correta, apagando a contradição. **Lição: o resumo de uma fonte não é a fonte,
e salvar a fonte íntegra no repositório foi o que permitiu achar isto.**
