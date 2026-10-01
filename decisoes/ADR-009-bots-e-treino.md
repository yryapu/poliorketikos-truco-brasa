# ADR-009 — Bots: o bot é um cliente, e mesa com bot não vale nada

**Data:** 2026-10-01 · **Estado:** VIVO

Pedido depois da v1: "crie bots pra podermos conseguir jogar". O problema real era que o
jogo exigia duas pessoas para qualquer partida, e uma só não conseguia nem ver a mesa.

## Decisão 1 — o bot é um cliente, não um caso especial do servidor

O bot recebe exatamente a **mesma `Visao`** que um humano no mesmo assento receberia, pelo
mesmo tipo de canal, e responde com as mesmas cinco mensagens do protocolo. Não há acesso ao
`Partida`, nem canal privilegiado, nem função que lhe conte mais.

**Isso não é elegância, é a propriedade de segurança.** O bot fica *incapaz* de ver a carta do
adversário, em vez de apenas não programado para ver. Se amanhã alguém piorar a estratégia,
continua impossível trapacear — e o teste de isolamento que vale para o humano vale para ele
sem uma linha a mais.

**Alternativa descartada:** dar ao bot o `&Partida` e deixá-lo calcular direto. Seria mais
simples de escrever e mais fácil de otimizar. Descartada porque torna a honestidade do bot uma
questão de disciplina do código, e disciplina não sobrevive a seis meses de manutenção.

## Decisão 2 — mesa com bot é treino: não vale moeda, não conta ranking

Aposta forçada a **zero no servidor** (não validada — forçada), nenhuma moeda movida, nenhuma
partida, vitória, derrota ou emblema registrado.

Dois motivos, e os dois são de integridade:

1. **Moeda saindo de um bot é inflação.** O bot não tem ficha no banco, então não há saldo de
   onde a moeda do vencedor sairia. Qualquer pagamento aqui cria dinheiro.
2. **Vitória contra bot no ranking é o farm que a v1 já declarou como risco.** O risco
   conhecido nº 2 da entrega é "duas contas jogando entre si para farmar reputação". Deixar
   treino pontuar seria *implementar* esse farm e chamá-lo de funcionalidade.

O tipo é quem garante: `Ocupante` é `Humano(Jogador)` ou `Bot { apelido }`, e só o primeiro
tem ficha. Pagamento e estatística passam por `ocupante.humano()`, que devolve `None` para
bot — então não existe o `if` que alguém pode esquecer.

**Alternativa descartada:** deixar apostar contra bot e criar as moedas. Tentador porque é o
que muitos jogos casuais fazem. Descartada porque o enunciado desta v1 diz que o saldo "é só
do jogo", e um saldo que infla a cada treino deixa de medir qualquer coisa — inclusive o
ranking, que é ordenado por vitórias e desempatado por moedas.

## O custo central que esta decisão aceita

**Quem só joga treino não sobe no ranking nem ganha emblema.** Aceito: a alternativa corrompe
a única coisa que o ranking mede. O jogador vê um selo "treino: não vale moeda nem ranking"
fixo na mesa, então não descobre isso no fim.

**Gate de reversão:** se treino virar o modo dominante e as pessoas quiserem progresso nele,
a saída é um **segundo** eixo de reputação (nível de treino), separado do ranking de partidas
entre pessoas. Nunca misturar os dois no mesmo número.

## Estratégia do bot, e por que ela é simples de propósito

Cobre a carta alheia com **o mais barato que vence**; descarta a mais fraca quando não pode
vencer, e **de costas** quando a regra permite (o que esconde do adversário o quanto a mão era
ruim); puxa com a mais forte, porque levar a primeira rodada dá a vantagem de empate nas
outras duas. Aceita truco com mão que mata, corre com mão fraca, pede com duas cartas fortes —
e só 60% das vezes, porque um bot que pede sempre que pode fica legível em três mãos.

Não é um bot bom. É um bot **honesto e terminante**, e isso é o que o teste prova.

## Cai se

Alguém mostrar que o bot é tão fraco que treinar contra ele ensina o jogo errado. Aí a
estratégia melhora — e o desenho não muda, porque a estratégia é uma função de `Visao` para
`Acao` e nada mais depende dela.
