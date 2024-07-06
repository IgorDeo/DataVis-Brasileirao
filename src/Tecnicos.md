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
Aproveitamento de pontos é a quantidade de pontos ganhos divididos pela quantidade de pontos disputados.
Aproveitamento de partidas é a quantidade de vitórias divididas pela a quantidade de partidas 

1. **Quais técnicos têm a maior média de pontos por jogo ao longo de suas carreiras?**
    - Justificativa: Determinar a média de pontos por jogo pode ajudar a identificar técnicos consistentemente bem-sucedidos.

    ```sql id=perguntaDois
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
                        END AS tecnico_vencedor,
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
                        END AS tecnico_perdedor,
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
                ),
                total_jogos_por_tecnico AS (
                    SELECT 
                        tecnico,
                        SUM(qtd_partidas) AS total_jogos
                    FROM resultados
                    WHERE tecnico IS NOT NULL
                    GROUP BY tecnico
                ),
                tecnicos_com_mais_jogos AS (
                    SELECT tecnico, total_jogos
                    FROM total_jogos_por_tecnico
                    ORDER BY total_jogos DESC
                    LIMIT 5
                ),
                media_pontos_por_tecnico_ano AS (
                    SELECT 
                        ano_campeonato,
                        tecnico,
                        STRING_AGG(DISTINCT time, ' ') AS times,
                        SUM(pontos) AS total_pontos,
                        SUM(qtd_partidas) AS total_partidas,
                        AVG(pontos::float / qtd_partidas) AS media_pontos_jogo
                    FROM resultados
                    WHERE tecnico IS NOT NULL
                    GROUP BY ano_campeonato, tecnico
                )
                SELECT 
                    m.ano_campeonato,
                    m.tecnico,
                    m.times,
                    m.total_pontos,
                    m.total_partidas,
                    m.media_pontos_jogo,
                    t.total_jogos
                FROM media_pontos_por_tecnico_ano m
                JOIN tecnicos_com_mais_jogos t ON m.tecnico = t.tecnico
                ORDER BY m.tecnico DESC, m.ano_campeonato

    ```
    


    ```js
    function perguntaDoisGrafico(dados){
        const mediaDePontos = d3.mean(dados, d => d.media_pontos_jogo);
         return {
            spec: {
            description: "Aproveitamento dos técnicos ao longo dos anos",
            data: {
                values: perguntaDois
            },
            layer: [
                {
                selection: {
                    tecnico_highlight: {
                    type: "multi",
                    fields: ["tecnico"],
                    bind: "legend"
                    }
                },
                mark: "line",
                encoding: {
                    x: { field: "ano_campeonato", type: "ordinal", title: "Ano" },
                    y: { field: "media_pontos_jogo", type: "quantitative", title: "Média de pontos por partida" },
                    color: { field: "tecnico", type: "nominal", title: "Técnico" },
                    opacity: {
                    condition: { param: "tecnico_highlight", value: 1 },
                    value: 0.1
                    }
                }
                },
                {
                mark: "point",
                encoding: {
                    x: { field: "ano_campeonato", type: "ordinal", title: "Ano" },
                    y: { field: "media_pontos_jogo", type: "quantitative", title: "Média de pontos por partida" },
                    color: { field: "tecnico", type: "nominal", title: "Técnico" },
                    opacity: {
                    condition: { param: "tecnico_highlight", value: 1 },
                    value: 0.1
                    },
                    tooltip: [
                    { field: "tecnico", type: "nominal", title: "Técnico" },
                    { field: "ano_campeonato", type: "ordinal", title: "Ano" },
                    { field: "times", type: "nominal", title: "Times" },
                    { field: "media_pontos_jogo", type: "quantitative", title: "Média de pontos por partida" },
                    {field: "total_partidas", type:"quantitative", title: "Quantidade de partidas"}
                    ]
                }
                },
                   {
          mark: "rule",
          encoding: {
            y: {
              datum: mediaDePontos, // Use "datum" para posicionar a linha corretamente
              title: "Média de pontos"
            },
            size: { value: 1 },
            color: { value: "purple" },
            tooltip: { value: `Média de pontos: ${mediaDePontos.toFixed(2)}%` }
          }
        },
        {
          mark: {
            type: "text",
            align: "left",
            dx: 3,
            dy: -3,
          },
          encoding: {
            y: {
              datum: mediaDePontos, // Use "datum" para posicionar o texto corretamente
              title: "Média de pontos"
            },
            text: { value: `Média: ${mediaDePontos.toFixed(2)}%`,  },
            color: { value: "purple" }
          }
        }
            ],
            width: 600,
            height: 400
            }
        };
        }

    ```

    <div class="card">
    ${vl.render(perguntaDoisGrafico(perguntaDois))}
    </div>

    ## Análise da visualização
    O gráfico mostra os Top-5 técnicos com maior média de pontos por partida do campeonato brasileiro. Podemos observar que não há nenhuma tendência de melhora ou piora para cada um dos técnicos, todos tem seu desempenho bem variado ao longo dos anos, sendo o Cuca o que melhor performou, com um aproveitamento de pontos médio de 2.26 pontos por partida em 2016 liderando o Palmeiras.

    Foi utilizado gráfico de linhas, para sermos capazes de observar a méida de pontos de cada técnico por partida, conforme o passar dos anos. Os dados do eixo X, "Ano", são do tipo temporal, enquanto que os do eixo Y, "Média de pontos por partida" são do tipo quantitativo. O marcador visual são as linhas, interligadas por pontos e o canal visual são as diferentes cores aplicada nas linhas, em que cada uma representa um técnico distinto.

