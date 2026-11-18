# Changelog

Todas as mudanças relevantes deste projeto são documentadas aqui.

O formato segue o [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/) e o projeto adota o [Versionamento Semântico](https://semver.org/lang/pt-BR/).

## [Não lançado]

### Adicionado
- Licença MIT: o projeto passa a ser open source.
- Código-fonte extraído para `src/` para facilitar leitura e revisão.
- Documentação: README, arquitetura, guia de contribuição, política de segurança.
- JSON Schema do estado persistido (`schemas/status.schema.json`).
- Templates de issue e pull request, `.editorconfig` e `.gitignore`.

## [0.2.0] - 2026-11-17

### Adicionado
- Adaptação ao formato de painel do Kindroid (`kin.json`, slot `panel`).
- Ponte de arquivos via `postMessage` (`files.read` / `files.write`) para `status.json`.
- Painel de diagnóstico (interface, host, persistência, dados do Kin).
- Log de atividades, exportação JSON e recarregamento do estado salvo.

### Alterado
- Modelo de dados mantido idêntico ao da v0.1.0 (`schemaVersion: 1`).

## [0.1.0]

### Adicionado
- Protótipo inicial com as métricas Energia, Humor, Social e Estresse.

[Não lançado]: https://github.com/Varg67/kindroid-statusbar-test/compare/main...HEAD
[0.2.0]: https://github.com/Varg67/kindroid-statusbar-test/commits/main
