---
title: Análises Gerais sobre o Brasileirão
toc: false
sql:
    jogos: ./data/jogos.csv
---
```js
import * as vega from "npm:vega";
import * as vegaLite from "npm:vega-lite";
import * as vegaLiteApi from "npm:vega-lite-api";

const vl = vegaLiteApi.register(vega, vegaLite);
const divWidth = Generators.width(document.querySelector("#ex01"));

```


# Análises Gerais sobre o Brasileirão
<hr>

1. **Para cada um dos times do RJ, qual arbitro participou de mais vitórias e mais derrotas de cada time?**

    Pensamos em fazer uma análise para identificar algum padrão de favoritismo de um árbitro com algum dos 4 principais times do Rio de Janeiro
    Contando a quantidade de vitorias e derrotas dos times apitados por cada um dos árbitros, podemos chegar ao saldo de vitorias.
    A partir disso, foram selecionados os 6 árbitros que apresentaram uma maior discrepância entre os times.


    ```sql id=queryPerguntaUm 
        WITH vitorias AS (
            SELECT 
                arbitro, 
                CASE 
                    WHEN gols_mandante > gols_visitante THEN time_mandante
                    WHEN gols_visitante > gols_mandante THEN time_visitante 
                END AS vencedor,
                COUNT(*) AS qtd_vitorias
            FROM jogos
            GROUP BY arbitro, vencedor
        ),
        derrotas AS (
            SELECT 
                arbitro, 
                CASE 
                    WHEN gols_mandante > gols_visitante THEN time_visitante
                    WHEN gols_visitante > gols_mandante THEN time_mandante 
                END AS perdedor,
                COUNT(*) AS qtd_derrotas
            FROM jogos
            GROUP BY arbitro, perdedor
        ),
        saldos AS (
            SELECT 
                v.arbitro,
                v.vencedor,
                COALESCE(v.qtd_vitorias, 0) AS qtd_vitorias,
                COALESCE(d.qtd_derrotas, 0) AS qtd_derrotas,
                COALESCE(v.qtd_vitorias, 0) - COALESCE(d.qtd_derrotas, 0) AS saldo_vit_der
            FROM vitorias v
            LEFT JOIN derrotas d ON v.arbitro = d.arbitro AND v.vencedor = d.perdedor
            WHERE v.vencedor IN ('Fluminense', 'Vasco da Gama', 'Flamengo', 'Botafogo') AND v.arbitro IS NOT NULL
        ),
        saldos_por_arbitro as (
          select max(saldo_vit_der) -  min(saldo_vit_der) as diff, arbitro from saldos
          group by arbitro
          order by diff desc
          limit 6
        )
        SELECT 
            saldos.arbitro,
            saldos.vencedor,
            SUM(saldos.qtd_vitorias) AS qtd_vitorias,
            -1 * SUM(saldos.qtd_derrotas) AS qtd_derrotas,
            SUM(saldos.saldo_vit_der) AS total_saldo_vit_der
        FROM saldos
        right join saldos_por_arbitro on saldos.arbitro = saldos_por_arbitro.arbitro
        GROUP BY saldos.arbitro, saldos.vencedor
        order by saldos.arbitro

    ```


    ```js
      function graficoDiscrepanciaVitDer(dados) {
        const larguraGrafico = 150;
            return {
              spec: {
              description: "Discrepância entre vitórias e derrotas dos times do RJ com cada árbitro",
              data: {
                values: dados
              },
                      facet: {
                column: {
                  field: "arbitro",
                  type: "nominal",
                  title: "Árbitro"
                }
              },
              spec: {
                layer: [
                  {
                    mark: { type: "bar", tooltip: true },
                    encoding: {
                      x: {
                        field: "vencedor",
                        type: "nominal",
                        title: "Time",
                        sort: "-y"
                      },
                      y: {
                        field: "qtd_vitorias",
                        type: "quantitative",
                        title: "Derrotas e Vitórias",
                        scale: { domain: [-30, 30] }
                      },
                      color: {
                        field: "vencedor",
                        type: "nominal",
                        legend: null
                      }
                    }
                  },
                  {
                    mark: { type: "bar", tooltip: true, color: "red", opacity: 0.6 },
                    encoding: {
                      x: {
                        field: "vencedor",
                        type: "nominal",
                        title: "Time",
                        sort: "-y"
                      },
                      y: {
                        field: "qtd_derrotas",
                        type: "quantitative"
                      },
                      color: {
                        field: "vencedor",
                        type: "nominal",
                        legend: null
                      }
                    }
                  }
                ]
              },
              resolve: {
                scale: {
                  y: "independent"
                }
              },
              width: 300,
              height: 400
              
              }
            }
          }

    ```


    <div class="scrollable-container card">
      ${vl.render(graficoDiscrepanciaVitDer(queryPerguntaUm))}
    </div>

    Podemos observar que, entre os 4 principais times do RJ, alguns árbitros possuem um certo favoritismo...
    O gráfico apresenta a quantiade de derrotas como um valor negativo e a quantidade de vitórias como um valor positivo. Como podemos observar, o Flamengo ganha muito mais do que perde com o árbitro Anderson Daronco, chegando a um saldo positivo de 13 vitórias.
    
    Foi utilizado gráfico de barras, para facilitar a comparação dos dados, já que os dados são de duas categorias diferentes. Os dados do eixo X, "Time", são do tipo catégorico, enquanto que os do eixo Y, "Vitórias e Derrotas", são do tipo quantitativo. O marcador visual é a barra e o canal visual são as diferentes cores aplicadas na barra. Perceba ainda que a barra adota um tom mais esbranquiçado quando os valores do eixo Y são negativos, enquanto adota tons mais fortes para valores positivos.

