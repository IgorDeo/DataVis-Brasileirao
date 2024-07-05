---
title: Técnicos
toc: false
sql:
    jogos: ./data/jogos.csv
---

```js
    import * as vega from "npm:vega";
    import * as vegaLite from "npm:vega-lite";
    import * as vegaLiteApi from "npm:vega-lite-api";

    const vl = vegaLiteApi.register(vega, vegaLite);
```


# Técnico
Aproveitamento é a quantidade de pontos ganhos divididos pela quantidade de pontos disputados.
1. **Para cada técnico, qual time teve maior aproveitamento?**
    - Justificativa: Avaliar o desempenho dos técnicos em diferentes times pode ajudar a identificar quais técnicos obtêm melhores resultados e em quais contextos eles se destacam.

    ```sql id=perguntaUm display
            WITH vitorias AS (
        SELECT 
            CASE 
                WHEN gols_mandante > gols_visitante THEN time_mandante
                WHEN gols_visitante > gols_mandante THEN time_visitante 
            END AS time,
            CASE 
                WHEN gols_mandante > gols_visitante THEN tecnico_mandante
                WHEN gols_visitante > gols_mandante THEN tecnico_visitante 
            END as tecnico_vencedor,
            COUNT(*) AS qtd_vitorias
        FROM jogos
        WHERE gols_mandante <> gols_visitante
        GROUP BY 
            CASE 
                WHEN gols_mandante > gols_visitante THEN time_mandante
                WHEN gols_visitante > gols_mandante THEN time_visitante 
            END,
            CASE 
                WHEN gols_mandante > gols_visitante THEN tecnico_mandante
                WHEN gols_visitante > gols_mandante THEN tecnico_visitante 
            END
    ),
    derrotas AS (
        SELECT 
            CASE 
                WHEN gols_mandante > gols_visitante THEN time_visitante
                WHEN gols_visitante > gols_mandante THEN time_mandante 
            END AS time_perdedor,
            CASE 
                WHEN gols_mandante > gols_visitante THEN tecnico_visitante
                WHEN gols_visitante > gols_mandante THEN tecnico_mandante 
            END as tecnico_perdedor,
            COUNT(*) AS qtd_derrotas
        FROM jogos
        WHERE gols_mandante <> gols_visitante
        GROUP BY 
            CASE 
                WHEN gols_mandante > gols_visitante THEN time_visitante
                WHEN gols_visitante > gols_mandante THEN time_mandante 
            END,
            CASE 
                WHEN gols_mandante > gols_visitante THEN tecnico_visitante
                WHEN gols_visitante > gols_mandante THEN tecnico_mandante 
            END
    ),
    empates_mandante AS (
        SELECT 
            tecnico_mandante AS tecnico,
            time_mandante AS time,
            COUNT(*) AS qtd_empates
        FROM jogos
        WHERE gols_mandante = gols_visitante
        GROUP BY tecnico_mandante, time_mandante
    ),
    empates_visitante AS (
        SELECT 
            tecnico_visitante AS tecnico,
            time_visitante AS time,
            COUNT(*) AS qtd_empates
        FROM jogos
        WHERE gols_mandante = gols_visitante
        GROUP BY tecnico_visitante, time_visitante
    ),
    empates AS (
        SELECT tecnico, time, SUM(qtd_empates) AS qtd_empates
        FROM (
            SELECT * FROM empates_mandante
            UNION ALL
            SELECT * FROM empates_visitante
        ) combined
        GROUP BY tecnico, time
    )
    SELECT 
        vitorias.tecnico_vencedor AS tecnico, 
        vitorias.time, 
        qtd_vitorias, 
        COALESCE(qtd_derrotas, 0) AS qtd_derrotas, 
        COALESCE(qtd_empates, 0) AS qtd_empates,
        (qtd_vitorias + COALESCE(qtd_derrotas, 0) + COALESCE(qtd_empates, 0)) AS qtd_partidas,
        100 * (qtd_vitorias / qtd_partidas) as aproveitamento
    FROM vitorias
    LEFT JOIN derrotas ON vitorias.tecnico_vencedor = derrotas.tecnico_perdedor 
                        AND vitorias.time = derrotas.time_perdedor
    LEFT JOIN empates ON vitorias.tecnico_vencedor = empates.tecnico 
                        AND vitorias.time = empates.time
    where vitorias.tecnico_vencedor is not null and vitorias.time is not null
    order by vitorias.time



    ```

