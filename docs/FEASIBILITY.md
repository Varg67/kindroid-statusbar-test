# Análise de viabilidade

Avaliação da [visão](VISION.md) de motor de simulação de vida do Kin frente ao que a plataforma do **Apps Workshop do Kindroid (alpha)** comprovadamente oferece.

> **Status:** rascunho. A visão ainda está sendo amadurecida; este documento registra o que se sabe, o que falta confirmar e as decisões em aberto.

## 1. Capacidades da plataforma

A única fonte confirmada é o painel atual ([`src/screen/index.html`](../src/screen/index.html)). Não há documentação pública da ponte além do que ele usa.

| Capacidade exigida | Situação | Alternativa se indisponível |
| --- | --- | --- |
| Armazenamento persistente | ✅ Confirmado: `files.read` / `files.write` | — |
| Passagem de tempo com o app fechado | ✅ Viável sem execução em segundo plano: ao abrir, calcula o tempo decorrido e simula em passos de 15–30 min, de forma determinística | — |
| Ler mensagens do chat (tags de delta) | ❓ Não confirmado | Registro de eventos no painel por botões rápidos ("comeu", "brigaram", "presente"), no estilo visual novel |
| Escrever no contexto / diário / memória do Kin | ❓ Não confirmado, e é a peça central | Painel gera o resumo qualitativo com botão "copiar", para colar manualmente num campo do Kin |
| Ler a agenda nativa do Kin | ❓ Não confirmado | Rotina simples configurada no próprio app |
| Execução agendada | ❓ Não confirmado | Desnecessária: o cálculo na abertura cobre o caso |
| Requisições de rede / chamadas a LLM pelo app | ❌ Fora do escopo (o projeto não faz requisições externas) | — |

**Regra:** nenhum nome de mensagem da ponte será inventado. Capacidades ❓ só serão usadas após confirmação por documentação ou teste com o host.

## 2. Pontos de design a decidir

1. **Escopo vs. orçamento de contexto.** A visão tem mais de 60 variáveis, mas o resumo injetado tem poucas centenas de tokens. Proposta: o resumo prioriza o que altera comportamento visível (humor, energia, estados ativos, tensão, saudade, pendências); o restante vive só no motor.
2. **Confiabilidade das tags de delta.** O LLM vai omitir tags, inventar campos e errar a sintaxe. A validação da seção 9 da visão cobre isso, mas os botões manuais devem existir como caminho principal ou reserva. Tags visíveis na mensagem do Kin quebram a imersão.
3. **Dono do "arco de crise".** Quem ativa a flag que remove os pisos de segurança? Proposta: apenas o usuário, nunca o LLM.
4. **Migração de esquema.** O `status.json` v1 tem `energy`, `mood`, `social` e `stress`. Na visão, energia e humor passam a ser derivados. Será necessário `schemaVersion: 2` com migração do v1.
5. **Determinismo.** Eventos aleatórios na simulação offline devem usar PRNG com semente (ID do Kin + índice do passo), para que reabrir o app produza sempre o mesmo resultado.

## 3. Conferência dos valores iniciais

Os valores da visão são coerentes entre si e podem virar a configuração padrão:

| Verificação | Cálculo | Resultado |
| --- | --- | --- |
| Pressão de sono zera com 8h de sono bom | 16h acordado × 6 = 96; 8h × −12 = −96 | ✅ |
| Fome "com fome" em ~6h, crítica em ~12h | 6 × 8 = 48; 12,5 × 8 = 100 | ✅ |
| Sede 50 em ~4h | 4 × 12 = 48 | ✅ |
| Fadiga mental de um dia de trabalho se recupera dormindo | 8 × 10 = 80; 8h × −10 = −80 | ✅ |

## 4. Plano de fases

| Fase | Entrega | Depende de capacidades ❓? |
| --- | --- | --- |
| 0 | Visão e viabilidade documentadas | Não |
| 1 | Motor de simulação em JS puro com testes: tempo real, avanço offline, camada de corpo e energia derivada | Não |
| 2 | Camada de mente, humor derivado e resumo qualitativo com botão "copiar" | Não |
| 3 | Registro manual de eventos e protocolo de deltas com validação e limites | Não (manual); sim (automático) |
| 4 | Camada de relacionamento com o usuário (vínculo e gates, confiança, tensão, atração, saudade) | Não |
| 5 | Integração real com o host (contexto, chat, agenda) | **Sim** |
| 6 | Módulos opcionais e NPCs | Não |

## 5. Perguntas em aberto

- Existe documentação, exemplo ou canal do Apps Workshop alpha que liste as mensagens aceitas pela ponte além de `files.*`?
- Um único app deve gerenciar vários Kins, ou cada Kin tem sua instância? (O `status.json` atual é por programa.)
- Quais variáveis da visão são essenciais para o MVP e quais podem esperar?
