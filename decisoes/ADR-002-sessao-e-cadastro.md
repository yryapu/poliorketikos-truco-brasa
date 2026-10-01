# ADR-002 — Cadastro em dois campos, sessão por cookie opaco com tabela

**Data:** 2026-10-01 · **Estado:** VIVO

O enunciado pede duas coisas que se puxam: **"entra e começa a jogar em menos de um minuto"**
e **"sessão segura, com os dados de cada jogador isolados dos outros"**.

## Decisão

- **Cadastro:** `apelido` + `senha`. Nada mais. Sem e-mail, sem confirmação, sem captcha.
  Um `POST /api/registrar` cria o jogador com 1000 moedas e **já devolve a sessão** — não
  existe "cadastrou, agora faça login".
- **Sessão:** token opaco de 32 bytes do CSPRNG, em cookie `HttpOnly; SameSite=Strict;
  Path=/`, com `Secure` quando servido em HTTPS. O servidor guarda **o hash SHA-256 do
  token**, nunca o token.
- **Isolamento:** toda consulta que devolve dado de jogador é parametrizada pelo
  `jogador_id` *da sessão*, nunca por id vindo do cliente. A mão de cartas de um jogador só
  é serializada no `WebSocket` dele; o que vai para a mesa é a carta **jogada**.

## Alternativa descartada: JWT

Descartada. Um JWT não se revoga: para expulsar uma sessão (senha trocada, token vazado)
preciso de uma lista de revogação — que é uma tabela de sessões, que é exatamente o que o
cookie opaco já me dá, só que sem assinatura, sem chave para rodar, sem *clock skew* e sem a
superfície de `alg: none`. JWT compra validação sem ir ao banco; aqui eu vou ao banco de todo
jeito, para carregar o saldo. **Compra nada, custa uma chave.**

## Alternativa descartada: só anônimo

"Entra sem cadastro" seria ainda mais rápido, e foi tentador. Descartada porque o enunciado
também pede **ranking e emblemas por histórico**: sem identidade que sobreviva ao fechar o
navegador, não existe histórico. Guest viraria, na prática, um cadastro com apelido gerado —
o mesmo custo, com a perda de o jogador não conseguir voltar à própria conta.

## O custo central que esta decisão aceita

**Sem e-mail não há recuperação de senha.** Quem perder a senha perde o saldo e os emblemas.
Aceito para a v1 porque o saldo "não tem valor fora do jogo" (enunciado) e porque exigir
e-mail custaria o minuto que o enunciado quer economizar. **Gate de reversão:** no primeiro
pedido real de recuperação, entra e-mail opcional — opcional, para não reintroduzir o atrito
em quem não quer.

## Hashes

Senha com **Argon2id** (parâmetros padrão da crate, que seguem a recomendação da OWASP).
Token de sessão com **SHA-256 cru e sem sal** — e isto é deliberado, não desleixo: o token já
é 32 bytes uniformes do CSPRNG, então não há dicionário a atacar e o *stretching* do Argon2
só adicionaria latência em **toda** requisição autenticada.
