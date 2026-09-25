# Estudo Estrangeiros – MVP Acadêmico de Engenharia de Dados

## Estrutura do Projeto

`
Estudo estrangeiros/
├── data/
│   ├── bronze/          # Dados brutos provenientes do FBref
│   ├── silver/          # Dados tratados e normalizados
│   └── gold/            # Dados analíticos derivados
├── src/
│   ├── extract/         # Scripts de extração
│   ├── transform/       # Scripts de transformação
│   └── validate/        # Scripts de validação
├── notebooks/           # Análises exploratórias
├── sql/                 # Scripts SQL
└── README.md            # Este arquivo
`

## Camadas de Dados

- **Bronze**: contém dados brutos, preservando o máximo possível a estrutura original da fonte (FBref).
- **Silver**: conterá dados tratados, padronizados e prontos para integração/análise.
- **Gold**: conterá dados derivados e agregados para responder às perguntas de negócio.

## Sobre as Temporadas

- Os dados de **2014–2025** representam temporadas completas.
- **2026** representa a temporada corrente/parcial e deverá ser identificada como tal nas análises.

## Fonte e Coleta dos Dados

Os dados brutos vêm do FBref/Sports Reference (Campeonato Brasileiro Série A, temporadas 2014–2025, com 2026 tratada à parte por estar em andamento).

A coleta foi feita **manualmente**, por meio da exportação nativa de tabelas (CSV/Excel) disponibilizada pelo próprio site, tabela por tabela e temporada por temporada. Essa abordagem foi escolhida em vez de um scraper automatizado porque o FBref adota políticas de limitação de requisições por minuto e bloqueio de acesso automatizado; tentar contornar essas proteções violaria os termos de uso da fonte.

## Atribuição e Licença de Uso

Os dados do FBref/Sports Reference devem ser devidamente atribuídos à fonte original ("please cite us and provide a link and/or a mention", conforme consta nos próprios arquivos exportados).

Os termos de uso do FBref/Sports Reference não autorizam a redistribuição/republicação em massa dos dados coletados. Por isso, os arquivos brutos (`data/`) **não são versionados neste repositório público** — apenas o código (extração/documentação do processo, transformação, SQL, catálogo de dados) é publicado, conforme item 4 do enunciado do MVP ("Não é necessário a disponibilização dos dados utilizados"). Os dados são carregados diretamente em um Volume do Databricks (fora do controle de versão) para alimentar o pipeline.
