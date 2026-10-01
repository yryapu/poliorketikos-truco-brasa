# poliorketikos-truco-brasa — a pesquisa

Repositório de **pesquisa** do experimento `truco-brasa`: aqui ficam as fontes, as regras com
procedência, as decisões com a alternativa descartada, as previsões feitas **antes** de cada
verificação, e as refutações. O código fica no operacional:
**https://github.com/yryapu/truco-brasa**

## Por que dois repositórios

Porque uma regra de jogo errada invalida todo o código que a implementa, e as duas coisas têm
ciclos de revisão diferentes: **fonte se refuta, código se refatora**. Juntas num repo só, o
`git log` do código enterra a procedência da regra, e seis meses depois ninguém consegue
responder *por que* a manilha é a carta seguinte à vira sem reabrir a pesquisa do zero.
Separadas, a pesquisa é citável e versiona o raciocínio; o operacional versiona o artefato.

**Onde eu discordo da separação:** ela vira dois repos órfãos se o código não apontar de volta.
A ligação é a **etiqueta de regra** (`R-01`…`R-31`): cada regra aqui tem uma, e cada teste lá
cita a sua. Sem essa costura, a separação é só burocracia.

## Mapa

| pasta | o que tem |
|-------|-----------|
| `fontes/` | as fontes **íntegras**, mais `00-indice.md` com como e quando cada uma foi obtida, o sha256, e a hierarquia de autoridade entre elas |
| `regras/` | `truco-paulista.md` — a especificação normativa, cada regra com a citação que a sustenta ou a marca `[DECISÃO]` |
| `decisoes/` | um ADR por decisão de desenho, com o custo aceito e a alternativa descartada |
| `previsoes/` | o que eu esperava antes de cada verificação, com probabilidade, e se acertei |
| `refutacoes/` | onde eu estava errado, ou onde a fonte estava |

## Estado

v1 entregue. A entrega está em
[`resultado.json`](https://github.com/yryapu/truco-brasa/blob/main/resultado.json) do repo
operacional: dez critérios, cada um com o comando que o prova, e dez riscos conhecidos, cada
um com o cenário que o dispara e o nome de quem paga.

Comece pelos riscos. A lista de feitos é a parte fácil de escrever.

| ler | para |
|-----|------|
| [`regras/truco-paulista.md`](regras/truco-paulista.md) | as 31 regras, cada uma com a citação da fonte ou a marca `[DECISÃO]` |
| [`fontes/00-indice.md`](fontes/00-indice.md) | como e quando cada fonte foi obtida, o sha256, e a hierarquia de autoridade |
| [`refutacoes/`](refutacoes/) | seis erros: um da fonte, cinco meus — inclusive um teste que **afirmava** o comportamento defeituoso, e quatro rótulos errados que só apareceram ao olhar a tela renderizada |
| [`previsoes/00-registro.md`](previsoes/00-registro.md) | o que eu esperava antes de olhar, e o Brier |
| [`decisoes/`](decisoes/) | dez ADRs, cada um com o custo aceito e a alternativa descartada |
| [`diario.md`](diario.md) | o caminho, os erros e como cada um foi achado — e o que **jogar de verdade** revelou que 54 testes não tinham achado |