<hr>

2. **Qual é a distribuição dos gols marcados em diferentes rodadas do campeonato, ao longo de sua história?**
    Buscamos visualizar se existem determinadas rodadas mais propensas a possuirem maior quantidade de gols. Pois, dependendo do momento do campeonato e das necessidades dos times, poderia existir alguma discrepância na quantiadade de gols por rodada.

    ```sql id=gols_por_rodada
      select rodada ,SUM(gols_mandante + gols_visitante) as Gols_Totais
      FROM jogos
      group by rodada
      order by rodada asc
    ```

    ```js
      function ex01(divWidth) {
        return {
          spec: {
            width: divWidth,
            height: 400, // Ajuste a altura conforme necessário
            data: {
              values: gols_por_rodada
            },
            "mark": {
              "type": "line",
              "point": true,
              "tooltip": true
            },
            "encoding": {
              "x": {
                "field": "rodada",
                "type": "ordinal",
                "title": "Rodada"
              },
              "y": {
                "field": "Gols_Totais",
                "type": "quantitative",
                "title": "Total de Gols",
                "scale": {
                  "domain": [500, 600]
                }
              },
              "tooltip": [
                {"field": "rodada", "type": "ordinal", "title": "Rodada"},
                {"field": "Gols_Totais", "type": "quantitative", "title": "Total de Gols"}
              ]
            }
          }
        };
      }
    ```

    <div id="ex01" class="card scrollable-container">
          <h1>Gols por Rodada</h1>
          <div style="width: 100%; margin-top: 15px;">
              ${ vl.render(ex01(divWidth - 45)) }
          </div>
    </div>

    #### Análise da visualização
      Por meio do gráfico de linhas acima, pode-se perceber que não há uma constância ou padrão seguido de acordo com as rodadas. Pelo contrário, há uma variação muito grande de rodada para rodada. No entanto, a partir da 33º rodada até a penúltima(37º), existe um crescimento praticamente exponencial na quantidade de gols que, provavelmente, ocorre por conta do desespero dos times que estão tentando fugir do rebaixamento e para os times que estão lutando por vaga na Libertadores(1º até 6º, atualmente) ou na busca pelo título. A queda na quantidade de gols volta a ocorrer somente na última rodada, por conta da tensão máxima e erro zero exigido, pois não há mais segunda chance, é tudo ou nada.

      Foi utilizado gráfico de linhas, para sermos capazes de observar o total de gols por rodada, conforme o passar dos anos. Os dados do eixo X, "Rodada", são do tipo quantitativo, enquanto que os do eixo Y, "Total de Gols" são do tipo quantitativo. O marcador visual são as linhas, interligadas por pontos e o canal visual é a cor azul aplicada nas linhas.


<hr>

