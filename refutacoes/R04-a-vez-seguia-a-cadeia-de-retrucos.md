# R04 — A vez seguia a cadeia de retrucos. Um teste meu afirmava o erro.

**Quando:** 2026-10-01, construindo os bots. **Como achei:** um **teste de propriedade**
falhou — 400 partidas (200 sementes × 2 modos) com todos os assentos jogados pelo bot,
exigindo que `Partida::aplicar` nunca recuse a ação que o bot escolheu.

```
modo DoisContraDois semente 8: o bot do assento 3 tentou
Jogar { indice: 0, coberta: true } e a regra recusou: você não tem essa carta.
acoes=["jogar", "jogar_coberta"]
```

Índice 0 recusado por "você não tem essa carta" significa **mão vazia**. E o diagnóstico
mostrou algo impossível num jogo legal:

```
vez=3  rodadas=[Some(0), Some(1)]  mesa=[(1,false),(2,true)]
minhas_cartas=[]   cartas_na_mao=[2, 0, 0, 0]
```

Na terceira rodada, com duas cartas na mesa, o assento 0 tinha **duas** cartas e os outros
três tinham **zero**. A ordem de jogada estava corrompida.

## A causa

`aceitar` fazia:

```rust
self.mao.vez = p.assento_pedinte;   // p = a pendência CORRENTE
```

Isso é correto para o **primeiro** pedido: quem pediu truco ainda deve a carta, a vez volta
para ele. Mas numa cadeia de retrucos de tamanho **par**:

| passo | quem | pendência corrente |
|-------|------|--------------------|
| A (vez de A) pede **truco** | A | `pedinte: A` |
| B pede **seis** | B | `pedinte: B` |
| A **aceita** | — | `pedinte: B` ← e a vez ia para **B** |

B nunca deveu carta nenhuma. A vez passava para ele, ele jogava fora de ordem, e dali em
diante as mãos esvaziavam desiguais.

## O conserto

A vez **não se move durante a negociação do truco**. Ela já aponta para quem deve a carta, e
`jogar` recusa enquanto houver pendência — então não havia nada a proteger mudando-a. O
conserto foi apagar a linha.

## A parte que dói: o meu teste afirmava o comportamento errado

`r15_r20_a_escada_inteira_e_o_teto_no_doze` terminava com:

```rust
assert_eq!(p.mao.vez, 1);   // ERRADO
```

Eu havia **codificado o bug no teste**. Nenhum teste de exemplo podia pegá-lo, porque o único
que chegava ao fim da escada afirmava o resultado defeituoso como se fosse o esperado. O
valor certo é `0` — o assento que pediu o primeiro truco é quem devia a carta.

## Por que um teste de propriedade pegou e doze de exemplo não

Os testes de exemplo conferiam **o valor da escada** (`1 → 3 → 6 → 9 → 12`), que estava
certo. Nenhum conferia **de quem era a vez depois dela**, porque eu não imaginei que a
negociação pudesse mexer na ordem de jogada — e um teste de exemplo só verifica o que quem o
escreveu pensou em verificar.

O teste de propriedade não precisou que eu imaginasse nada. Ele afirma uma invariante do
sistema inteiro — *nenhuma ação escolhida a partir de `acoes` é recusada por `aplicar`* — e
deixa 400 partidas aleatórias procurarem o contraexemplo.

**É a diferença entre "eu testei os casos que pensei" e "eu afirmei uma propriedade".** O
custo foi uma função de 40 linhas.

## A segunda lição, sobre de onde veio o teste

Eu escrevi esse teste para provar que **o bot** não trapaceava nem travava. Ele encontrou um
defeito no **motor de regras**, que é código mais antigo e muito mais testado. Construir um
segundo cliente do próprio sistema — mesmo um cliente bobo — exercitou caminhos que nenhum
teste direcionado tinha exercitado.

## Cai se

Alguém mostrar uma casa em que a vez realmente passa a quem retrucou por último. Nenhuma das
quatro fontes diz isso, e F2 é explícita em que o pedido de truco é feito "na sua vez" — o
que pressupõe que a vez não é alterada pelo pedido.
