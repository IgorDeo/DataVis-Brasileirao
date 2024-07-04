
## DataSet Completo

link do dataset [clique aqui](https://basedosdados.org/dataset/c861330e-bca2-474d-9073-bc70744a1b23?table=18835b0d-233e-4857-b454-1fa34a81b4fa)

```js
const arquivo = await FileAttachment("./data/Brasileirao-DataSet.csv").csv({typed: true});

arquivo.sort((a,b) => {
   
      return b.ano_campeonato - a.ano_campeonato
    
  })
view(Inputs.table(arquivo));
```


# Análises que iremos fazer:
## Times camepoes
1. A torcida impacta para um time ser campeao?
2. Para o campeao em cada ano, o desempenho jogando fora de casa é pior do que jogando em casa?
3. Quantos pontos um time tem que conseguir ganhar fora de casa para poder ter maiores chances de ser campeao?

## Geral
1. Para cada um dos times, qual juiz participou de mais vitórias e mais derrotas de cada time?
<!-- 2. O desempenho de um time -->

## Tecnico
Aproveitamento é a quantidade de pontos ganhos divididos pela quantidade de pontos disputados.
1. Para cada tecnico, qual time teve maior aproveitamento?

<!-- ```js
import * as duckdb from "npm:@duckdb/duckdb-wasm";

const db = DuckDBClient.of({brasileirao:  FileAttachment("./data/Brasileirao-DataSet.csv").csv()});
```



```js
const a = await db.queryRow(`SELECT COUNT(*) AS num_jogos FROM brasileirao`);
console.log(a)

``` -->