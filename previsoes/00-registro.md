# Previsões — o que eu esperava, antes de olhar

Registradas **antes** de cada verificação, com probabilidade, e fechadas com evidência. O
Brier é calculado pelo servidor de previsões, não por mim.

| id | o que eu afirmei | p | deu | Brier | o que eu aprendi |
|----|------------------|---|-----|-------|------------------|
| `0a0914da` | As quatro fontes concordam na ordem das cartas e na tabela de empate, e discordam em ≥1 ponto de pontuação | 0,80 | **sim** | — | Discordaram exatamente onde eu previ: o PDF do Jogatina diz que o truco vale 2, contra si mesmo e contra as outras três (`refutacoes/R01`). O acordo na ordem das cartas foi total, inclusive no contraintuitivo `Q < J`. |
| `6df81013` | O primeiro `cargo build` falha por mudança de API nas versões novas, não por erro meu de lógica | 0,75 | **sim** | 0,0625 | Quatro erros: dois de API (`rand 0.10` moveu `random_range` para `RngExt`; depois `password-hash 0.6`, `sqlx 0.9` e `hmac 0.13` também mudaram) e dois meus. A previsão estava certa na causa dominante, e **eu estava errado em ter escrito "não por erro meu"** como se fosse exclusivo. |
| `7c33ee40` | O fluxo completo 1x1 por WebSocket funciona na primeira tentativa, sem mudar código | 0,35 | **sim** | 0,4225 | Subestimei feio. O motivo do erro é identificável: eu contei a dificuldade das APIs novas **duas vezes** — uma na previsão do build (onde ela de fato apareceu) e outra na do comportamento, onde ela já tinha sido paga. Lição: previsão sobre etapa posterior não deve reusar o risco que a etapa anterior já consumiu. |
| `a10ba0db` | `docker build` passa de primeira e o container responde | 0,55 | **sim** | 0,2025 | Passou. O que eu tinha errado era de forma e eu peguei antes de rodar: `HEALTHCHECK` em forma exec não interpreta `\|\| exit 1`. |
| `b8647a5e` | Um clone limpo do repo operacional compila e passa os 50 testes, sem arquivo faltando | 0,85 | **sim** | 0,0225 | `git clone` num diretório vazio: 39 arquivos, `cliente/index.html` presente (o `include_str!` depende dele), `cargo test` = 50 ok. O risco que eu tinha em mente era exatamente esse arquivo. |

**Brier medido pelo servidor de previsões:** `itens=5 · brier≈0,15` (o comando é
`previsao brier --by sessao-truco-brasa`). Com quatro itens era 0,1775.

## O padrão que aparece nas quatro

**Três das quatro saíram "melhor que eu esperava", e duas por margem larga.** Isso não é
sorte boa, é *subconfiança sistemática* na fase de construção — o espelho do excesso de
confiança, e igualmente um defeito de calibração. O mecanismo que eu consigo nomear é o de
`7c33ee40`: contar o mesmo risco mais de uma vez, em etapas diferentes.

O que eu **não** posso concluir: que eu sou bem calibrado. Quatro previsões é amostra
pequena demais, e um caderno com quatro acertos em quatro é sinal ruim, não bom — significa
que eu não arrisquei nada perto do limite onde a previsão informa.

## Previsão aberta que esta v1 não fecha

- A guarda de destino de webhook tem uma janela de *DNS rebinding* (TOCTOU) entre a minha
  resolução e a do `reqwest`. **p = 0,9 de que um atacante com controle de DNS consegue
  atravessá-la.** Não fechei porque fechar exige conectar ao IP já validado e passar o `Host`
  à mão. Está em `riscos_conhecidos`, não na lista de feitos. Uma revisão de segurança
  automática do ambiente apontou exatamente este item de forma independente — o que é
  confirmação, não novidade.