3. **Qual é a média de idade dos times por campeonato?**

    Com o passar dos anos, os times brasileiros vêm, cada vez mais, adquirindo jogadores mais velhos para seus times(pois são mais baratos) e vendendo jogadores mais novos precocemente, devido a falta de competitividade financeira com os times da Europa. Esse cenário sugere um aumento na idade média dos times brasileiros, ainda que as categorias de base foram e ainda são um grande reforço. Portanto, queremos saber se, de fato, a média de idade aumentou por conta do desequilíbrio financeiro em relação a Europa e a chegada de jogadores com idade avançada, ou se ainda manteve-se equilibrada por conta dos jogadores advindos das categorias de base dos clubes.


    ```sql id=media_idade_por_ano
      select ano_campeonato, ROUND(avg(((idade_media_titular_mandante + idade_media_titular_visitante)/2)/10),2) as media_idade
      from jogos
      where ano_campeonato >= 2007
      group by ano_campeonato
      order by ano_campeonato asc
    ```

    ```js

    function ex02(divWidth) {
      return {
        spec: {
          width: divWidth,
          height: 100,
          data: {
            values: media_idade_por_ano,
          },
          "mark": {
            "type": "line",
            "point": true,
            "tooltip": true
          },
          "encoding": {
            "x": {
              "field": "ano_campeonato",
              "type": "ordinal",
              "title": "Ano do Campeonato"
            },
            "y": {
              "field": "media_idade",
              "type": "quantitative",
              "title": "Média de Idade",
              "scale": {
                "domain": [23, 26]
              }
            },
            "tooltip": [
              {"field": "ano_campeonato", "type": "ordinal", "title": "Ano do Campeonato"},
              {"field": "media_idade", "type": "quantitative", "title": "Média de Idade"}
            ]
          }
        }
      };
    }

    ```
      
    <div id="ex02" class="card">
        <h1>Media time titular por Ano do Campeonato</h1>
        <div style="width: 100%; margin-top: 15px;">
            ${ vl.render(ex02(divWidth - 45)) }
        </div>
    </div>


    #### Análise da visualização
    A visualização confirma a tese de que a idade média dos times do Brasileirão aumentaram conforme o passar dos anos. Note que, no ínicio, em 2007, a média de idade era menor que 24 anos. Nos anos subsequentes houve um aumento gradual na média dos times brasileiros e atualmente a idade média e maior que 25 anos. Pode não parecer um grande aumento, mas, considerando o conceito de média e uma quantidade de anos não muito grande, é um fator significativo.

    Foi utilizado gráfico de linhas, para sermos capazes de observar o total de gols por rodada, conforme o passar dos anos. Os dados do eixo X, "Ano do Campeonato", são do tipo temporal, enquanto que os do eixo Y, "Média de Idade" são do tipo quantitativo. O marcador visual são as linhas, interligadas por pontos e o canal visual é a cor azul aplicada nas linhas.


<hr>

4. **A média de público vem crescendo ou diminuindo?**

    Ultimamente, acredita-se que o futebol está mais chato e que a qualidade dos jogos está menor. Portanto, queremos visualizar a média de público presente nos estádios por ano, se está aumentando ou diminuindo.

    ```sql id=media_publico_por_campeonato 
      select ano_campeonato, ROUND(AVG(publico),2) as media_publico_presente
      from jogos
      where ano_campeonato > 2006
      group by ano_campeonato
      order by ano_campeonato
    ```

    ```js
      function ex03(divWidth) {
        return {
          spec: {
            width: divWidth,
            data: {
              values: media_publico_por_campeonato,
            },
            "mark": {
              "type": "bar",
              "point": true,
              "tooltip": true
            },
            "encoding": {
              "x": {
                "field": "ano_campeonato",
                "type": "ordinal",
                "title": "Ano do Campeonato"
              },
              "y": {
                "field": "media_publico_presente",
                "type": "quantitative",
                "title": "Média de Público Presente",
                  scale: {
                  domain: [0, 28000]
                }
              },
              "tooltip": [
                {"field": "ano_campeonato", "type": "ordinal", "title": "Ano do Campeonato"},
                {"field": "media_publico_presente", "type": "quantitative", "title": "Média de Idade"}
              ]
            }
          }
        };
      }
    ```

    <div id="ex03" class="card">
        <h1>Media de publico presente por Ano do Campeonato</h1>
        <div style="width: 100%; margin-top: 15px;">
            ${ vl.render(ex03(divWidth - 45)) }
        </div>
    </div>

    #### Análise da visualização

    Apesar do público, de uma forma geral, questionar e reclamar sobre a qualidade do futebol brasileiro atualmente, a visualização nos mostra que a média de público presente nos jogos do campeonato brasileiro está aumentando a cada ano, inclusive batendo recordes nos últimos anos. Note que em 2020 e 2021 a média foi irrisória, isso se deve a pandemia do Coronavírus, que não permitia público nos estádios. Em 2021 ocorreu uma liberação gradual e no meio do campeonato da presença de público nos estádios, variando de acordo com o que cada governo estadual decidia, por isso a média não é zero, mas extremamente baixa. Porém, os anos subsequentes mantiveram o constante aumento na média de público presente nos estádios.

    Foi utilizado gráfico de barras, para facilitar a comparação dos dados. Os dados do eixo X, "Ano do Campeonato", são do tipo temporal, enquanto que os do eixo Y, "Média de Público Presente", são do tipo quantitativo. O marcador visual é a barra e o canal visual é a cor aplicada na barra que, nesse caso, é o azul.
