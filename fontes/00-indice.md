# Fontes — como cada uma foi obtida, quando, e o que ela decide

Tudo nesta pasta foi obtido em **2026-10-01** (fuso da máquina, `America/Sao_Paulo`),
pela sessão `sessao-truco-brasa`. Cada fonte está salva **íntegra** no repositório, não
parafraseada: quem auditar não depende de o site continuar no ar nem da minha leitura dela.

| id | o que é | como foi obtida | sha256 (dos 16 primeiros) |
|----|---------|-----------------|---------------------------|
| F1 | `Regras de Truco Paulista`, PDF oficial do Jogatina.com | `WebFetch` do PDF em `https://s3.amazonaws.com/static.jogatina.com/downloads/truco-paulista/regras-truco-paulista.pdf`, achado por busca; texto extraído com `pdftotext -layout` | `6b9b0cb44deff2b5` (txt) / `44ce843afd545c09` (pdf) |
| F2 | `Regras do Truco Paulista`, regulamento completo hospedado em servidor da UFSC | `curl` de `https://casin.paginas.ufsc.br/files/2010/10/Regras_Truco_Paulista.txt`, convertido de ISO-8859-1 para UTF-8 com `iconv` | `57870f55c4ac7c45` |
| F3 | `Regras Oficiais do Truco Paulista`, MegaJogos | `WebFetch` de `https://www.megajogos.com.br/truco-online/regras`; trechos citados em `regras/truco-paulista.md`, página não salva íntegra (HTML dinâmico) | — |
| F4 | Verbete `Truco`, Wikipédia em português | `WebFetch` de `https://pt.wikipedia.org/wiki/Truco`; trechos citados | — |

Comandos exatos, para reexecução:

```sh
curl -sS -A "Mozilla/5.0" https://s3.amazonaws.com/static.jogatina.com/downloads/truco-paulista/regras-truco-paulista.pdf -o f1.pdf
pdftotext -layout f1.pdf f1.txt
curl -sS -A "Mozilla/5.0" https://casin.paginas.ufsc.br/files/2010/10/Regras_Truco_Paulista.txt | iconv -f ISO-8859-1 -t UTF-8 > f2.txt
```

## Hierarquia de autoridade que eu adotei, e por quê

Quando duas fontes discordam, **F2 vence**, e depois F1. Motivo: F2 é o único documento que
se comporta como *regulamento* — enumera os casos de borda (empate em cada rodada, quem puxa
depois de um empate, quem pode retrucar, carta de costas, mão de ferro aberta/fechada) e é
internamente consistente. F1 é oficial de uma plataforma grande e cobre o mesmo núcleo, mas é
material de divulgação e **contradiz a si mesmo** na pontuação do truco (ver
`refutacoes/R01-jogatina-truco-vale-dois.md`). F3 e F4 entram como testemunhas de
triangulação: servem para confirmar, não para decidir sozinhas.

Onde **nenhuma** fonte fala — notadamente o jogo **1x1** — a regra é minha decisão declarada,
marcada `[DECISÃO]` em `regras/truco-paulista.md`, nunca apresentada como regra encontrada.
