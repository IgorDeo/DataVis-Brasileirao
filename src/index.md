
## DataSet Completo

link do dataset [clique aqui](https://basedosdados.org/dataset/c861330e-bca2-474d-9073-bc70744a1b23?table=18835b0d-233e-4857-b454-1fa34a81b4fa)

```js
const arquivo = await FileAttachment("./data/jogos.csv").csv({typed: true});

arquivo.sort((a,b) => {
   
      return b.ano_campeonato - a.ano_campeonato
    
  })
view(Inputs.table(arquivo));
```


### Análises que iremos fazer:

## Times campeões
1. **A torcida impacta para um time ser campeão?**
    - Justificativa: Analisar a correlação entre o número de torcedores presentes nos jogos e o desempenho dos times pode revelar a influência do apoio da torcida nos resultados do time.
  
2. **Para o campeão em cada ano, o desempenho jogando fora de casa é pior do que jogando em casa?**
    - Justificativa: Verificar se há uma diferença significativa no desempenho do time campeão jogando como mandante e como visitante pode ajudar a entender a importância do fator casa para conquistar o título.

3. **Quantos pontos um time tem que conseguir ganhar fora de casa para poder ter maiores chances de ser campeão?**
    - Justificativa: Determinar o número mínimo de pontos que um time precisa conquistar fora de casa para aumentar suas chances de ser campeão pode ajudar a estabelecer metas estratégicas para os times.

4. **Qual é a média de pontos conquistados pelo campeão em cada temporada?**
    - Justificativa: Analisar a média de pontos dos campeões ao longo dos anos pode oferecer insights sobre o nível de competição e as mudanças na exigência de desempenho para se tornar campeão.

## Geral
1. **Para cada um dos times, qual juiz participou de mais vitórias e mais derrotas de cada time?**
    - Justificativa: Identificar quais árbitros estão mais associados às vitórias e derrotas dos times pode revelar tendências e possíveis impactos das arbitragens nos resultados dos jogos.

2. **Qual é o estádio com maior média de público e como isso impacta no desempenho dos times da casa?**
    - Justificativa: Analisar a média de público dos estádios e o desempenho dos times da casa pode revelar se a presença de um grande número de torcedores influencia positivamente os resultados.

3. **Qual é a distribuição dos gols marcados em diferentes rodadas do campeonato?**
    - Justificativa: Avaliar em quais rodadas ocorrem mais gols pode ajudar a entender padrões de jogo e identificar momentos críticos da competição.

4. **Qual é a relação entre o número de chutes e o número de gols marcados?**
    - Justificativa: Analisar a correlação entre o número de chutes e o número de gols pode oferecer insights sobre a eficiência dos ataques dos times.

## Técnico
Aproveitamento é a quantidade de pontos ganhos divididos pela quantidade de pontos disputados.
1. **Para cada técnico, qual time teve maior aproveitamento?**
    - Justificativa: Avaliar o desempenho dos técnicos em diferentes times pode ajudar a identificar quais técnicos obtêm melhores resultados e em quais contextos eles se destacam.

2. **Quais técnicos têm a maior média de pontos por jogo ao longo de suas carreiras?**
    - Justificativa: Determinar a média de pontos por jogo pode ajudar a identificar técnicos consistentemente bem-sucedidos e compreender seus métodos de trabalho.

3. **Para cada técnico, qual foi o desempenho do time em jogos fora de casa?**
    - Justificativa: Verificar o desempenho dos times sob a liderança de diferentes técnicos em jogos fora de casa pode revelar quais técnicos conseguem motivar e preparar melhor suas equipes para enfrentar desafios longe de casa.

4. **Quais técnicos tiveram a maior variação de desempenho entre diferentes temporadas?**
    - Justificativa: Analisar a variação de desempenho dos técnicos ao longo das temporadas pode ajudar a entender sua capacidade de adaptação e resposta a diferentes condições e contextos de campeonato.


## Extra!
1. 
<!-- ```js
import * as duckdb from "npm:@duckdb/duckdb-wasm";

const db = DuckDBClient.of({brasileirao:  FileAttachment("./data/Brasileirao-DataSet.csv").csv()});
```



```js
const a = await db.queryRow(`SELECT COUNT(*) AS num_jogos FROM brasileirao`);
console.log(a)

``` -->