2. **Quais técnicos têm a maior média de pontos por jogo ao longo de suas carreiras?**
    - Justificativa: Determinar a média de pontos por jogo pode ajudar a identificar técnicos consistentemente bem-sucedidos e compreender seus métodos de trabalho.

    ```sql id=pergunta2 display
            WITH vitorias AS (
            SELECT 
                ano_campeonato,
                CASE 
                    WHEN gols_mandante > gols_visitante THEN time_mandante
                    WHEN gols_visitante > gols_mandante THEN time_visitante 
                END AS time,
                CASE 
                    WHEN gols_mandante > gols_visitante THEN tecnico_mandante
                    WHEN gols_visitante > gols_mandante THEN tecnico_visitante 
                END as tecnico_vencedor,
                COUNT(*) AS qtd_vitorias
            FROM jogos
            WHERE gols_mandante <> gols_visitante
            GROUP BY 
                ano_campeonato,
                CASE 
                    WHEN gols_mandante > gols_visitante THEN time_mandante
                    WHEN gols_visitante > gols_mandante THEN time_visitante 
                END,
                CASE 
                    WHEN gols_mandante > gols_visitante THEN tecnico_mandante
                    WHEN gols_visitante > gols_mandante THEN tecnico_visitante 
                END
        ),
        derrotas AS (
            SELECT 
                ano_campeonato,
                CASE 
                    WHEN gols_mandante > gols_visitante THEN time_visitante
                    WHEN gols_visitante > gols_mandante THEN time_mandante 
                END AS time_perdedor,
                CASE 
                    WHEN gols_mandante > gols_visitante THEN tecnico_visitante
                    WHEN gols_visitante > gols_mandante THEN tecnico_mandante 
                END as tecnico_perdedor,
                COUNT(*) AS qtd_derrotas
            FROM jogos
            WHERE gols_mandante <> gols_visitante
            GROUP BY 
                ano_campeonato,
                CASE 
                    WHEN gols_mandante > gols_visitante THEN time_visitante
                    WHEN gols_visitante > gols_mandante THEN time_mandante 
                END,
                CASE 
                    WHEN gols_mandante > gols_visitante THEN tecnico_visitante
                    WHEN gols_visitante > gols_mandante THEN tecnico_mandante 
                END
        ),
        empates_mandante AS (
            SELECT 
                ano_campeonato,
                tecnico_mandante AS tecnico,
                time_mandante AS time,
                COUNT(*) AS qtd_empates
            FROM jogos
            WHERE gols_mandante = gols_visitante
            GROUP BY ano_campeonato, tecnico_mandante, time_mandante
        ),
        empates_visitante AS (
            SELECT 
                ano_campeonato,
                tecnico_visitante AS tecnico,
                time_visitante AS time,
                COUNT(*) AS qtd_empates
            FROM jogos
            WHERE gols_mandante = gols_visitante
            GROUP BY ano_campeonato, tecnico_visitante, time_visitante
        ),
        empates AS (
            SELECT ano_campeonato, tecnico, time, SUM(qtd_empates) AS qtd_empates
            FROM (
                SELECT * FROM empates_mandante
                UNION ALL
                SELECT * FROM empates_visitante
            ) combined
            GROUP BY ano_campeonato, tecnico, time
        ),
        resultados AS (
            SELECT 
                vitorias.ano_campeonato,
                vitorias.tecnico_vencedor AS tecnico, 
                vitorias.time, 
                qtd_vitorias, 
                COALESCE(qtd_derrotas, 0) AS qtd_derrotas, 
                COALESCE(qtd_empates, 0) AS qtd_empates,
                (qtd_vitorias + COALESCE(qtd_derrotas, 0) + COALESCE(qtd_empates, 0)) AS qtd_partidas,
                (qtd_vitorias * 3 + COALESCE(qtd_empates, 0)) AS pontos
            FROM vitorias
            LEFT JOIN derrotas ON vitorias.tecnico_vencedor = derrotas.tecnico_perdedor 
                                AND vitorias.time = derrotas.time_perdedor
                                AND vitorias.ano_campeonato = derrotas.ano_campeonato
            LEFT JOIN empates ON vitorias.tecnico_vencedor = empates.tecnico 
                                AND vitorias.time = empates.time
                                AND vitorias.ano_campeonato = empates.ano_campeonato
        )
    
        SELECT 
            ano_campeonato,
            tecnico, 
            SUM(pontos) AS total_pontos, 
            SUM(qtd_partidas) AS total_partidas, 
            (SUM(pontos) / SUM(qtd_partidas)) AS media_pontos_jogo
        FROM resultados
        where tecnico is not null
        GROUP BY ano_campeonato, tecnico
        ORDER BY ano_campeonato desc, media_pontos_jogo DESC
        

    ```

