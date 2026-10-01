# R06 — Nenhum dos meus 64 testes lê um rótulo como gente lê

**Quando:** 2026-10-01, tirando as capturas de tela para o README. **Como achei:** olhando as
imagens.

## Os quatro defeitos, todos visíveis, nenhum detectável pela suíte

| o que a tela dizia | o que está errado |
|---|---|
| **"Mão 0 vale 1"** | `numero_da_mao` é índice no protocolo (começa em zero) e vazava cru para a tela. Para quem joga, a primeira mão é a 1. |
| **"Eles fez a mão: 3 ponto(s)"** | "Nós" e "Eles" são plurais e pedem verbo no plural; e `pontos` tem de concordar com o número. |
| **"Dupla 1 venceu. Placar 5 x 14."** | Obriga quem lê a lembrar qual dupla é a dele, no momento em que ele quer saber se ganhou. O resto da interface já usa "Nós × Eles". |
| emblema *Batismo* com `🃏` | O coringa é do bloco *Playing Cards* e renderiza monocromático; ao lado de `🌱 ⭐ 🔥 🛡️ 💎`, que são emoji de cor, ele some — em miniatura no ranking virava um quadrado. |

## Por que a suíte não pegou, e não ia pegar

A suíte tinha, nesse momento, **57 testes em Rust e 7 de interface**. Todos verificam uma de
duas coisas:

- **existência**: o elemento está lá, tem este `data-teste`, aparece, some;
- **comportamento**: o botão habilita quando `acoes` permite, o clique manda a mensagem
  certa, o saldo fecha, a carta do adversário continua visível.

Nenhuma asserção minha reprova uma frase que está **em português errado**, ou que está certa
mas **faz o leitor trabalhar**. Um `toContainText(/levamos|levaram|empatou/i)` passa
alegremente com "Eles fez a mão".

Isso não é falha da suíte — é o limite da categoria. Uma asserção sobre texto só pode
verificar o que eu já sei que quero; e se eu soubesse que queria "Eles fizeram", eu teria
escrito "Eles fizeram".

## O que torna a imagem um verificador diferente

Olhar a tela renderizada é o único passo em que **eu leio a interface como um estranho**, sem
o modelo mental de quem a construiu. Foi por isso que quatro defeitos que eu passei horas ao
lado de, sem ver, apareceram em dez minutos de olhar PNG.

E vale notar de onde vieram: três dos quatro só existem porque eu **desenhei uma tela para o
README**. Se a entrega não pedisse fotos, os quatro continuariam lá — provados, testados,
verdes, e errados.

## A regra que eu tiro disto

**Antes de declarar pronta uma interface, renderize-a e leia.** Não é "teste manual" no
sentido de clicar por aí: é um passo específico e barato — uma captura por tela, lida como
texto. O custo, aqui, foi de um minuto e meio de execução por rodada.

E o corolário incômodo: a mesma lógica vale para qualquer saída destinada a humanos. Uma
mensagem de erro, um relatório, um `--help`. Eu testo que eles *existem*; não testo que eles
*se leem*.

## Cai se

Alguém mostrar uma forma de asserção automática que reprovasse "Eles fez a mão" sem que eu
tivesse de antecipar a frase certa. Um corretor gramatical no CI chegaria perto para o caso
de concordância — mas não para "Dupla 1 venceu", que é gramaticalmente impecável e mesmo
assim ruim.