<hr>

2. **Para cada técnico, qual foi o desempenho do time em jogos fora de casa?**
    
     Todos sabem que é sempre mais difícil um time performar jogando como visitante da mesma maneira que quando é mandante. Estar sem a presença da sua torcida com certeza impacta na moral da equipe que pode acabar falhando dentro de campo. Saber motivar, arquitetar e conduzir uma equipe de maneira a obter excelencia dentro de campo é um desafio para qualquer bom técnico. Assim, buscamos observar quais foram os melhores técnicos a cumprir com a árdua tarefa de vencer fora de casa, obtendo o maior aproveitamento jogando como visitante.

    Consideramos somente os treinadores que possuem um número de partidas disputadas maior do que a média de todos os treinadores a fim de manter uma visualização mais clara.

    ```sql id=perguntaTres
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
            STRING_AGG(t.times, ' ') AS times,
            sum(t.total_jogos) as total_jogos,
            avg(t.aproveitamento) as aproveitamento
        FROM tecnico_time_max_aproveitamento t
        JOIN media_total_jogos m ON t.total_jogos >= m.avg_jogos
        group by t.tecnico
        ORDER BY avg(t.aproveitamento) desc, t.tecnico  DESC
        LIMIT 15

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

    <div class="card ">
    <h1>Técnicos e time com maior aproveitamento em partidas como visitante</h1>
    ${vl.render(perguntaTresGrafico)}
    </div>

    Como podemos observar acima, Jorge Jesus foi o treinador que obteve o maior aproveitamento nas partidas fora de casa, obtendo cerca de 71% vitórias jogando como visitante. Esse feito foi durante sua passagem pelo Flamengo em 2019, ano quando o rubro negro conquistou o Campeonato Brasileiro e Taça Libertadores da América. Curiosamente, outros 5 treinadores aparecem na lista como top vencedores visitantes como técnicos do Flamengo: Domènec Torrent, Renato Portaluppi, Jorge Sampaoli, Andrade, Rogério Ceni. Luís Castro, ex treinador do Botafogo(2022-2023) aparece como terceiro da lista com 52% de aproveitamento, que talvez pudesse ser maior se ele não tivesse deixado o Botafogo no meio do campeonato brasileiro de 2023. Edição essa na qual o Botafogo executou a maior pipocada da história, conseguindo perder um campeonato semi-ganho para o Palmeiras.

    Foi utilizado gráfico de barras, para facilitar a comparação dos dados, já que os dados são de duas categorias diferentes. Os dados do eixo Y, "Técnicos", são do tipo catégorico, enquanto que os do eixo X, "Aproveitamento(%)", são do tipo quantitativo. O marcador visual é a barra e o canal visual são as cores aplicadas nas barras.



<hr>

3. **Quanto de aproveitamento um técnico tem que ter para ser convocado para seleção brasileira?**
    Buscamos visualizar qual é a porcentagem de aproveitamento de pontos no campeonato Brasileiro, que credencia um técnico a ser convocado para a seleção brasileria
        
    **OBS: é possivel clicar em um nome da legenda para visualizar individualmente cada treinador**
        
    - Os técnicos considerados foram: Mano Menezes, Tite, Fernando Diniz, Luiz Felipe Scolari, Dorival Júnior

     ```js
    function graficoMultiplasLinhasAproveitamentoTecnico(dados) {
        const mediaAproveitamento = d3.mean(dados, d => d.aproveitamento);

        return {
            spec: {
            description: "Aproveitamento dos técnicos ao longo dos anos",
            data: {
                values: dados
            },
            layer: [
                {
                selection: {
                    tecnico_highlight: {
                    type: "multi",
                    fields: ["tecnico"],
                    bind: "legend"
                    }
                },
                mark: "line",
                encoding: {
                    x: { field: "ano_campeonato", type: "ordinal", title: "Ano" },
                    y: { field: "aproveitamento", type: "quantitative", title: "Aproveitamento" },
                    color: { field: "tecnico", type: "nominal", title: "Técnico" },
                    opacity: {
                    condition: { param: "tecnico_highlight", value: 1 },
                    value: 0.1
                    }
                }
                },
                {
                mark: "point",
                encoding: {
                    x: { field: "ano_campeonato", type: "ordinal", title: "Ano" },
                    y: { field: "aproveitamento", type: "quantitative", title: "Aproveitamento" },
                    color: { field: "tecnico", type: "nominal", title: "Técnico" },
                    opacity: {
                    condition: { param: "tecnico_highlight", value: 1 },
                    value: 0.1
                    },
                    tooltip: [
                    { field: "tecnico", type: "nominal", title: "Técnico" },
                    { field: "ano_campeonato", type: "ordinal", title: "Ano" },
                    { field: "times", type: "nominal", title: "Times" },
                    { field: "aproveitamento", type: "quantitative", title: "Aproveitamento" },
                    {field: "qtd_partidas", type:"quantitative", title: "Quantidade de partidas"}
                    ]
                }
                },
                   {
          mark: "rule",
          encoding: {
            y: {
              datum: mediaAproveitamento, // Use "datum" para posicionar a linha corretamente
              title: "Média de Aproveitamento"
            },
            size: { value: 1 },
            color: { value: "purple" },
            tooltip: { value: `Média de Aproveitamento: ${mediaAproveitamento.toFixed(2)}%` }
          }
        },
        {
          mark: {
            type: "text",
            align: "left",
            dx: 3,
            dy: -3,
          },
          encoding: {
            y: {
              datum: mediaAproveitamento, // Use "datum" para posicionar o texto corretamente
              title: "Média de Aproveitamento"
            },
            text: { value: `Média: ${mediaAproveitamento.toFixed(2)}%`,  },
            color: { value: "purple" }
          }
        }
            ],
            width: 600,
            height: 400
            }
        };
        }

   

                

    ```

    ```sql id=tecnicosDaSelecao
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
            WHERE tecnico in ('Mano Menezes', 'Tite', 'Fernando Diniz', 'Luiz Felipe Scolari', 'Dorival Júnior')
            GROUP BY ano_campeonato, tecnico
        ),
        media_aproveitamento as (
            select avg(aproveitamento) as aproveitamento_medio from agg_times
        )
        SELECT 
            agg_times.ano_campeonato,
            agg_times.tecnico,
            agg_times.times,
            agg_times.qtd_vitorias,
            agg_times.qtd_derrotas,
            agg_times.qtd_empates,
            agg_times.qtd_partidas,
            agg_times.aproveitamento,
            media_aproveitamento
        FROM agg_times, media_aproveitamento
        ORDER BY agg_times.tecnico, agg_times.ano_campeonato

    ```

    <div  class="card ">
    <h1>Aproveitamento dos técnicos convocados para seleção brasileira</h1>
    <div id="tecnicosDaSelecao" style="">
        ${vl.render(graficoMultiplasLinhasAproveitamentoTecnico(tecnicosDaSelecao))}
    </div>
    </div>

    #### Análise da visualização
    Apesar do ótimo rendimento dos técnicos convocados variar, o aproveitamento mais baixo deles ficam, quase sempre, próximos da média, com algumas poucas exceções. Isso mostra que, mesmo na fase ruim, o aproveitamento dos técnicos é próximo da média. Podemos perceber também que os técnicos tiveram ótimos aproveitamentos, ultrapassando ou chegando bem próximo de 70% de aproveitamento. Portanto, o nível de exigência é bem elevado para ter chance de ser convocado para a maior seleção do futebol mundial. 
    
    Foi utilizado gráfico de linhas, para sermos capazes de observar o aproveitamento de pontos de cada técnico, conforme o passar dos anos. Os dados do eixo X, "Ano", são do tipo temporal, enquanto que os do eixo Y, "Aproveitamento" são do tipo quantitativo. O marcador visual são as linhas, interligadas por pontos e o canal visual são as diferentes cores aplicada nas linhas, em que cada uma representa um técnico distinto.

<hr>
