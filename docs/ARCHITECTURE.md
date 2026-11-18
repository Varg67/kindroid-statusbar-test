# Arquitetura

Este documento descreve o funcionamento interno do painel **Status Bar** (v0.2.0).

## Visão geral

O painel é um único arquivo HTML autocontido (`src/screen/index.html`) com CSS e JavaScript embutidos. Não há build, framework ou dependências externas. Ele é carregado pelo host Kindroid dentro de um iframe, no *slot* `panel` declarado em `src/kin.json`.

```text
┌─────────────── Host Kindroid ───────────────┐
│                                              │
│   ┌──────── iframe (slot: panel) ────────┐   │
│   │  screen/index.html                    │   │
│   │   ├─ UI (métricas, ações, diagnóstico)│   │
│   │   ├─ Estado em memória                │   │
│   │   ├─ Backup: localStorage             │   │
│   │   └─ Ponte: postMessage ◄────────────►│◄──┼── files.read / files.write
│   └───────────────────────────────────────┘   │        (status.json)
└──────────────────────────────────────────────┘
```

## Manifesto (`kin.json`)

| Campo | Valor | Descrição |
| --- | --- | --- |
| `kin` | `1` | Versão do formato de manifesto. |
| `name` | `Status Bar` | Nome exibido. |
| `purpose` | texto | Finalidade declarada do mini-app. |
| `screens[].slot` | `panel` | Onde a tela é exibida no host. |
| `screens[].entry` | `screen/index.html` | Arquivo de entrada. |

## Constantes principais

| Constante | Valor | Uso |
| --- | --- | --- |
| `PATH` | `status.json` | Arquivo lido/gravado no host. |
| `LOCAL_KEY` | `kindroid.status-bar.demo.v2` | Chave do backup em `localStorage`. |
| `DEFAULTS` | `{energy:74, mood:86, social:48, stress:23}` | Valores iniciais / reset. |
| `DELTA` | `{energy:-4, mood:2, social:-1, stress:3}` | Variação do "tick" de demonstração. |

## Protocolo da ponte

Toda a comunicação usa `window.parent.postMessage`. A origem de destino é derivada de `document.referrer` quando disponível (caso contrário, `'*'`).

### Mensagens enviadas pelo painel

```js
// Leitura
{ kin: 'files.read', path: 'status.json' }

// Escrita (com debounce de 140 ms)
{ kin: 'files.write', path: 'status.json', body: '<JSON serializado>' }
```

### Respostas esperadas do host

```js
{ kin: 'files.read',  path?: 'status.json', ok?: boolean, body?: string | object }
{ kin: 'files.write', path?: 'status.json', ok?: boolean }
```

### Regras de recebimento

Uma mensagem é ignorada se:

1. o painel não estiver em um iframe;
2. `event.source` não for `window.parent`;
3. a origem conhecida não coincidir com `event.origin`;
4. `kin` não for `files.read` nem `files.write`;
5. `path` estiver presente e for diferente de `status.json`.

Para `files.read`, a resposta só é aplicada se houver uma requisição pendente, se `ok !== false`, se o corpo for JSON válido conforme o esquema **e** se o usuário não tiver feito edições desde o carregamento (`isDirty`). Neste último caso, as edições locais prevalecem e são regravadas no host.

## Ciclo de vida

1. **Inicialização** — testa `localStorage`, carrega o backup local se válido e renderiza.
2. **Leitura inicial** — se estiver em um iframe, envia `files.read`.
3. **Alteração** — cada `commit` incrementa `revision`, atualiza `updatedAt`, salva em `localStorage`, re-renderiza, registra no log e agenda `files.write`.
4. **Diagnóstico** — o status do host muda para *Bridge responded* na primeira resposta válida.

## Estados de persistência

| Situação | Indicador exibido |
| --- | --- |
| Host respondeu | `Kindroid files (+ local backup)` |
| Sem host, `localStorage` disponível | `Browser local storage` |
| Sem host e sem `localStorage` | `Session only` |

## Validação do estado

A função `valid()` exige:

- objeto não-array com `schemaVersion === 1`;
- `revision` inteiro seguro ≥ 0;
- `updatedAt` string parseável como data;
- `values` com **exatamente** 4 chaves (`energy`, `mood`, `social`, `stress`), cada uma inteira entre 0 e 100.

O equivalente formal está em [`schemas/status.schema.json`](../schemas/status.schema.json).
