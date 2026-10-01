# Diário — o caminho, incluindo os erros

Uma sessão, 2026-10-01, da primeira fonte ao `resultado.json`: **55 minutos** (medido da
primeira previsão registrada, `07:26:25Z`, por `date -u`).

## A ordem em que as coisas aconteceram

1. **Consultei o corpus antes de decidir desenho** e não havia nada: `reino_consultar` sobre
   jogo de cartas, servidor, sessão, aposta devolveu só páginas de governança, nenhuma de
   domínio; `memoria_consultar` devolveu zero. Zero é dado — significa que nada estava
   pré-decidido, não que eu não perguntei.
2. **Pesquisa antes de código.** Quatro fontes, salvas íntegras. A especificação normativa
   (`regras/truco-paulista.md`) nasceu antes da primeira linha de Rust, com 31 etiquetas.
3. **Repo de pesquisa criado primeiro**, como o enunciado manda, e o primeiro commit foi das
   fontes — não de código.
4. Motor de regras → protocolo fixado → **servidor e cliente em paralelo** → testes de
   integração → Docker → teste de interface → `verbum-pronto` → `resultado.json`.

## Os meus erros, e como achei cada um

| erro | como apareceu | o que mudou por causa dele |
|------|---------------|----------------------------|
| Li a mão de onze como evento único, e ela é **estado permanente** | um teste meu de outra regra (R-25) falhou | o tipo da mão passou a ser derivado do placar a cada distribuição. Como evento, o truco voltaria a ser permitido com a dupla a um ponto de ganhar — bug que só aparece em placar alto (`refutacoes/R02`) |
| Mandei a manilha no protocolo **como carta** de paus | o Playwright falhou com "a tela da bia mostrou 🃔, que é da ana" | a manilha passou a ir como número, e o invariante de isolamento foi reescrito de lista negra para **lista branca** (`refutacoes/R03`) |
| O `/api/ranking` público devolvia o **id interno** de cada jogador | eu olhei a saída de um `curl` no teste de fumaça | tipo próprio para a linha do ranking, sem id |
| A guarda de saldo no `UPDATE` não estava testada | apaguei `AND moedas >= ?2` e **nenhum teste ficou vermelho** | teste de oito conexões simultâneas apostando tudo; sem a guarda, o saldo vai a −5000 |
| Não tinha resposta para "e se o jogador simplesmente **parar**?" | a pergunta da skill `verbum-pronto` | relógio de 60 s por ação; a mesa liquida em vez de prender a aposta do adversário |
| Apliquei uma mutação errada e quase concluí que R-18 não estava testada | o `cargo fmt` tinha quebrado a linha e o meu `replace` não casou em silêncio | refiz; R-18 está testada (mutada de verdade, dois testes vermelhos). **Lição: mutação que não derruba nada pode ser mutação que não foi aplicada** |
| Escrevi "54 testes Rust" numa mensagem de commit | derivei o número depois | são 50 em Rust, 54 no total com os de interface. Corrigido no commit seguinte, com o comando que conta |
| O meu teste de front usava `exemplo.invalid` | o teste falhou | domínio que não resolve é legitimamente recusado pela guarda de destino. O teste estava errado, o código estava certo |

Sete dos oito foram achados por **uma ferramenta, não por leitura**: teste que falhou,
mutação, `curl`, ou pergunta de checklist. Um foi achado por um leitor independente — ver
abaixo.

## O subagente, medido

Um só, para o cliente web. **63.055 tokens, 89,5 s, 3 chamadas de ferramenta, 469 linhas.**

Rendeu duas coisas, e a segunda eu não esperava:

1. O cliente saiu funcionando. Os quatro testes de Playwright passam contra ele, e eu mudei
   **duas linhas** suas em toda a sessão — e por causa de um defeito meu de protocolo, não
   dele.
2. **Ele achou uma contradição no meu próprio `PROTOCOLO.md`**: a tabela dizia
   `valor_proposto` 3 = TRUCO, e o meu exemplo narrava "pediu SEIS" para `valor_proposto: 3`.
   Ele escolheu a tabela — que era a certa — e **disse que havia escolhido**, em vez de
   escolher em silêncio. Um leitor independente do contrato pegou o que o autor do contrato
   não pegou.

Valeu. E o que eu faria diferente: teria pedido um `node --check` no JS. Ele avisou com
honestidade que o script nunca foi sequer checado por sintaxe, e eu só soube que estava
íntegro quando o Playwright passou, meia hora depois.

**O que eu não deleguei, de propósito:** o motor de regras. É a parte cuja correção invalida
todo o resto, e delegar significaria não poder responder por que cada regra cita a fonte que
cita.

## O que eu diria a quem for continuar

Leia `resultado.json` começando pelos **riscos conhecidos**, não pelos critérios. A lista de
feitos é a parte fácil de escrever; os dez riscos são onde o trabalho real está, e o décimo
deles é que ninguém além de mim leu nada disto.
