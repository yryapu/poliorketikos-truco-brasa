# ADR-008 — Playwright, dentro de container, e por quê

**Data:** 2026-10-01 · **Estado:** VIVO

O enunciado deixa a escolha: "use Playwright ou o que você julgar melhor, e diga por que
escolheu".

## Decisão

**Playwright**, rodando **dentro de container**, contra o servidor também em container.

## Por que Playwright, e não as alternativas

O que precisa ser provado no front é específico: que **dois navegadores independentes** se
encontram numa mesa e que o que um clica aparece no outro. Isso elimina quase tudo:

- **jsdom / happy-dom + Vitest.** Não têm WebSocket de verdade nem layout; eu estaria testando
  o meu próprio mock do servidor. Para um cliente cuja função é *desenhar o que o servidor
  manda*, testar contra mock prova zero.
- **Selenium.** Faz o trabalho, mas precisa de WebDriver por navegador e a espera é por
  tempo, não por condição — o que num jogo com mensagens assíncronas significa teste
  intermitente, que é pior que teste nenhum porque ensina a ignorar vermelho.
- **Cypress.** Boa ferramenta, mas o modelo de um navegador por execução torna "dois
  jogadores na mesma mesa" desconfortável (precisa de duas execuções coordenadas).
  Playwright tem `browser.newContext()`, e **dois contextos têm cookies separados** — que é
  exatamente a forma do nosso problema: duas sessões, dois jogadores.
- **curl / teste em Rust só.** Já existe e prova o protocolo (sete testes de integração).
  Não prova que a tela desenha a carta, que o clique manda `indice` certo, nem que o botão
  de truco aparece só quando `acoes` permite.

Playwright ganha por **três coisas concretas**: contextos isolados (duas sessões no mesmo
processo), espera por condição (`expect(...).toBeVisible()` com nova tentativa automática,
em vez de `sleep`), e a captura de falha com vídeo e rastro, que é o que faz um teste de
interface ser diagnosticável em vez de só vermelho.

## Por que em container, e não nesta máquina

Porque **o `node` desta máquina está quebrado**:

```
$ node --version
dyld[90772]: Library not loaded: /opt/homebrew/opt/llhttp/lib/libllhttp.9.3.dylib
  Reason: tried: '.../libllhttp.9.3.dylib' (no such file) …
```

Um pacote do Homebrew ficou com a ligação pendurada. Eu poderia consertar com um comando —
e não vou: mexer na instalação de ferramentas da máquina de quem avalia é efeito colateral
que ninguém pediu, e um teste que só roda depois de eu consertar o ambiente alheio não é um
teste reproduzível. O container resolve o problema **e** a reprodutibilidade de uma vez: a
imagem oficial `mcr.microsoft.com/playwright` já traz os navegadores e as bibliotecas de
sistema deles, que é justamente a parte que falha em máquina limpa.

## O custo central que esta decisão aceita

O teste de front **não roda sem Docker**. `cargo test` continua sendo a prova que roda em
qualquer lugar; o front exige `docker compose -f compose.teste.yaml up`. Está dito no README.

## Cai se

O avaliador não tiver Docker. Nesse caso o conserto é rodar `npx playwright test` apontando
`BASE=http://127.0.0.1:8080` para um servidor local — o arquivo de teste é o mesmo, porque
ele lê a base do ambiente e não tem nada de container dentro.
