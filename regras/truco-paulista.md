# Truco paulista — especificação normativa

Cada regra abaixo tem **uma etiqueta** (`R-xx`), a **fonte** de onde veio, e — onde a fonte
não cobre o caso — a marca `[DECISÃO]` com o porquê. O código do repositório operacional
cita esta etiqueta no teste que prova a regra. Fontes em `../fontes/00-indice.md`.

> Convenção de leitura: `F1` = PDF do Jogatina, `F2` = regulamento da UFSC, `F3` = MegaJogos,
> `F4` = Wikipédia-pt. `[DECISÃO]` = não encontrado em fonte; é escolha minha, e cai se
> alguém mostrar fonte em contrário.

---

## 1. Baralho

**R-01 — 40 cartas.** Baralho francês de 52 menos os `8`, `9`, `10` e os curingas.
> F1: "ele é disputado com um baralho especial, que não tem as seguintes cartas: '8', '9',
> '10' e curinga." · F2: "No baralho sujo (ou cheio) são 40 cartas, retirando-se apenas os 8,
> 9, 10 e curingas." · F4: "removendo-se as cartas 8, 9 e 10 do baralho, como forma de obter
> as 40 cartas necessárias".

F2 documenta também o *baralho limpo* de 24 cartas (sem 4,5,6,7). **[DECISÃO]** v1 joga só
com o sujo de 40: é o baralho que as quatro fontes tratam como padrão, e o limpo seria um
segundo modo de jogo sem comprar capacidade nenhuma nesta v1.

**R-02 — Ordem de força, crescente, independente de naipe:**

```
4 < 5 < 6 < 7 < Q < J < K < A < 2 < 3
```

> F2: "4 < 5 < 6 < 7 < Q < J < K < Ás < 2 < 3" · F1: "O valor das cartas, do maior para o
> menor é 3, 2, A, K, J, Q, 7, 6, 5, 4 […] o '2' é mais forte que o 'A', e a 'Q' é mais fraca
> que o 'J'." · F3: "3 2 1 K J Q 7 6 5 4" · F4: "3 (Terno), 2 (Duque), A (Ás), K (Rei), J
> (Valete), Q (Dama), 7, 6, 5 e 4."

As quatro fontes concordam, **inclusive na inversão contraintuitiva `Q < J`** (a Dama é mais
fraca que o Valete). Essa inversão é a regra mais fácil de errar de cabeça, e é por isso que
ela tem teste próprio.

**R-03 — Naipe não desempata carta comum.** Duas cartas comuns de mesmo número empatam, seja
qual for o naipe.
> F2: "O naipe da carta só é usado como critério de desempate nas rodadas em que há mais de
> uma manilha na mesa. No caso de empate onde as cartas mais altas da mesa não são manilhas,
> não há desempate e a rodada é considerada empatada."

---

## 2. Vira e manilha

**R-04 — A vira define a manilha.** No começo da mão vira-se uma carta; as quatro cartas do
número **imediatamente superior** na ordem de R-02 são as manilhas da mão.
> F2: "A carta imediatamente superior a carta virada determina a manilha da mão." · F1: "se
> uma carta '5' for a vira da rodada, as manilhas serão os '6'".

**R-05 — A ordem é circular: vira `3` ⇒ manilha `4`.**
> F1: "Atenção: quando a vira for o '3', as manilhas são as cartas '4'." · F2: "Se a carta
> virada for um 3, as manilhas são as cartas com 4."

**R-06 — Manilha bate qualquer carta comum,** inclusive o `3`.
> F1: "as manilhas são as cartas mais fortes de cada rodada, com força para bater o '3'."

**R-07 — Entre manilhas, desempata o naipe:** `ouros < espadas < copas < paus`.
> F2: "6 de Ouro < 6 de Espadas < 6 de Copas < 6 de Paus" · F1: "Paus > Copas > Espadas >
> Ouros" · F3: "Paus (Zap/Gato), Copas (Copeta), Espadas (Espadilha), Ouros (Mole/Pica-fumo)".

Nomes, por F2: paus = **zap**, copas = **copeta**/escopeta, espadas = **espadilha**,
ouros = **picafumo**.

**R-08 — O naipe da vira é irrelevante.** A vira define só o número.
> F4: "O naipe da carta virada não tem importância."

F2 documenta a variante **manilha fixa** (`7 ouros < A espadas < 7 copas < 4 paus`).
**[DECISÃO]** v1 implementa só manilha variável: é o que define *truco paulista* em oposição
ao mineiro, e é o que as quatro fontes descrevem como regra principal.

