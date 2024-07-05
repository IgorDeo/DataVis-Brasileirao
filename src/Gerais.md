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
const divWidth = Generators.width(document.querySelector("#ex01"));

```

```sql id=gols_por_rodada display
    select rodada ,SUM(gols_mandante + gols_visitante) as Gols_Totais
    FROM jogos
    group by rodada
    order by rodada asc
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


3. **Qual é a relação entre o número de chutes e o número de gols marcados?**
    - Justificativa: Analisar a correlação entre o número de chutes e o número de gols pode oferecer insights sobre a eficiência dos ataques dos times.

    ```sql id=algumaCoisa display
    select ano_campeonato, sum(chutes_mandante + chutes_visitante) as finalizacoes, sum(gols_mandante + gols_visitante) as gols from jogos
    where ano_campeonato > 2017
    group by ano_campeonato
    order by ano_campeonato desc
    ```


2. **Qual é a distribuição dos gols marcados em diferentes rodadas do campeonato?**
    - Justificativa: Avaliar em quais rodadas ocorrem mais gols pode ajudar a entender padrões de jogo e identificar momentos críticos da competição.

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
    
<div id="ex01" class="card">
    <h1>Gols por Rodada</h1>
    <div style="width: 100%; margin-top: 15px;">
        ${ vl.render(ex01(divWidth - 45)) }
    </div>
</div>


4. **Qual é a média de idade dos times por campeonato?**
Com o passar dos anos, os times brasileiros vêm, cada vez mais, contendo jogadores mais velhos em seus times(pois são mais baratos), devido a falta de competitividade financeira com os times da Europa

```sql id=media_idade_por_ano display
select ano_campeonato, ROUND(avg(((idade_media_titular_mandante + idade_media_titular_visitante)/2)/10),2) as media_idade
from jogos
where ano_campeonato >= 2007
group by ano_campeonato
order by ano_campeonato
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


media de publico por rodada do campeonato //a partir de 2014 temos qtd maxima de publico e publico presente//
//somente o publico presente temos em praticamente todos os anos
quais estadios tem a maior quantidade de jogos

```sql id=media_publico_por_campeonato display
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

```sql id= valor_time display
-- Seleciona os 4 melhores times de cada ano na última rodada (rodada 38)
WITH top_4_teams AS (
  SELECT 
    ano_campeonato,
    time_mandante AS time
  FROM 
    jogos
  WHERE 
    rodada = 38 AND colocacao_mandante IN (1, 2, 3, 4)
  UNION
  SELECT 
    ano_campeonato,
    time_visitante AS time
  FROM 
    jogos
  WHERE 
    rodada = 38
    AND colocacao_mandante IN (1, 2, 3, 4)
)
SELECT 
  jogos.*
FROM 
  jogos
JOIN 
  top_4_teams
ON 
  (jogos.ano_campeonato = top_4_teams.ano_campeonato 
   AND (jogos.time_mandante = top_4_teams.time OR jogos.time_visitante = top_4_teams.time))
ORDER BY 
  jogos.ano_campeonato, jogos.rodada;

```

<!-- <!-- 
```js
function ex04(divWidth) {
  return {
    spec: {
      width: divWidth,
      data: {
        values: valor_time,
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
          "field": "valor_medio_elenco",
          "type": "quantitative",
          "title": "Média de Público Presente",
            scale: {
            domain: [0, 28000]
          }
        },
        "tooltip": [
          {"field": "ano_campeonato", "type": "ordinal", "title": "Ano do Campeonato"},
          {"field": "valor_medio_elenco", "type": "quantitative", "title": "Média de Idade"}
        ]
      }
    }
  };
} 
```
-->



<div id="ex04" class="card">
    <h1>Idade x Valor de Elenco</h1>
    <div style="width: 100%; margin-top: 15px;">
        ${ vl.render(ex04(divWidth - 45)) }
    </div>
</div>