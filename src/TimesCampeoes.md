---
title: Times Campeões VS Times Rebaixados
toc: false
sql:
    jogos: ./data/jogos.csv
---

```js
const divWidth = Generators.width(document.querySelector("#ex01"));
```

# Times Campeões VS Times Rebaixados

<hr>
    
```sql id=campeoes
WITH campeoes AS (
    SELECT 
        ano_campeonato,
        time_campeao,
        SUM(pontos) AS pontos
    FROM (
        SELECT 
            ano_campeonato,
            time_mandante AS time_campeao,
            3 * SUM(CASE WHEN gols_mandante > gols_visitante THEN 1 ELSE 0 END) + SUM(CASE WHEN gols_mandante = gols_visitante THEN 1 ELSE 0 END) AS pontos
        FROM jogos
        GROUP BY ano_campeonato, time_mandante
        UNION ALL
        SELECT 
            ano_campeonato,
            time_visitante AS time_campeao,
            3 * SUM(CASE WHEN gols_visitante > gols_mandante THEN 1 ELSE 0 END) + SUM(CASE WHEN gols_visitante = gols_mandante THEN 1 ELSE 0 END) AS pontos
        FROM jogos
        GROUP BY ano_campeonato, time_visitante
    ) AS pontuacao
    GROUP BY ano_campeonato, time_campeao
    ORDER BY ano_campeonato, pontos DESC
),
times_campeoes AS (
    SELECT DISTINCT ON (ano_campeonato) 
        ano_campeonato,
        time_campeao
    FROM campeoes
    ORDER BY ano_campeonato, pontos DESC
)
SELECT
  jogos.ano_campeonato,
  jogos.rodada,
  jogos.estadio,
  jogos.time_mandante,
  jogos.time_visitante,
  jogos.gols_mandante,
  jogos.gols_visitante,
  times_campeoes.time_campeao AS campeao,
  CASE
    WHEN jogos.time_mandante = times_campeoes.time_campeao THEN jogos.escanteios_mandante
    ELSE jogos.escanteios_visitante
  END AS escanteios_campeao,
  CASE
    WHEN jogos.time_mandante = times_campeoes.time_campeao THEN jogos.chutes_mandante
    ELSE jogos.chutes_visitante
  END AS chutes_campeao
    
FROM jogos
JOIN times_campeoes 
ON jogos.ano_campeonato = times_campeoes.ano_campeonato
AND (jogos.time_mandante = times_campeoes.time_campeao OR jogos.time_visitante = times_campeoes.time_campeao)
ORDER BY jogos.ano_campeonato desc, jogos.rodada desc;
```

```js

function ex01(divWidth) {
    return {
        spec: {
        "data": {
            values: campeoes  },
        "transform": [{
            "filter": {"and": [
                {"field": "chutes_campeao", "valid": true}, 
                {"field": "escanteios_campeao", "valid": true} 
            ]}
        }],
        "mark": "rect",
        "width": 300,
        "height": 200,
        "encoding": {
            "y": {
                "bin": {"maxbins": 7},
                "field": "chutes_campeao", 
                "title": "chutes_campeao",
                "type": "quantitative"
            },
            "x": {
                "bin": {"maxbins": 7},
                "field": "escanteios_campeao", 
                "title": "escanteios_campeao",
                "type": "quantitative"
            },
            "color": {
                "aggregate": "count",
                "type": "quantitative"
            }
        },
        "config": {
            "view": {
                "stroke": "transparent"
            }
        }
    }
  };
}
```

# **Existe alguma correlação de características que aproximam um time de se tornar campeão?**
Para isso, buscamos todos os jogos dos times campeões de cada ano e analisamos algumas características ofensivas desses times, buscando encontrar um padrão para um time ter maior probabilidade de se tornar campeão

<div id="ex01" class="card">
        <h1>Volume Ofensivo de Times Campeões</h1>
        <div style="width: 100%; margin-top: 15px;">
            ${ vl.render(ex01(divWidth - 175)) }
        </div>
</div>

## Análise da visualização
Como podemos perceber pelo gráfico de heatmap gerado, há uma intensificação de valores quando os times campeões possuem entre 5 e 10 escanteios e de 10 a 20 finalizações no jogo. Isso comprova que, para um time ter maior chance de ser campeão é necessário ter um alto volume ofensivo por jogo, de modo que isso o aproxima de vencer. 

O marcador visual utilizado nesse gráfico são as células, que possui como canal visual diferentes cores, em que quanto mais escura, ou seja, quanto mais forte é a cor, há maior frequência desses valores no conjunto de dados.