---

## 3. Estrutura da mão

**R-09 — Três cartas por jogador, melhor de três rodadas.**
> F2: "cada jogador recebe 3 cartas viradas do baralho. Cada mão é composta por 3 rodadas."

**R-10 — Ganha a mão quem vencer duas rodadas, ou vencer uma e empatar outra.**
> F2: "Vence a mão a dupla que ganhar 2 rodadas da mão ou vencer uma e empatar outra."

**R-11 — Tabela de empate**, que é R-10 desdobrada caso a caso:

| caso | quem leva a mão | fonte |
|------|-----------------|-------|
| empate na 1ª rodada | quem vencer **primeiro** uma das seguintes (2ª, ou 3ª se a 2ª também empatar) | F2, F1, F3, F4 |
| empate na 2ª, 1ª decidida | quem venceu a **1ª** | F2, F1, F3 |
| empate na 1ª e na 2ª | quem vencer a **3ª** | F1, F2, F4 |
| empate na 3ª, 1ª decidida | quem venceu a **1ª** | F1, F2 |
| as três empatadas | **ninguém pontua**; passa-se à mão seguinte | F1, F2, F3 |

> F1: "Se houver empate na primeira rodada, o vencedor da segunda ganha a mão; […] Se houver
> empate na terceira rodada, o vencedor da primeira ganha a mão; Se todas as três rodadas
> empatarem, ninguém ganha ponto." · F2: "Se todas as 3 rodadas de uma mão terminarem
> empatadas, ela é finalizada sem dupla vencedora."

**R-12 — Quem puxa cada rodada.** A 1ª rodada da mão é puxada pelo *mão*. A 2ª e a 3ª são
puxadas por **quem venceu a rodada anterior**; se a rodada anterior empatou, por **quem pôs
na mesa a primeira carta do empate**.
> F2: "A segunda e terceira rodada de uma mão começam por aquele que venceu a rodada
> anterior. No caso da rodada anterior ter empatado, começa por aquele jogador que pôs na
> mesa a primeira carta que empatou."

**R-13 — Rotação do *mão* entre as mãos.** A cada mão nova, o direito de puxar anda um
assento.
> F2: "Nas demais mãos, o jogador que começa a primeira rodada é sempre o da esquerda ao que
> começou a mão anterior."

