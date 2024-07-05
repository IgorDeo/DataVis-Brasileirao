---
title: Gerais
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


# Geral
1. **Para cada um dos times do RJ, qual arbitro participou de mais vitórias e mais derrotas de cada time?**
    - Justificativa: Identificar quais árbitros estão mais associados às vitórias e derrotas dos times pode revelar tendências e possíveis impactos das arbitragens nos resultados dos jogos.
    ```sql id=queryPerguntaUm display
         select 
            v.arbitro,
            v.vencedor,
            coalesce(v.qtd_vitorias,0) as qtd_vitorias,
            coalesce(d.qtd_derrotas,0) as qtd_derrotas
        from (
            select 
                arbitro, 
                case 
                    when gols_mandante > gols_visitante then time_mandante
                    when gols_visitante > gols_mandante then time_visitante 
                end as vencedor,
                count(*) as qtd_vitorias
            from jogos
            group by cube(arbitro, vencedor)
        ) v
        left join (
            select 
                arbitro, 
                case 
                    when gols_mandante > gols_visitante then time_visitante
                    when gols_visitante > gols_mandante then time_mandante 
                end as perdedor,
                count(*) as qtd_derrotas
            from jogos
            group by cube(arbitro, perdedor)
        ) d on v.arbitro = d.arbitro and v.vencedor = d.perdedor
        where vencedor in ('Fluminense', 'Vasco', 'Flamengo','Botafogo') and v.arbitro is not null
        order by qtd_vitorias desc;
    ```



2. **Qual é a distribuição dos gols marcados em diferentes rodadas do campeonato?**
    - Justificativa: Avaliar em quais rodadas ocorrem mais gols pode ajudar a entender padrões de jogo e identificar momentos críticos da competição.

3. **Qual é a relação entre o número de chutes e o número de gols marcados?**
    - Justificativa: Analisar a correlação entre o número de chutes e o número de gols pode oferecer insights sobre a eficiência dos ataques dos times.

    ```sql id=algumaCoisa display
    select ano_campeonato, sum(chutes_mandante + chutes_visitante) as finalizacoes, sum(gols_mandante + gols_visitante) as gols from jogos
    group by ano_campeonato
    order by gols desc
    ```

    ```sql
    select ano_campeonato, sum(publico) as publico from jogos
    group by ano_campeonato
    order by ano_campeonato desc
    ```



```js
const divWidth = Generators.width(document.querySelector("#ex01"));

function ex01(divWidth) {
    return {
        spec: {
            width: divWidth,
            data: { values: queryPerguntaUm },
            mark: { 
                type: "bar", 
                cornerRadiusTopLeft: 3, 
                cornerRadiusTopRight: 3 
            },
            encoding: {
                x: { 
                    field: "vencedor", 
                    type: "ordinal",  // Considerando anos como categorias
                },
                y: { 
                    field: "arbitro", 
                    type: "nominal", 
                    title: "Gols" 
                },
                color: { field: "qtd_vitorias" }  // Mantendo o campo de cor baseado em "arbitro"
            }
        }
    };
}

```
<div id="ex01" class="card">
    <h1>Gols por Ano do Campeonato</h1>
    <div style="width: 100%; margin-top: 15px;">
        ${ vl.render(ex01(divWidth - 45)) }
    </div>
</div>


```sql
select * from jogos order by ano_campeonato desc
```