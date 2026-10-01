# ADR-007 — Pareamento por modo **e** aposta iguais; bolo único

**Data:** 2026-10-01 · **Estado:** VIVO

O enunciado diz: "todo jogador começa com 1000 moedas e **aposta quanto quiser** na partida".
"Quanto quiser" cria um problema que o enunciado não resolve: se a Ana quer apostar 500 e o
Caio 10, qual é a aposta da mesa?

## Decisão

A fila é indexada por `(modo, aposta)`. Só se pareiam jogadores que escolheram **o mesmo
modo e a mesma aposta**. O bolo é `n × aposta` e vai inteiro para os vencedores, em partes
iguais: cada vencedor recebe `2 × aposta` e termina `+aposta`; cada perdedor termina
`−aposta`. A soma dos saldos da mesa não muda — e isso tem teste
(`o jogo não cria nem destrói moeda`).

A aposta é **debitada na entrada da fila**, não no início da partida, e a condição de saldo
mora dentro do próprio `UPDATE`:

```sql
UPDATE jogadores SET moedas = moedas - ?2 WHERE id = ?1 AND moedas >= ?2
```

Assim não existe janela entre "conferi o saldo" e "cobrei" — nem com duas abas do mesmo
jogador apostando tudo ao mesmo tempo. Quem sai da fila antes de parear recebe de volta, e
só se **ainda estava na fila**: se a mesa já levou a espera, a aposta está em jogo e devolver
seria criar moeda.

## Alternativas descartadas

**Negociar a aposta na mesa.** Cada jogador propõe, a mesa acorda. Descartada: precisa de
uma fase de negociação com temporizador, tela própria e regra de desempate, antes de a
primeira carta sair. Muito atrito para um jogo cuja promessa é "jogar em menos de um minuto".

**Usar a menor aposta da mesa.** Simples e tentador: pareia qualquer um com qualquer um e
cobra o mínimo. Descartada porque transforma a escolha da aposta em mentira — quem escolheu
500 joga por 10 e não entende por quê. Pior: cria incentivo a entrar com aposta alta só para
parear rápido, sabendo que vai pagar o mínimo.

**Faixas de aposta** (0–50, 51–200, …) para encher mesa mais rápido. É a evolução natural
disto, e provavelmente o que a v2 vai querer. Não entra na v1 porque exige a regra de
"quanto cada um paga numa mesa de apostas diferentes", que é a questão que a decisão atual
evita por construção.

## O custo central que esta decisão aceita

**Mesa demora mais a encher.** Uma aposta esquisita (437 moedas) pode nunca parear, e o 2x2
precisa de quatro pessoas com o mesmo número. A v1 aceita porque o conjunto de apostas que as
pessoas realmente escolhem é pequeno (0, 10, 50, 100, 500) e porque a interface sugere essas.
**Gate de reversão:** se a mediana de tempo na fila passar de um minuto, entram faixas — e
aí a questão do "quanto cada um paga" tem de ser resolvida, não evitada.

## Quem paga se eu estiver errado

O jogador que fica na fila sem parear. Ele vê `faltam: N` e pode cancelar, recuperando a
aposta — então o pior caso é tempo perdido, não moeda perdida.
