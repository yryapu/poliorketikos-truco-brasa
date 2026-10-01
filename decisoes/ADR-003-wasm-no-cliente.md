# ADR-003 — **Não** uso WASM no cliente, e o motivo é de segurança, não de preguiça

**Data:** 2026-10-01 · **Estado:** VIVO

O enunciado manda investigar e dizer por quê. Investiguei. A resposta é não.

## O que WASM compraria aqui

A promessa real é **uma regra, um código**: o crate `truco-regras` compilaria para
`wasm32-unknown-unknown` e o navegador avaliaria a força das cartas, a tabela de empate e a
escada do truco com exatamente o mesmo código do servidor. Divergência cliente/servidor em
regra de jogo é uma classe de bug clássica, e WASM a elimina por construção. É um argumento
bom e eu quase comprei.

## Por que não

**1. O servidor tem que ser autoritativo de todo modo.** Num jogo de informação oculta
valendo saldo, nenhuma decisão de regra pode ser do cliente — se eu confiar no cliente para
dizer quem ganhou a rodada, o cliente mente. Então a regra roda no servidor *obrigatoriamente*.
WASM não substituiria essa execução: ela a **duplicaria**. O ganho cai de "uma regra, um
código" para "a mesma regra, rodando duas vezes, e a segunda sem autoridade".

**2. E a duplicação é ativamente perigosa.** Para o cliente avaliar quem ganha, ele precisa
do modelo de baralho. Mandar o motor de regras para o navegador é mandar, junto, a estrutura
que um trapaceiro usa de alavanca — e aumenta a tentação de, "só para a animação ficar
fluida", mandar também cartas que aquele jogador não deveria ver. **O cliente que não sabe as
regras é o cliente que não pode vazar a mão do adversário.** O que trafega para cada jogador
é: as três cartas *dele*, a vira, e as cartas já *jogadas* na mesa. Nada mais.

**3. O que o cliente faz de verdade não é computação.** É DOM, clique e `onmessage`. Nenhuma
dessas três coisas é limitada por CPU. WASM não acelera o que não é cálculo — e um truco tem
40 cartas e 3 rodadas, não é física.

**4. O custo é real e imediato:** `wasm-pack`/`trunk` na build, um alvo a mais no CI, `wasm-bindgen`
para falar com o DOM, e um artefato binário que o avaliador não lê. Hoje o cliente é **um
arquivo HTML servido estático, sem passo de build**. Clonar e `cargo run` basta.

## Conclusão, em uma linha

WASM aqui pagaria *toolchain* e superfície de trapaça para acelerar o que não é lento e
replicar o que não pode ter autoridade. Fica fora.

## Cai se

- a v2 ganhar **modo offline** ou **replay local** de partida, onde o cliente precisa
  genuinamente avaliar regra sem servidor — aí `truco-regras` já está isolado de I/O
  justamente para compilar para `wasm32` sem tocar o resto;
- ou se aparecer cálculo de verdade no cliente (um bot sugerindo jogada, por exemplo).

Note que o crate de regras **não** tem `tokio`, `sqlx` nem `axum` nas dependências. Essa
fronteira é o que mantém esta decisão reversível por 20 linhas em vez de por um refactor.
