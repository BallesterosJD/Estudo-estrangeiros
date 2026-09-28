# Decisões Metodológicas

> Registro das decisões de modelagem tomadas durante a construção do pipeline, com o racional de cada uma. Complementa o [catálogo de dados](catalogo_dados.md), que descreve o resultado; aqui está o porquê. Ordem cronológica.

## 1. Metadados de ingestão na camada Bronze

**Decisão**: toda tabela Bronze recebe quatro colunas de controle, além das colunas originais do FBref: `_season` (temporada de origem), `_source_table` (tabela do FBref de onde veio), `_source_file` (caminho do arquivo no Volume) e `_ingested_at` (timestamp da carga).

**Motivação**: garantir rastreabilidade. A partir de qualquer linha das camadas seguintes é possível identificar de qual arquivo e de qual execução de carga ela se originou. As colunas originais são preservadas como texto, sem conversão de tipo — tipagem e interpretação acontecem na Silver.

## 2. Detecção automática da linha de cabeçalho

**Decisão**: em vez de fixar `skiprows=N` por tabela, o pipeline procura dinamicamente a primeira linha 100% preenchida (todas as colunas com valor) como cabeçalho real.

**Motivação**: os exports manuais do FBref trazem uma linha de citação, linhas em branco e uma linha de "categoria" (ex: `,,,Playing Time,,,`) antes do cabeçalho de verdade — e a quantidade dessas linhas variou entre tabelas e entre a extração antiga via script (sem preâmbulo) e a extração manual (com preâmbulo). Detecção automática torna o código robusto a essa variação, sem precisar de tratamento caso a caso por tabela/ano.

## 3. Sanitização de nomes de coluna (não de valores)

**Decisão**: caracteres não aceitos pelo Delta em nome de coluna (espaço, vírgula, `;{}()` etc.) são trocados por `_` — ex: `"# Pl"` → `"#_Pl"`, `"Top Team Scorer"` → `"Top_Team_Scorer"`.

**Motivação**: exigência técnica de armazenamento (Delta não aceita esses caracteres em identificador de coluna), não uma limpeza de conteúdo. Nenhum valor de linha é alterado.

## 4. `player_id` como chave, não o nome do jogador

**Decisão**: a coluna originalmente chamada `-9999` (artefato do export do FBref, sem nome de coluna reconhecível) foi renomeada para `player_id` e passou a ser a chave de identificação do jogador.

**Motivação**: nomes de jogadores têm homônimos (dois "Carlos Alberto" já apareceram nos dados de 2014, times diferentes) e grafias inconsistentes entre temporadas. O `player_id` do FBref é estável e único, evitando join incorreto por nome.

## 5. Grain player-season-club, com dedupe por `(player_id, season, squad)`

**Decisão**: a granularidade da tabela Silver principal (`player_season_club`) é jogador-temporada-clube, não jogador-temporada.

**Motivação**: jogadores transferidos no meio da temporada têm uma linha por clube — agregar isso perderia a informação de quais minutos foram jogados em qual equipe, importante pra qualquer análise por clube (perguntas 4 e 6 do objetivo).

## 6. Faixas etárias: `≤23` / `24-29` / `30+`

**Decisão**: (revisado de `24-28`/`29+` para `24-29`/`30+`).

**Motivação**: `≤23` é o corte usado pela própria CBF na motivação do estudo (sub-23) e pela convenção internacional de elegibilidade sub-23 (ex: Jogos Olímpicos). `30+` é o corte mais reconhecido no meio do futebol pra "veterano" — mais alinhado a estudos de curva etária de desempenho (ex: CIES Football Observatory), que mostram o pico de performance/valor de mercado geralmente entre 25-27, com platô até os 29 e declínio mais consistente só a partir dos 30-31 pra maioria das posições. Qualquer corte exato é uma simplificação de uma curva contínua — este foi escolhido por ter respaldo teórico e ser intuitivo pra quem for ler o trabalho.

## 7. Idade por geração (ano de nascimento), não idade nominal do FBref

**Decisão**: a idade usada para `age_band` é `season - birth_year` (ano da temporada menos ano de nascimento), não a coluna `Age` original do FBref. A idade nominal do FBref foi mantida como `age_fbref`, só para referência/auditoria.

**Motivação**: o `Age` do FBref é calculado numa data de corte fixa dentro da temporada (aparentemente próximo de 1º de fevereiro — evidência: um jogador nascido em 1983 aparece com 30 anos na temporada 2014, quando `2014-1983=31`, indicando que seu aniversário ainda não tinha passado na data de referência do FBref). Isso significa que dois jogadores da mesma geração podem cair em faixas etárias diferentes só por causa do mês de nascimento — um artefato de calendário, não uma diferença real de maturidade/geração. Isso é especialmente crítico no corte `≤23`, que é o eixo central da pergunta de pesquisa do projeto. Usar ano de nascimento (mesmo critério usado nas categorias de base do futebol — "sub-20 2005", etc.) elimina esse ruído: todo jogador nascido no mesmo ano é tratado com a mesma "idade de geração" durante toda a temporada, de forma sistemática e documentada, em vez do critério não documentado/inconsistente do FBref.

**Trade-off assumido**: um jogador nascido em dezembro é tratado como tendo a idade do ano inteiro mesmo antes de fazer aniversário — simplificação deliberada, mas uniforme pra todos os jogadores, ao contrário do ruído gerado pela idade nominal do FBref.

## 8. Posição primária e secundária

**Decisão**: a coluna `Pos` original (`position_raw`) foi separada em `position_primary` e `position_secondary`.

**Motivação**: o FBref concatena códigos de posição de 2 letras sem separador quando o jogador atua em mais de uma posição (ex: `"DFMF"` = DF + MF, não uma posição nova). Mantida como string única, qualquer agregação por posição (`GROUP BY position`) ficaria inconsistente — um volante que também defende nunca seria agrupado corretamente com os volantes puros. A separação foi feita fatiando a string de 2 em 2 caracteres (todos os códigos do FBref têm exatamente 2 letras), funcionando genericamente para 1, 2 ou mais posições. `position_secondary` fica `null` quando o jogador só tem uma posição registrada.
