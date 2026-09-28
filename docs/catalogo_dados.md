# Catálogo de Dados

> Documenta as tabelas das camadas Silver e Gold: nome, tipo, descrição e domínio de cada campo. A camada Bronze não é modelada aqui porque mantém as colunas originais do FBref sem tipagem (tudo como texto bruto, ver `decisoes_metodologicas.md`), com o acréscimo de 4 colunas de controle em toda tabela: `_season` (int, temporada de origem), `_source_table` (string, nome da tabela FBref de origem), `_source_file` (string, caminho do arquivo de origem no Volume), `_ingested_at` (string/timestamp ISO, momento da ingestão).

## Silver

### `estudo_estrangeiros.silver.player_season_club`

Granularidade: **jogador-temporada-clube** (uma linha por jogador, por temporada, por clube em que atuou — jogadores transferidos no meio da temporada têm mais de uma linha).

| Coluna | Tipo | Descrição | Domínio |
|---|---|---|---|
| `player_id` | string | Identificador único do jogador (extraído do FBref) | ID alfanumérico de 8 caracteres |
| `player_name` | string | Nome do jogador conforme FBref | texto livre |
| `nation_raw` | string | Nacionalidade como veio da fonte | ex: `"br BRA"` (código 2 letras + código 3 letras); nulo em casos raros de perfil incompleto na fonte |
| `position_raw` | string | Posição(ões) como veio da fonte, concatenada sem separador quando múltipla | ex: `"MF"`, `"DFMF"`; nulo em casos raros |
| `squad` | string | Clube | nome do clube conforme FBref |
| `age_fbref` | int | Idade nominal do FBref na data de referência da temporada | nulo para a temporada em andamento (formato "anos-dias" não parseado, não usado na análise) |
| `birth_year` | int | Ano de nascimento | ex: 1989 |
| `matches_played` | int | Partidas disputadas na temporada, por aquele clube | ≥ 0 |
| `starts` | int | Partidas como titular | ≥ 0, ≤ `matches_played` |
| `minutes` | int | Minutos jogados na temporada, por aquele clube | 0 a 3420 (38 jogos × 90 min) |
| `goals` | int | Gols marcados | ≥ 0 |
| `assists` | int | Assistências | ≥ 0 |
| `season` | int | Temporada (ano) | 2014-2026 (2026 = parcial, em andamento) |
| `nation_code` | string | Código de nacionalidade de 3 letras, extraído de `nation_raw` | ex: `"BRA"`, `"ARG"` |
| `is_brazilian` | boolean | Flag derivada: `nation_code == "BRA"` | `true`/`false`/`null` (quando `nation_code` é nulo) |
| `position_primary` | string | Primeira posição listada, extraída de `position_raw` | ex: `"GK"`, `"DF"`, `"MF"`, `"FW"` |
| `position_secondary` | string | Segunda posição listada, quando existe | mesmo domínio de `position_primary`; nulo quando o jogador só tem uma posição registrada |
| `age` | int | Idade **por geração** = `season - birth_year` (não é a idade nominal do FBref — ver `decisoes_metodologicas.md`) | tipicamente 15-45 |
| `age_band` | string | Faixa etária derivada de `age` | `"<=23"`, `"24-29"`, `"30+"`, nulo quando `age` é nulo |

### `estudo_estrangeiros.silver.league_table`

Granularidade: **clube-temporada** (classificação final do Brasileirão).

| Coluna | Tipo | Descrição | Domínio |
|---|---|---|---|
| `position` | int | Posição final na tabela | 1-20 |
| `squad` | string | Clube | nome do clube conforme FBref |
| `matches_played` | int | Jogos disputados na temporada | tipicamente 38 |
| `wins` / `draws` / `losses` | int | Vitórias / empates / derrotas | ≥ 0, soma = `matches_played` |
| `goals_for` / `goals_against` | int | Gols marcados / sofridos | ≥ 0 |
| `goal_diff` | int | Saldo de gols | `goals_for - goals_against` |
| `points` | int | Pontos na temporada | ≥ 0 |
| `points_per_match` | double | Pontos por jogo | `points / matches_played` |
| `season` | int | Temporada | 2014-2026 |

## Gold

### `estudo_estrangeiros.gold.age_nationality_minutes`

Responde à pergunta 3 (matriz nacionalidade × faixa etária). Granularidade: **temporada × nacionalidade (BRA/estrangeiro) × faixa etária**.

| Coluna | Tipo | Descrição |
|---|---|---|
| `season` | int | Temporada |
| `is_brazilian` | boolean | Brasileiro (`true`) ou estrangeiro (`false`) |
| `age_band` | string | `"<=23"` / `"24-29"` / `"30+"` |
| `total_minutes` | long | Soma de minutos jogados no grupo |
| `players` | long | Nº de jogadores distintos no grupo |
| `pct_of_season_minutes` | double | % do total de minutos da temporada que esse grupo representa (soma de todos os grupos de uma temporada = ~100%) |

### `estudo_estrangeiros.gold.foreign_players`

Responde às perguntas 1 e 5 (participação e produção ofensiva por nacionalidade). Granularidade: **temporada × nacionalidade (código de 3 letras)**.

| Coluna | Tipo | Descrição |
|---|---|---|
| `season` | int | Temporada |
| `nation_code` | string | Código de nacionalidade (ex: `"BRA"`, `"ARG"`) |
| `is_brazilian` | boolean | Flag de conveniência |
| `players` | long | Nº de jogadores distintos dessa nacionalidade na temporada |
| `total_minutes` | long | Minutos totais jogados |
| `total_goals` / `total_assists` | long | Gols / assistências totais |
| `pct_of_season_minutes` | double | % do total de minutos da temporada |
| `pct_of_season_goals` / `pct_of_season_assists` | double | % do total de gols / assistências da temporada |
| `goals_assists_per90` | double | (gols + assistências) a cada 90 minutos jogados — métrica de eficiência, não de volume. **Cautela**: instável para nacionalidades com poucos jogadores/minutos (ver `decisoes_metodologicas.md`) |

### `estudo_estrangeiros.gold.club_performance`

Responde à pergunta 4 (relação entre uso de estrangeiros/jovens e desempenho do clube). Granularidade: **clube × temporada**.

| Coluna | Tipo | Descrição |
|---|---|---|
| `season` | int | Temporada |
| `squad` | string | Clube |
| `position` | int | Posição final na tabela |
| `points` / `points_per_match` | int / double | Desempenho em pontos |
| `goal_diff` | int | Saldo de gols |
| `pct_minutes_foreign` | double | % dos minutos do clube na temporada jogados por estrangeiros |
| `pct_minutes_brazilian_u23` | double | % dos minutos do clube na temporada jogados por brasileiros ≤23 anos |
