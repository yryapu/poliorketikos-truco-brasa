# R03 — Eu mandei a manilha no protocolo como uma carta. Não é uma carta.

**Quando:** 2026-10-01. **Como achei:** o teste de interface (Playwright) falhou com

```
Error: a tela da bia mostrou 🃔, que é da ana
```

## O defeito

A mensagem `estado` levava `"manilha": "🃖"` — o caractere da carta daquele número no naipe
de **paus**. A intenção era inocente: a interface precisa mostrar "a manilha desta mão é o
6", e eu tinha um jeito pronto de desenhar carta.

Duas coisas erradas, e a segunda é pior que a primeira.

1. **Mente para o jogador.** O indicador desenha o 6 de paus — o *zap*, a carta mais forte
   do jogo. Quem olha a mesa lê "o zap está em jogo", que é informação que ninguém tem.
2. **Colide com a mão de alguém.** Quando um jogador tem justamente aquela carta de paus,
   o glifo que aparece na tela de **todos** é idêntico à carta dele. Não é vazamento de
   verdade — o servidor não disse de quem é —, mas é indistinguível de vazamento, tanto
   para o jogador quanto para o teste. E num jogo de informação oculta valendo saldo, "parece
   vazamento" já é defeito.

## O conserto

`manilha` passa a ser o **número**: `"4"`, `"Q"`, `"A"`. É exatamente o que a regra torna
público (R-04: a manilha é o número seguinte à vira, e a vira está aberta na mesa), e nada
além disso.

## Por que os 46 testes em Rust não pegaram

Porque o teste de isolamento fazia **a pergunta errada**:

> a visão do assento `i` contém alguma carta que está na mão de outro assento?

Isso deixa passar qualquer glifo de carta que não venha da mão de ninguém — e a colisão com
a mão de alguém dependia de qual carta tinha sido distribuída. Era um **teste latente**: ele
teria falhado um dia, com outra semente, sem ninguém entender por quê.

A pergunta certa é a inversa:

> **todo** caractere de carta na visão do assento `i` está no conjunto que ele tem direito de
> ver — a mão dele, a vira, e as cartas abertas na mesa?

Esse é o teste que agora existe (`a_visao_so_contem_as_cartas_que_aquele_assento_tem_direito_de_ver`),
e eu **verifiquei que ele pega o defeito**: revertendo o código, ele falha de forma
determinística apontando `🃓`; com o conserto, passa.

## A lição de método, que vale mais que o conserto

Invariante de segurança se escreve como **lista branca**, não como lista negra. "Não contém
o que é proibido" depende de eu ter imaginado todas as formas do proibido. "Contém apenas o
que é permitido" não depende da minha imaginação — e foi a diferença entre um teste que
passava por sorte e um que falha por construção.

E: o teste de interface não foi redundante com os de protocolo. Ele achou o que eles não
podiam achar, porque ele olha o **glifo na tela**, que é onde o jogador de verdade mora.

## Cai se

Alguém mostrar que mostrar a manilha como carta de paus é convenção estabelecida em mesa
física ou em outro produto de truco — aí volta a ser escolha de interface, e o conserto
passa a ser só não deixar o glifo colidir (por exemplo, desenhando o número grande em vez de
uma carta).