**[DECISÃO]** sobre o sentido: F2 diz que a vez anda **anti-horária** ("o próximo a jogar é o
que está a direita do que jogou") e que o *mão* seguinte é "o da esquerda" — ou seja, a
rotação do *mão* andaria **contra** o sentido da jogada. Pode ser imprecisão de redação da
fonte. Numa mesa virtual sem esquerda nem direita física, a distinção não existe: adoto
assentos `0,1,2,3` com duplas `{0,2}` e `{1,3}` (parceiro sempre à frente, por F2), jogada em
ordem crescente de assento módulo N, e *mão* da mão `k` = assento `k mod N`. Isso preserva o
que a regra **compra** (parceiro oposto, vez rodando, *mão* rodando) e descarta só a
lateralidade, que não é observável aqui.

**R-14 — Carta de costas ("esconder carta"), só na 2ª e 3ª rodadas.** O jogador pode pôr a
carta virada; ela não é revelada e é desconsiderada na disputa da rodada.
> F2: "Nas segunda e terceira rodada de uma mão há a opção do jogador na sua vez jogar a
> carta virada de costas. Neste caso, o valor dela não é revelado para os demais jogadores e
> ela é desconsiderada na hora de se avaliar qual carta ganhou a rodada." · F1: "não é
> permitido esconder a carta na primeira rodada de cada mão."

**[DECISÃO]** se *todas* as cartas de uma rodada forem de costas, a rodada empata — é o
resultado forçado por "desconsiderada na disputa" quando não sobra carta para disputar.
Nenhuma fonte trata o caso.

---

## 4. Truco, seis, nove, doze

**R-15 — Escada obrigatória e crescente: `1 → 3 → 6 → 9 → 12`.** Não se pula etapa nem se
volta atrás. Não existe mão valendo outro número.
> F2: "só é possível pedir truco se a mão está valendo 1 ponto, assim como só se pede 6 se
> ela já vale 3, só se pede 9 se ela já vale 6 e só se pede 12 se ela já vale 9. Não é
> possível pular nenhuma destas etapas e nem voltar atrás. […] Não existem outros valores de
> pontuação ganha por uma mão além destas: 1, 3, 6, 9 ou 12."

**R-16 — Só se pede na própria vez, antes de jogar a carta.**
> F2: "Qualquer jogador pode na sua vez pedir truco." · F1: "É um pedido de aumento de aposta
> que só pode ser feito na vez do jogador." · F3: "Jogadores podem pedir truco apenas na sua
> vez, antes de jogar."

**R-17 — Três respostas: correr, aceitar, retrucar.**
> F2: "Existem 3 opções de resposta a um pedido de truco: Correr […] Aceitar […] Retrucar".

**R-18 — Quem corre entrega o valor *anterior* ao pedido.** Correr de um truco dá 1 ponto a
quem pediu; correr de um seis dá 3; de um nove, 6; de um doze, 9.
> F2: "a dupla desiste da mão, a que pediu truco ganha imediatamente 1 ponto" e, para 6/9/12,
> "a que pediu truco ganha imediatamente o valor atual da mão (3, 6 ou 9)."

**R-19 — Não se retruca duas vezes seguidas.** Só pode pedir o aumento seguinte a dupla que
**não** fez o último pedido.
> F2: "O pedido de 6, 9 e 12 só pode ser feito pela dupla que não fez o último pedido de
> truco ou retruco na mão." · F3: "Uma dupla não pode aumentar duas vezes consecutivas."

**R-20 — No doze não há retruco:** só aceitar ou correr.
> F2: "No pedido de 12 não existe a opção de retrucar, só aceitar ou correr."

**R-21 — O valor não atravessa a mão.** A mão seguinte começa valendo 1 (exceto mão de onze).
> F2: "Um pedido de truco ou retruco aceito só vale para a mão atual."

**[DECISÃO]** quem responde: F2 diz que "qualquer um dos jogadores pode responder o pedido e
vale a primeira resposta dada". Na v1 **só quem está na vez de jogar** responde, porque
"vale a primeira resposta" num servidor é uma corrida entre dois WebSockets — e resolver
corrida de rede para reproduzir uma ambiguidade de mesa física é complexidade que não compra
capacidade. Custo aceito: o parceiro não pode responder no lugar de quem está na vez.
Registrado em `decisoes/ADR-005-quem-responde-o-truco.md`.

---

## 5. Mão de onze e mão de ferro

**R-22 — Mão de onze.** Quem chega a 11 pontos, no começo de cada mão, escolhe jogar ou
correr, **depois** de ver as cartas e a vira e **antes** da primeira carta na mesa.
> F2: "Quando uma dupla chega a onze pontos, ela está com a mão de onze. A partir deste
> momento, no começo de cada mão esta dupla pode escolher se continua com a mão ou não. […] A
> decisão […] é feita após cada jogador receber suas 3 cartas e a manilha ter sido
> determinada, mas necessariamente antes que o primeiro jogador ponha a primeira carta."

**R-23 — Aceita vale 3; corre dá 1 ao adversário.**
> F2: "Se a mão for aceita, ela já começa valendo 3 pontos […] Se a mão for recusada, a dupla
> rival que não está com 11 pontos recebe imediatamente 1 ponto a mais." · F1 e F3 idem.

**R-24 — Na mão de onze não se pede seis.** Ela vale 3 se aceita, 1 se recusada, e nada mais.
> F2: "Na mão de onze não é possível pedir 6, ou seja, ela sempre vale 3 se for aceita ou 1
> se for recusada."

**R-25 — Na mão de onze a dupla vê as cartas entre si.** Só nesse momento do jogo.
> F2: "a dupla que está com onze pontos pode ver entre si as cartas. Isso só é possível neste
> momento do jogo." · F4: "Na Mão de 11 é permitido olhar as cartas do parceiro."

**R-26 — Mão de ferro: as duas duplas com 11.** Ninguém decide nada, vale 1 ponto, não se
pede truco, e quem ganhar a mão ganha a partida.
> F2: "Joga-se a mão valendo 1 ponto, sem as duplas verem as cartas entre si e sem
> possibilidade de pedido de truco. Vence a partida a dupla que vencer a mão de ferro."

**[DECISÃO]** mão de ferro **aberta**: cada jogador vê as próprias cartas. F2 oferece as duas
variantes ("a mão de ferro é aberta […] ou é fechada"), e F1/F4 descrevem a fechada (jogar
"no escuro"). Escolho aberta porque na fechada o jogo decisivo da partida vira sorteio puro —
e porque num cliente web "você não pode ver a sua própria carta" é a tela mais fácil de o
jogador ler como bug. Cai se o avaliador considerar a fechada canônica. Registrado em
`decisoes/ADR-006-mao-de-ferro-aberta.md`.

Nota: o valor da mão de ferro é **inócuo** no placar — ambas as duplas estão em 11, então
`11 + 1 = 12` encerra a partida de todo modo. F2 (1 ponto) e F3 ("vale 3") discordam sem
consequência observável.

---

## 6. Fim de partida

**R-27 — Vence quem chegar a 12.** "Chega ou passa" — e pode acontecer sem passar pela mão de
onze, por exemplo 9×9 com uma mão trucada valendo 3.
> F2: "A partida termina quando a primeira dupla chega ou passa de 12 pontos. Isso pode
> acontecer mesmo que ela não passe pela mão de onze."

---

## 7. O jogo 1x1 — tudo aqui é `[DECISÃO]`

**Nenhuma das quatro fontes descreve truco paulista para dois jogadores.** F1 e F2 só falam
de quatro em duplas; F3 diz "o jogo clássico é em duplas". Não vou apresentar como regra
pesquisada o que não achei. O enunciado pede 1x1, então decido, e marco:

- **R-28 [DECISÃO]** 1x1 usa o mesmo baralho de 40, 3 cartas cada, mesma vira, mesma ordem.
  Sobram 33 cartas no monte — irrelevante, o monte não é usado depois da vira (F2: "O baralho
  é misturado no começo da cada mão, mas não entre as rodadas").
- **R-29 [DECISÃO]** cada jogador é uma "dupla" de um. Toda regra de dupla vale com cardinalidade 1.
- **R-30 [DECISÃO]** mão de onze em 1x1: o jogador vê as **próprias** três cartas e decide.
  R-25 ("ver as cartas do parceiro") é vazia quando não há parceiro, e vira exatamente o
  conteúdo informacional que a regra pretendia dar: decidir sabendo a mão.
- **R-31 [DECISÃO]** R-19 (não retrucar duas vezes seguidas) vale igual: num 1x1 ela já cai
  naturalmente, porque os pedidos alternam entre os dois jogadores.

Alternativa descartada: inventar um "truco de dois" com 2 cartas ou manilha fixa, como
algumas casas fazem. Descartada porque multiplicaria as regras a provar sem nenhuma fonte
que as sustente — e regra de jogo sem fonte é exatamente o que o enunciado proíbe.

---

## 8. Representação da carta no protocolo

Exigência do enunciado: a carta trafega como **o próprio caractere Unicode**. O bloco é
*Playing Cards* (`U+1F0A0`–`U+1F0FF`), que existe desde o Unicode 6.0.

- bases de naipe: espadas `U+1F0A0`, copas `U+1F0B0`, ouros `U+1F0C0`, paus `U+1F0D0`;
- deslocamento do número: `A`=1, `2`…`7`=2…7, `J`=0xB, `Q`=0xD, `K`=0xE;
- costas da carta (R-14): `U+1F0A0` 🂠 `PLAYING CARD BACK`.

**Achado que quase virou defeito:** o bloco tem uma figura a mais que o baralho francês — o
`KNIGHT` em `0x_C` (🂬 🂼 🃌 🃜). Mapear "a J-ésima figura" por índice sequencial colocaria a
Dama no Cavaleiro e deslocaria o Rei. O deslocamento de `Q` é `0xD`, **não** `0xC`.

Conferido contra o próprio exemplo do enunciado — `🂡 🂱 🃁 🃑` — que é exatamente
`U+1F0A1`, `U+1F0B1`, `U+1F0C1`, `U+1F0D1`, os quatro ases, um por naipe, na ordem
espadas-copas-ouros-paus. Comando que fecha a conferência:

```sh
python3 -c "import unicodedata as u; [print(hex(ord(c)), u.name(c)) for c in '🂡🂱🃁🃑🂠🃝🃛🃞']"
```

As 40 cartas, geradas por essa regra:

```
espadas  4🂤 5🂥 6🂦 7🂧 Q🂭 J🂫 K🂮 A🂡 2🂢 3🂣
copas    4🂴 5🂵 6🂶 7🂷 Q🂽 J🂻 K🂾 A🂱 2🂲 3🂳
ouros    4🃄 5🃅 6🃆 7🃇 Q🃍 J🃋 K🃎 A🃁 2🃂 3🃃
paus     4🃔 5🃕 6🃖 7🃗 Q🃝 J🃛 K🃞 A🃑 2🃒 3🃓
```
