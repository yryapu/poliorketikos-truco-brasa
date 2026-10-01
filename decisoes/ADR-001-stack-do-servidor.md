# ADR-001 — A stack do servidor: axum + tokio + sqlx/SQLite

**Data:** 2026-10-01 · **Estado:** VIVO

Rust no servidor já veio decidido pelo enunciado, e é a única decisão técnica dada. O resto é
meu, e é isto. Versões conferidas em `crates.io` no dia (comando no fim), não de memória.

## O cenário, antes das bibliotecas

O que este servidor realmente faz: (a) mantém N conexões WebSocket abertas por muito tempo,
com mensagens pequenas e frequentes; (b) guarda estado de partida em memória, mutável, com
donos concorrentes; (c) escreve dinheiro de jogo e placar em disco, com ACID, porque saldo
perdido é bug visível; (d) dispara HTTP de saída para webhooks de terceiros, que podem estar
lentos ou mortos. Isso é um serviço **I/O-bound com estado em processo**. Toda a escolha abaixo
sai dessa frase.

| peça | escolhida | versão | por que ela, e não a outra |
|------|-----------|--------|---------------------------|
| runtime | `tokio` | 1.53 | Único runtime async com ecossistema completo para isto. 1,0 bilhão de downloads. `async-std` está descontinuado; `smol` é bom e menor, mas obrigaria a abandonar o resto da tabela. |
| HTTP + WS | `axum` | 0.8 | WebSocket **de primeira classe** (`axum::extract::ws`) no mesmo roteador do HTTP, então o *upgrade* herda o cookie de sessão sem nenhuma ponte. Sobre `hyper`/`tower`, que é o que o resto do ecossistema fala. Descartado: `actix-web` (ótimo, mas o modelo de ator duplicaria o meu próprio modelo de mesa, e paga `unsafe` que eu não quero auditar); `tokio-tungstenite` cru (eu teria que escrever o roteamento HTTP e o servidor de estáticos à mão); `poem`/`salvo` (menos olhos). |
| serialização | `serde` + `serde_json` | 1.0 | Padrão de fato. JSON e não binário/MessagePack porque o cliente é o navegador e o protocolo precisa ser legível na aba *Network* — e porque a carta é um **caractere Unicode**, que em JSON é só uma string e em binário viraria um esquema de codificação que eu inventaria (exatamente o que o enunciado proíbe). |
| banco | `sqlx` + **SQLite** | 0.9 | Assíncrono de verdade, SQL conferido em tempo de compilação, migrações embutidas. SQLite porque saldo e ranking precisam de **ACID e durabilidade**, e nada aqui precisa de um segundo processo: um arquivo. Descartado: **Postgres** — compra concorrência de escrita e replicação que esta v1 não exerce, e custa um container, uma rede e uma senha; **só memória** — perde o saldo no restart, e saldo que evapora é o oposto do que foi pedido; **`rusqlite`** — bloqueante, exigiria `spawn_blocking` em todo acesso. |
| senha | `argon2` | 0.6 | Argon2id é a recomendação atual da OWASP para hash de senha; implementação pura em Rust da `RustCrypto`. Descartado: `bcrypt` (aceitável, mas sem resistência a GPU comparável) e qualquer coisa com SHA cru (errado por construção). |
| sessão | cookie opaco + tabela | — | Ver ADR-002. |
| aleatório | `rand` + `ChaCha20` | 0.10 | O embaralhamento decide dinheiro de jogo: precisa ser **CSPRNG semeado pelo SO**, não um Mersenne Twister. `rand::rngs::StdRng`/`ChaCha20Rng` com `from_os_rng`. Descartado: `fastrand` — rápido e previsível, e previsível aqui é trapaça. |
| HTTP de saída | `reqwest` | 0.13 | Para os webhooks. Cliente com *timeout* e pool, sobre o mesmo `hyper`. Descartado: `hyper` cru (eu reescreveria pool e redirect). |
| assinatura de webhook | `hmac` + `sha2` | 0.13 / 0.11 | HMAC-SHA256 no corpo, como GitHub e Stripe fazem. Ver ADR-004. |
| log | `tracing` + `tracing-subscriber` | 0.1 | Log estruturado com *spans*, que é o que serve para seguir uma partida por N mensagens. |
| id | `uuid` v4/v7 | 1.26 | Id de partida e de jogador sem coordenação. |

## O custo central que esta decisão aceita

**SQLite serializa as escritas.** Um processo, um arquivo, um escritor por vez. Esta v1
aceita isso porque a escrita é rara (fim de mão, fim de partida, registro) e o caminho quente
— jogar carta — não toca o disco. **Gate de reversão:** se a escrita passar de ~algumas
centenas por segundo, ou se precisar de um segundo processo de servidor, troca-se por
Postgres; `sqlx` mantém o mesmo código de consulta, então a reversão é de `Cargo.toml` e
migração, não de desenho.

## Comando que gerou as versões

```sh
for c in axum tokio sqlx serde argon2 uuid rand reqwest hmac sha2 tracing; do
  curl -sS "https://crates.io/api/v1/crates/$c" -H "User-Agent: truco-brasa" \
  | python3 -c "import sys,json;d=json.load(sys.stdin);print(d['crate']['name'], d['crate']['max_stable_version'])"
done
```
