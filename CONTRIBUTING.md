# Guia de contribuição

Obrigado pelo interesse em contribuir com o **Status Bar**!

Ao contribuir, você concorda que sua contribuição será licenciada sob a [licença MIT](LICENSE) do projeto.

## Fluxo de trabalho

1. Crie uma branch a partir da `main` com um prefixo descritivo:
   - `feat/` nova funcionalidade
   - `fix/` correção de bug
   - `docs/` documentação
   - `chore/` manutenção
2. Faça commits pequenos e focados.
3. Atualize o [`CHANGELOG.md`](CHANGELOG.md) na seção **[Não lançado]**.
4. Abra um pull request preenchendo o template.

## Padrão de commits

Usamos [Conventional Commits](https://www.conventionalcommits.org/pt-br/v1.0.0/):

```text
feat(panel): adiciona gráfico de histórico
fix(bridge): ignora respostas com path diferente
docs: atualiza diagrama de sequência
```

## Diretrizes de código

- **Sem dependências externas** e **sem requisições de rede** — é um requisito do projeto.
- Mantenha o painel em um único arquivo (`src/screen/index.html`), salvo decisão registrada em issue.
- Preserve a acessibilidade: rótulos ARIA, foco visível, contraste e `prefers-reduced-motion`.
- Qualquer mudança no formato de `status.json` exige:
  - incrementar `schemaVersion`;
  - atualizar `schemas/status.schema.json` e `docs/ARCHITECTURE.md`;
  - prever a migração do formato anterior.
- Após alterar `src/`, regenere o pacote em `code/` (veja o README).

## Testando localmente

```bash
python3 -m http.server 8080 --directory src
```

Verifique:

- [ ] Modo standalone (abrindo o arquivo direto) funciona e persiste no `localStorage`.
- [ ] Sliders, tick, reset, exportação e recarregamento funcionam.
- [ ] Layout responsivo abaixo de 570 px.
- [ ] Nenhum erro no console.

## Versionamento

Seguimos [SemVer](https://semver.org/lang/pt-BR/). Ao lançar uma versão, atualize o selo `vX.Y.Z` no cabeçalho do painel, o README e o CHANGELOG.
