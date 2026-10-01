# Previsões — o que eu esperava, antes de olhar

Registradas **antes** de cada verificação, com probabilidade, e fechadas com evidência. O
Brier é calculado pelo servidor de previsões, não por mim.

| id | o que eu afirmei | p | deu | Brier | o que eu aprendi |
|----|------------------|---|-----|-------|------------------|
| `0a0914da` | As quatro fontes concordam na ordem das cartas e na tabela de empate, e discordam em ≥1 ponto de pontuação | 0,80 | **sim** | 0,04 | Discordaram exatamente onde eu previ: o PDF do Jogatina diz que o truco vale 2, contra si mesmo e contra as outras três (`refutacoes/R01`). O acordo na ordem das cartas foi total, inclusive no contraintuitivo `Q < J`. |
| `6df81013` | O primeiro `cargo build` falha por mudança de API nas versões novas, não por erro meu de lógica | 0,75 | **sim** | 0,0625 | Quatro erros: dois de API (`rand 0.10` moveu `random_range` para `RngExt`; depois `password-hash 0.6`, `sqlx 0.9` e `hmac 0.13` também mudaram) e dois meus. A previsão estava certa na causa dominante, e **eu estava errado em ter escrito "não por erro meu"** como se fosse exclusivo. |
| `7c33ee40` | O fluxo completo 1x1 por WebSocket funciona na primeira tentativa, sem mudar código | 0,35 | **sim** | 0,4225 | Subestimei feio. O motivo do erro é identificável: eu contei a dificuldade das APIs novas **duas vezes** — uma na previsão do build (onde ela de fato apareceu) e outra na do comportamento, onde ela já tinha sido paga. Lição: previsão sobre etapa posterior não deve reusar o risco que a etapa anterior já consumiu. |
| `a10ba0db` | `docker build` passa de primeira e o container responde | 0,55 | **sim** | 0,2025 | Passou. O que eu tinha errado era de forma e eu peguei antes de rodar: `HEALTHCHECK` em forma exec não interpreta `\|\| exit 1`. |
| `b8647a5e` | Um clone limpo do repo operacional compila e passa os 50 testes, sem arquivo faltando | 0,85 | **sim** | 0,0225 | `git clone` num diretório vazio: 39 arquivos, `cliente/index.html` presente (o `include_str!` depende dele), `cargo test` = 50 ok. O risco que eu tinha em mente era exatamente esse arquivo. |

| `6496bd15` | Uma partida 1x1 de treino contra bot chega aos 12 sozinha, sem travar e **sem o bot tentar jogada ilegal**, na primeira execução | 0,60 | **NÃO** | 0,36 | A metade "chega aos 12" deu certo nos três testes de ponta a ponta. A metade "sem jogada ilegal" foi refutada pelo critério que eu mesmo escrevi: o teste de propriedade falhou na semente 8. E a causa não era o bot — era defeito **meu** de regra, em `aceitar` (`refutacoes/R04`). Melhor previsão errada da sessão. |

**Brier medido pelo servidor de previsões, não declarado por mim:**

```
previsao brier --by sessao-truco-brasa   →   itens=6 · brier≈0.1817
previsao abertas --by sessao-truco-brasa →   (vazio)
```

Nenhuma previsão ficou aberta — a primeira quase ficou, e foi um gancho de parada que me
cobrou o fecho. Vale como observação sobre mim: eu fechei as quatro que verifiquei por
comando e esqueci exatamente a que se fechava por **leitura**. A verificação que não tem
saída de terminal é a que escapa.

## O padrão, revisto depois da sexta

As cinco primeiras saíram todas a meu favor, três por margem larga, e eu escrevi aqui que
isso era **subconfiança sistemática**. A sexta errou — e errou na parte que eu não tinha
pensado em duvidar.

Repare na forma do erro, que é mais informativa que o Brier. A previsão tinha **duas
metades**: "chega aos 12" e "sem jogada ilegal". A primeira eu havia modelado; a segunda eu
pus no texto quase como enfeite, e foi ela que caiu. Previsão com duas cláusulas unidas por
"e" é mais fraca que a cláusula mais fraca dela, e eu tinha atribuído a probabilidade
olhando só para a cláusula que me interessava.

**A lição: separe as cláusulas.** "O bot joga até os 12" e "o bot nunca faz jogada ilegal"
são afirmações sobre coisas diferentes — uma sobre terminação, outra sobre correção — e
mereciam duas probabilidades. A segunda eu daria bem menos que 0,6 se tivesse olhado para
ela sozinha, porque eu nunca tinha exercitado o motor de regras com um segundo cliente.

Ainda não posso concluir que sou bem calibrado: seis previsões é amostra pequena.

## O que a discordância entre fontes revelou, e que eu não previ

Eu previ "discordam em pelo menos um ponto de pontuação" imaginando divergência **entre**
fontes. O que apareceu foi pior: divergência **dentro** de uma — o PDF oficial do Jogatina
contradiz a si mesmo sobre quanto vale um truco aceito, no mesmo documento, a três
parágrafos de distância. A previsão acertou pelo motivo errado, e isso conta como acerto
raso. Resolvido por triangulação em `refutacoes/R01`.

## Afirmação que esta v1 não fecha

- A guarda de destino de webhook tem uma janela de *DNS rebinding* (TOCTOU) entre a minha
  resolução e a do `reqwest`. **p = 0,9 de que um atacante com controle de DNS consegue
  atravessá-la.** Não fechei porque fechar exige conectar ao IP já validado e passar o `Host`
  à mão. Está em `riscos_conhecidos`, não na lista de feitos. Uma revisão de segurança
  automática do ambiente apontou exatamente este item de forma independente — o que é
  confirmação, não novidade.
