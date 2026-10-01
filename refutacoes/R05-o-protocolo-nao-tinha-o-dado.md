# R05 — "Não dá para ver o que o outro jogou": o dado não existia no protocolo

**Quando:** 2026-10-01, depois de o operador jogar. **Como achei:** ele me disse.

> "dps q jogo a carta nao da pra ver oq o outro usuario jogou na sequencia, deve ficar claro
> e ficar em tela pra vermos quem ganhou cada vez e com quais cartas, deve ter o historico
> pra acompanhar/filtrar"

## O que estava errado

Duas coisas, e a segunda é a que importa.

**1.** O servidor resolvia a rodada e fazia `self.mao.mesa.clear()` na mesma operação, depois
mandava o estado novo. A carta do adversário aparecia e desaparecia entre dois quadros.

**2.** E — isto é o defeito de verdade — o estado dizia **quem levou** cada rodada e não
**com quais cartas**:

```json
"rodadas": [0, null]
```

A dupla vencedora, e nada mais. **Nenhum cliente, por bem escrito que fosse, conseguiria
manter a rodada em tela**, porque o dado não trafegava. Eu teria olhado o cliente, não achado
nada errado nele, e estaria certo: o erro era meu, uma camada abaixo.

## O conserto

- `Rodada` passa a guardar as jogadas; `estado.rodadas` vira uma lista com as cartas, a dupla
  que levou e **o assento que pôs a carta vencedora**.
- Aviso novo `mao_resolvida` com o detalhe inteiro da mão que acabou, incluindo o `valor`.
  Existe porque o estado seguinte já é da mão nova: sem ele, a última rodada de cada mão
  continuaria se perdendo.

`assento_vencedor` vem do servidor porque **o cliente não pode derivá-lo**: saber qual carta
venceu exige a ordem de força e a manilha, e por ADR-003 o cliente não conhece as regras. Sem
esse campo a interface só podia dizer "a dupla X levou", nunca "foi esta carta".

## O que o meu invariante de segurança disse sobre a mudança

O teste de lista branca — *todo glifo de carta na visão de um assento está no conjunto
permitido* — **falhou**, e corretamente: a exposição havia mudado. Carta aberta de rodada
resolvida é legitimamente pública, então a lista permitida cresceu. Mas a recíproca ganhou
asserção própria, porque era exatamente o que a mudança poderia ter quebrado:

> carta jogada de costas continua escondida **depois** de a rodada fechar — R-14 não expira.

Um invariante escrito como lista branca não só impede vazamento: ele **avisa quando a
superfície muda**, e obriga a decidir se a mudança é intencional. A lista negra teria passado
em silêncio.

## A lição de método

O operador relatou um **sintoma de interface**. Eu tinha quatro caminhos: culpar a tela,
remendar a tela guardando estado que o servidor já devia mandar, perguntar o que ele queria,
ou descobrir se o dado existia. Só o quarto acha a causa — e a resposta foi que **o protocolo
era insuficiente por construção**, não que alguém tivesse errado a implementação.

E um corolário desconfortável: sete testes de interface passavam antes disso. Nenhum
verificava que a carta do adversário **permanece** visível, porque eu testei que a partida
funciona e nunca que ela é **acompanhável**. Teste de funcionamento não é teste de
experiência, e a diferença custou um relato de quem jogou.

## Cai se

Alguém mostrar que manter a rodada resolvida em tela confunde mais do que ajuda — aí a
decisão de interface muda, mas o dado continua tendo de existir no protocolo: a escolha de
mostrar ou não é do cliente, e sem o dado ela não existe.