3. **Para cada técnico, qual foi o desempenho do time em jogos fora de casa?**
    - Justificativa: Verificar o desempenho dos times sob a liderança de diferentes técnicos em jogos fora de casa pode revelar quais técnicos conseguem motivar e preparar melhor suas equipes para enfrentar desafios longe de casa.

    ```sql id=perguntaTres display
        WITH vitorias_fora AS (
            SELECT 
                tecnico_visitante AS tecnico,
                time_visitante AS time,
                COUNT(*) AS qtd_vitorias
            FROM jogos
            WHERE gols_visitante > gols_mandante AND tecnico_visitante IS NOT NULL
            GROUP BY tecnico_visitante, time_visitante
        ),
        derrotas_fora AS (
            SELECT 
                tecnico_visitante AS tecnico,
                time_visitante AS time,
                COUNT(*) AS qtd_derrotas
            FROM jogos
            WHERE gols_visitante < gols_mandante AND tecnico_visitante IS NOT NULL
            GROUP BY tecnico_visitante, time_visitante
        ),
        empates_fora AS (
            SELECT 
                tecnico_visitante AS tecnico,
                time_visitante AS time,
                COUNT(*) AS qtd_empates
            FROM jogos
            WHERE gols_visitante = gols_mandante AND tecnico_visitante IS NOT NULL
            GROUP BY tecnico_visitante, time_visitante
        ),
        resultados_fora AS (
            SELECT 
                COALESCE(vitorias_fora.tecnico, derrotas_fora.tecnico, empates_fora.tecnico) AS tecnico,
                COALESCE(vitorias_fora.time, derrotas_fora.time, empates_fora.time) AS time,
                COALESCE(vitorias_fora.qtd_vitorias, 0) AS qtd_vitorias,
                COALESCE(derrotas_fora.qtd_derrotas, 0) AS qtd_derrotas,
                COALESCE(empates_fora.qtd_empates, 0) AS qtd_empates
            FROM vitorias_fora
            FULL OUTER JOIN derrotas_fora ON vitorias_fora.tecnico = derrotas_fora.tecnico AND vitorias_fora.time = derrotas_fora.time
            FULL OUTER JOIN empates_fora ON vitorias_fora.tecnico = empates_fora.tecnico AND vitorias_fora.time = empates_fora.time
        ),
        resultados_aproveitamento AS (
            SELECT 
                tecnico, 
                time,
                (qtd_vitorias + qtd_derrotas + qtd_empates) AS total_jogos,
                100 * (qtd_vitorias::float / (qtd_vitorias + qtd_derrotas + qtd_empates)) AS aproveitamento
            FROM resultados_fora
        ),
        media_total_jogos AS (
            SELECT AVG(total_jogos) AS avg_jogos FROM resultados_aproveitamento
        ),
        max_aproveitamento_tecnico AS (
            SELECT 
                tecnico,
                MAX(aproveitamento) AS max_aproveitamento
            FROM resultados_aproveitamento
            GROUP BY tecnico
        ),
        tecnico_time_max_aproveitamento AS (
            SELECT 
                r.tecnico,
                STRING_AGG(r.time, ', ') AS times,
                sum(r.total_jogos) as total_jogos,
                r.aproveitamento
            FROM resultados_aproveitamento r
            JOIN max_aproveitamento_tecnico m ON r.tecnico = m.tecnico AND r.aproveitamento = m.max_aproveitamento
            GROUP BY r.tecnico, r.aproveitamento
        )
        SELECT 
            t.tecnico,
            t.times,
            t.total_jogos,
            t.aproveitamento
        FROM tecnico_time_max_aproveitamento t
        JOIN media_total_jogos m ON t.total_jogos >= m.avg_jogos
        ORDER BY t.aproveitamento desc , t.tecnico  DESC
        LIMIT 20

    ```


    ```js
    const perguntaTresGrafico = {
        spec: {
        "description": "Aproveitamento dos técnicos em jogos fora de casa.",
        "data": { "values": perguntaTres },
        "mark": "bar",
        "encoding": {
            "x": {
            "field": "aproveitamento", 
            "type": "quantitative", 
            "title": "Aproveitamento (%)"
            },
            "y": {
            "field": "tecnico", 
            "type": "nominal", 
            "title": "Técnico",
            "sort": "-x"
            },
            "color": {
            "field": "times", 
            "type": "nominal", 
            "title": "Time"
            },
            "tooltip": [
            {"field": "tecnico", "type": "nominal", "title": "Técnico"},
            {"field": "times", "type": "nominal", "title": "Time"},
            {"field": "total_jogos", "type": "quantitative", "title": "Total de Jogos"},
            {"field": "aproveitamento", "type": "quantitative", "title": "Aproveitamento (%)"}
            ]
        },
        "width": 600,
        "height": 400
        }
    }
    ```

    <div class="card">
    <h1>Técnicos e time com maior aproveitamento em partidas como visitante</h1>
    ${vl.render(perguntaTresGrafico)}
    </div>