```sql id=rebaixados 
-- Calcular pontos por jogo para cada time
WITH pontos_por_jogo AS (
    SELECT 
        ano_campeonato,
        rodada,
        time_mandante,
        time_visitante,
        gols_mandante,
        gols_visitante,
        CASE 
            WHEN gols_mandante > gols_visitante THEN 3
            WHEN gols_mandante = gols_visitante THEN 1
            ELSE 0
        END AS pontos_mandante,
        CASE 
            WHEN gols_visitante > gols_mandante THEN 3
            WHEN gols_visitante = gols_mandante THEN 1
            ELSE 0
        END AS pontos_visitante
    FROM jogos
),
-- Calcular pontos acumulados por time e ano
pontos_acumulados AS (
    SELECT 
        ano_campeonato,
        time_mandante AS time,
        SUM(pontos_mandante) AS pontos
    FROM pontos_por_jogo
    GROUP BY ano_campeonato, time_mandante
    UNION ALL
    SELECT 
        ano_campeonato,
        time_visitante AS time,
        SUM(pontos_visitante) AS pontos
    FROM pontos_por_jogo
    GROUP BY ano_campeonato, time_visitante
),
-- Selecionar os 4 piores times de cada ano
piores_times AS (
    SELECT ano_campeonato, time
    FROM (
        SELECT 
            ano_campeonato,
            time,
            RANK() OVER (PARTITION BY ano_campeonato ORDER BY SUM(pontos) ASC) AS rank
        FROM pontos_acumulados
        GROUP BY ano_campeonato, time
    ) AS ranked
    WHERE rank <= 4
),
-- Criar uma lista de partidas envolvendo os piores times sem duplicação
partidas_piores_times AS (
    SELECT DISTINCT
        jogos.ano_campeonato,
        jogos.rodada,
        jogos.estadio,
        jogos.time_mandante,
        jogos.time_visitante,
        jogos.gols_mandante,
        jogos.gols_visitante,
        CASE
            WHEN piores_times.time = jogos.time_mandante THEN jogos.escanteios_mandante
            ELSE jogos.escanteios_visitante
        END AS escanteios_pior,
        CASE
            WHEN piores_times.time = jogos.time_mandante THEN jogos.chutes_mandante
            ELSE jogos.chutes_visitante
        END AS chutes_pior
    FROM jogos
    JOIN piores_times 
    ON jogos.ano_campeonato = piores_times.ano_campeonato
    AND (jogos.time_mandante = piores_times.time OR jogos.time_visitante = piores_times.time)
)
SELECT
    ano_campeonato,
    rodada,
    estadio,
    time_mandante,
    time_visitante,
    gols_mandante,
    gols_visitante,
    escanteios_pior,
    chutes_pior
FROM partidas_piores_times
ORDER BY ano_campeonato DESC, rodada DESC;
```


```js
function ex02(divWidth) {
    return {
        spec: {
        "data": {
            values: rebaixados},
        "transform": [{
            "filter": {"and": [
                {"field": "chutes_pior", "valid": true}, 
                {"field": "escanteios_pior", "valid": true} 
            ]}
        }],
        "mark": "rect",
        "width": 300,
        "height": 200,
        "encoding": {
            "y": {
                "bin": {"maxbins": 7},
                "field": "chutes_pior", 
                "title": "chutes_pior",
                "type": "quantitative"
            },
            "x": {
                "bin": {"maxbins": 7},
                "field": "escanteios_pior", 
                "title": "escanteios_pior",
                "type": "quantitative"
            },
            "color": {
                "aggregate": "count",
                "type": "quantitative"
            }
        },
        "config": {
            "view": {
                "stroke": "transparent"
            }
        }
    }
  };
}
```

<br>
<hr>

# **Existe alguma "receita" para um time ter mais chance de ser rebaixado?**
Analogamente ao que fizemos para os times campeões, queremos descobrir se existe algum conjunto de características que aproximam um time de cair para a segunda divisão do campeonato brasileiro.

<div id="ex02" class="card">
        <h1>Volume Ofensivo - Times Rebaixados</h1>
        <div style="width: 100%; margin-top: 15px;">
            ${ vl.render(ex02(divWidth))}
        </div>
</div>

## Análise da visualização
Podemos perceber que existe uma concentração maior nas faixas menores de chutes e escanteios, as cores mais escuras estã presentes mais à esquerda e na parte inferior do gráfico, quando comparamos com o mesmo heatmap para os times que foram campeões. Para os piores times do Brasileirão, note que há uma condensação maior de valores na faixa de escanteios, variando de 0 até 5, e chutes, variando de 5 a 10, formam a parte mais escura do gráfico, portanto, a que possui maior número de jogos com tal característica. Logo, os times rebaixados atacam pouco o time adversário, de modo que oferecem pouco risco e, dessa forma, é muito mais dificíl fazer gols e, consequentemente, ganhar jogos. 

Os marcadores e canais visuais utilizados são os mesmos explicados no último gráfico plotado acima.