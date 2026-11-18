# Política de segurança

## Versões suportadas

| Versão | Suporte |
| --- | --- |
| 0.2.x | ✅ |
| < 0.2 | ❌ |

## Reportando uma vulnerabilidade

**Não abra issues públicas para problemas de segurança.**

Use o recurso [Private vulnerability reporting](https://github.com/Varg67/kindroid-statusbar-test/security/advisories/new) do GitHub, informando:

- descrição do problema e impacto;
- passos para reproduzir;
- versão afetada.

Você receberá uma resposta assim que possível.

## Escopo

São especialmente relevantes:

- falhas na filtragem de mensagens `postMessage` (origem/fonte);
- aceitação de estado que viole o esquema de `status.json`;
- qualquer forma de requisição de rede ou vazamento de dados.