4. **Quais técnicos tiveram a maior variação de desempenho entre diferentes temporadas?**
    - Justificativa: Analisar a variação de desempenho dos técnicos ao longo das temporadas pode ajudar a entender sua capacidade de adaptação e resposta a diferentes condições e contextos de campeonato.

    ```sql id=tecnicosComMaisTempoDeAtuacaoDoQue10Anos
            WITH vitorias AS (
            SELECT 
                ano_campeonato,
                tecnico_mandante AS tecnico,
                time_mandante AS time,
                COUNT(*) AS qtd_vitorias
            FROM jogos
            WHERE gols_mandante > gols_visitante
            GROUP BY ano_campeonato, tecnico_mandante, time_mandante
            UNION ALL
            SELECT 
                ano_campeonato,
                tecnico_visitante AS tecnico,
                time_visitante AS time,
                COUNT(*) AS qtd_vitorias
            FROM jogos
            WHERE gols_visitante > gols_mandante
            GROUP BY ano_campeonato, tecnico_visitante, time_visitante
        ),
        derrotas AS (
            SELECT 
                ano_campeonato,
                tecnico_mandante AS tecnico,
                time_mandante AS time,
                COUNT(*) AS qtd_derrotas
            FROM jogos
            WHERE gols_mandante < gols_visitante
            GROUP BY ano_campeonato, tecnico_mandante, time_mandante
            UNION ALL
            SELECT 
                ano_campeonato,
                tecnico_visitante AS tecnico,
                time_visitante AS time,
                COUNT(*) AS qtd_derrotas
            FROM jogos
            WHERE gols_visitante < gols_mandante
            GROUP BY ano_campeonato, tecnico_visitante, time_visitante
        ),
        empates AS (
            SELECT 
                ano_campeonato,
                tecnico_mandante AS tecnico,
                time_mandante AS time,
                COUNT(*) AS qtd_empates
            FROM jogos
            WHERE gols_mandante = gols_visitante
            GROUP BY ano_campeonato, tecnico_mandante, time_mandante
            UNION ALL
            SELECT 
                ano_campeonato,
                tecnico_visitante AS tecnico,
                time_visitante AS time,
                COUNT(*) AS qtd_empates
            FROM jogos
            WHERE gols_mandante = gols_visitante
            GROUP BY ano_campeonato, tecnico_visitante, time_visitante
        ),
        resultados AS (
            SELECT 
                ano_campeonato,
                tecnico,
                time,
                SUM(qtd_vitorias) AS qtd_vitorias,
                SUM(qtd_derrotas) AS qtd_derrotas,
                SUM(qtd_empates) AS qtd_empates,
                (SUM(qtd_vitorias) + SUM(qtd_derrotas) + SUM(qtd_empates)) AS qtd_partidas,
                100* (SUM(qtd_vitorias) * 3 + SUM(qtd_empates))::float / (SUM(qtd_vitorias) * 3 + SUM(qtd_derrotas) * 3 + SUM(qtd_empates) * 3) AS aproveitamento
            FROM (
                SELECT ano_campeonato, tecnico, time, qtd_vitorias, 0 AS qtd_derrotas, 0 AS qtd_empates FROM vitorias
                UNION ALL
                SELECT ano_campeonato, tecnico, time, 0 AS qtd_vitorias, qtd_derrotas, 0 AS qtd_empates FROM derrotas
                UNION ALL
                SELECT ano_campeonato, tecnico, time, 0 AS qtd_vitorias, 0 AS qtd_derrotas, qtd_empates FROM empates
            ) AS combinado
            GROUP BY ano_campeonato, tecnico, time
        ),
        anos_por_tecnico AS (
            SELECT tecnico, COUNT(DISTINCT ano_campeonato) AS anos
            FROM resultados
            GROUP BY tecnico
        ),
        tecnicos_selecionados AS (
            SELECT tecnico
            FROM anos_por_tecnico
            WHERE anos >= 10
        ),
        agg_times AS (
            SELECT 
                ano_campeonato,
                tecnico,
                string_agg(DISTINCT time) AS times,
                SUM(qtd_vitorias) AS qtd_vitorias,
                SUM(qtd_derrotas) AS qtd_derrotas,
                SUM(qtd_empates) AS qtd_empates,
                SUM(qtd_partidas) AS qtd_partidas,
                AVG(aproveitamento) AS aproveitamento
            FROM resultados
            WHERE tecnico IN (SELECT tecnico FROM tecnicos_selecionados)
            GROUP BY ano_campeonato, tecnico
        )
        SELECT 
            ano_campeonato,
            tecnico,
            times,
            qtd_vitorias,
            qtd_derrotas,
            qtd_empates,
            qtd_partidas,
            aproveitamento
        FROM agg_times
        ORDER BY tecnico, ano_campeonato


    ```

    ```js
         const perguntaQuatroGrafico = {
            spec: {
                description: "Aproveitamento dos técnicos ao longo dos anos",
                "data": {
                    values: perguntaQuatro
                },
                "layer": [
                {
                    "selection": {
                    "tecnico_highlight": {
                        "type": "multi",
                        "fields": ["tecnico"],
                        "bind": "legend"
                    }
                    },
                    "mark": "line",
                    "encoding": {
                    "x": {"field": "ano_campeonato", "type": "ordinal", "title": "Ano"},
                    "y": {"field": "aproveitamento", "type": "quantitative", "title": "Aproveitamento"},
                    "color": {
                        "field": "tecnico", 
                        "type": "nominal", 
                        "title": "Técnico",
                    },
                    "opacity": {
                        "condition": {"param": "tecnico_highlight", "value": 1},
                        "value": 0.1
                    }
                    }
                },
                {
                    "mark": "point",
                    "encoding": {
                    "x": {"field": "ano_campeonato", "type": "ordinal", "title": "Ano"},
                    "y": {"field": "aproveitamento", "type": "quantitative", "title": "Aproveitamento"},
                    "color": {
                        "field": "tecnico", 
                        "type": "nominal", 
                        "title": "Técnico",
                    },
                    "opacity": {
                        "condition": {"param": "tecnico_highlight", "value": 1},
                        "value": 0.1
                    },
                    "tooltip": [
                        {"field": "tecnico", "type": "nominal", "title": "Técnico"},
                        {"field": "ano_campeonato", "type": "ordinal", "title": "Ano"},
                        {"field": "times", "type": "nominal", "title": "Times"},
                        {"field": "aproveitamento", "type": "quantitative", "title": "Aproveitamento"}
                    ]
                    }
                }
                ],
                "width": 600,
                "height": 400
            }
                        
            
            }
                

    ```

    ```sql id=perguntaQuatro
            WITH vitorias AS (
            SELECT 
                ano_campeonato,
                tecnico_mandante AS tecnico,
                time_mandante AS time,
                COUNT(*) AS qtd_vitorias
            FROM jogos
            WHERE gols_mandante > gols_visitante
            GROUP BY ano_campeonato, tecnico_mandante, time_mandante
            UNION ALL
            SELECT 
                ano_campeonato,
                tecnico_visitante AS tecnico,
                time_visitante AS time,
                COUNT(*) AS qtd_vitorias
            FROM jogos
            WHERE gols_visitante > gols_mandante
            GROUP BY ano_campeonato, tecnico_visitante, time_visitante
        ),
        derrotas AS (
            SELECT 
                ano_campeonato,
                tecnico_mandante AS tecnico,
                time_mandante AS time,
                COUNT(*) AS qtd_derrotas
            FROM jogos
            WHERE gols_mandante < gols_visitante
            GROUP BY ano_campeonato, tecnico_mandante, time_mandante
            UNION ALL
            SELECT 
                ano_campeonato,
                tecnico_visitante AS tecnico,
                time_visitante AS time,
                COUNT(*) AS qtd_derrotas
            FROM jogos
            WHERE gols_visitante < gols_mandante
            GROUP BY ano_campeonato, tecnico_visitante, time_visitante
        ),
        empates AS (
            SELECT 
                ano_campeonato,
                tecnico_mandante AS tecnico,
                time_mandante AS time,
                COUNT(*) AS qtd_empates
            FROM jogos
            WHERE gols_mandante = gols_visitante
            GROUP BY ano_campeonato, tecnico_mandante, time_mandante
            UNION ALL
            SELECT 
                ano_campeonato,
                tecnico_visitante AS tecnico,
                time_visitante AS time,
                COUNT(*) AS qtd_empates
            FROM jogos
            WHERE gols_mandante = gols_visitante
            GROUP BY ano_campeonato, tecnico_visitante, time_visitante
        ),
        resultados AS (
            SELECT 
                ano_campeonato,
                tecnico,
                time,
                SUM(qtd_vitorias) AS qtd_vitorias,
                SUM(qtd_derrotas) AS qtd_derrotas,
                SUM(qtd_empates) AS qtd_empates,
                (SUM(qtd_vitorias) + SUM(qtd_derrotas) + SUM(qtd_empates)) AS qtd_partidas,
                100* (SUM(qtd_vitorias) * 3 + SUM(qtd_empates))::float / (SUM(qtd_vitorias) * 3 + SUM(qtd_derrotas) * 3 + SUM(qtd_empates) * 3) AS aproveitamento
            FROM (
                SELECT ano_campeonato, tecnico, time, qtd_vitorias, 0 AS qtd_derrotas, 0 AS qtd_empates FROM vitorias
                UNION ALL
                SELECT ano_campeonato, tecnico, time, 0 AS qtd_vitorias, qtd_derrotas, 0 AS qtd_empates FROM derrotas
                UNION ALL
                SELECT ano_campeonato, tecnico, time, 0 AS qtd_vitorias, 0 AS qtd_derrotas, qtd_empates FROM empates
            ) AS combinado
            GROUP BY ano_campeonato, tecnico, time
        ),
        anos_por_tecnico AS (
            SELECT tecnico, COUNT(DISTINCT ano_campeonato) AS anos
            FROM resultados
            GROUP BY tecnico
        ),
        tecnicos_selecionados AS (
            SELECT tecnico
            FROM anos_por_tecnico
            WHERE anos >= 10
        ),
        agg_times AS (
            SELECT 
                ano_campeonato,
                tecnico,
                string_agg(DISTINCT time, ', ') AS times,
                SUM(qtd_vitorias) AS qtd_vitorias,
                SUM(qtd_derrotas) AS qtd_derrotas,
                SUM(qtd_empates) AS qtd_empates,
                SUM(qtd_partidas) AS qtd_partidas,
                AVG(aproveitamento) AS aproveitamento
            FROM resultados
            WHERE tecnico IN (SELECT tecnico FROM tecnicos_selecionados)
            GROUP BY ano_campeonato, tecnico
        ),
        var_tecnico AS (
            SELECT
                tecnico,
                MAX(aproveitamento) - MIN(aproveitamento) AS variacao_aproveitamento
            FROM agg_times
            GROUP BY tecnico
        ),
        top10_tecnicos AS (
            SELECT tecnico
            FROM var_tecnico
            ORDER BY variacao_aproveitamento DESC
            LIMIT 10
        )
        SELECT 
            agg_times.ano_campeonato,
            agg_times.tecnico,
            agg_times.times,
            agg_times.qtd_vitorias,
            agg_times.qtd_derrotas,
            agg_times.qtd_empates,
            agg_times.qtd_partidas,
            agg_times.aproveitamento
        FROM agg_times
        JOIN top10_tecnicos ON agg_times.tecnico = top10_tecnicos.tecnico
        ORDER BY agg_times.tecnico, agg_times.ano_campeonato

    ```

<div  class="card">
    <h1>Aproveitamento dos técnicos com maior variação ao longo dos anos</h1>
    <div id="perguntaQuatroGrafico" style="">
        ${vl.render(perguntaQuatroGrafico)}
    </div>
</div>

