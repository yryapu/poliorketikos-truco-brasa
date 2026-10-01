# ADR-004 — Webhooks: HMAC-SHA256, allowlist de destino, e entrega best-effort

**Data:** 2026-10-01 · **Estado:** VIVO

"Um jogador ou integrador registra uma URL e recebe eventos de partida (começou, terminou,
resultado)." Três eventos, e cada um é um `POST` com JSON.

## Decisão

- **Registro:** `POST /api/webhooks` com `{url}`. O servidor gera um **segredo** de 32 bytes
  e o devolve **uma única vez**, na resposta do registro. Guarda só o hash? Não — aqui guarda
  o segredo em claro, porque ele é necessário para **assinar** cada entrega. Isso é uma
  diferença real em relação a senha e token de sessão, e está registrada como risco conhecido.
- **Assinatura:** cada entrega leva `X-Truco-Assinatura: sha256=<hex>` = HMAC-SHA256 do corpo
  exato, com o segredo. Mais `X-Truco-Evento` e `X-Truco-Entrega` (uuid). É o esquema que
  GitHub e Stripe usam, e o integrador verifica com três linhas em qualquer linguagem.
- **Eventos:** `partida.comecou`, `partida.terminou`. O "resultado" do enunciado **não** é um
  terceiro evento: ele é o corpo de `partida.terminou` (quem ganhou, placar, aposta, saldos).
  Um evento vazio chamado `resultado` seria camada sem capacidade.
- **Entrega:** `tokio::spawn`, fora do caminho da partida. *Timeout* de 5 s, **uma** nova
  tentativa após 2 s. Falha definitiva é registrada, não propagada: **o webhook de um
  integrador lento não pode travar uma mesa.**

## A parte que é segurança, não funcionalidade

Um endpoint que aceita URL arbitrária e faz o servidor buscá-la é **SSRF por desenho**. Sem
guarda, um jogador registra `http://169.254.169.254/...` ou `http://127.0.0.1:8080/api/...` e
usa o meu servidor como procurador contra a minha própria rede. Então:

- só `https://` (e `http://` apenas quando o servidor roda em modo de desenvolvimento);
- o host é **resolvido** e cada IP resultante é verificado: recusa laço (`127/8`, `::1`),
  privado (`10/8`, `172.16/12`, `192.168/16`), link-local (`169.254/16`, `fe80::/10`),
  `0.0.0.0/8`, multicast e ULA;
- sem seguir redirecionamento (redirect é como se escapa de uma allowlist de destino);
- limite de webhooks por jogador.

**Conheço o furo que sobra:** entre a minha resolução de DNS e a conexão do `reqwest` há uma
segunda resolução — é a janela de *DNS rebinding* (TOCTOU). Fechá-la exige conectar ao IP já
validado e passar o `Host` à mão. Fica em `riscos_conhecidos` em vez de fingir que não existe.

## Alternativa descartada: fila persistente com repetição exponencial

Descartada para a v1. Entrega garantida pediria tabela de entregas, estado, tentativas e um
trabalhador — uma camada inteira para três eventos de um jogo de cartas. A v1 entrega
*best-effort* e **diz** que é best-effort. **Gate de reversão:** no primeiro integrador que
precise de garantia, a tabela entra; a assinatura e o formato do corpo não mudam, então a
troca é interna.
