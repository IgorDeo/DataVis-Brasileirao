---
title: Times campeões
toc: false
sql:
    jogos: ./data/jogos.csv
---


# Times campeões
  
1. **Para o campeao em cada ano ou top 4, Qual é a média de idade do time durante o campeonato.e PARA OS 4 piores times de cada ano?** 


4. **Qual é a média de pontos conquistados pelos 4 times rebaixados em casa em cada temporada?**
    - Justificativa: O fator casa tem muito impacto em um jogo de futebol, principalmente no brasileirao. Queremos saber se o fator casa é preponderante para determinar o rebaixamento ou nao de um time.

## DOS TIMES QUE TIVERAM APROVEITAMENTO EM CASA MENOR QUE A MEDIA DO CAMPEONATO, EM QUAIS POSIÇÕES ELES FICARAM? posiçao media na tabela?


2. **Para o campeão em cada ano, o desempenho jogando fora de casa é pior do que jogando em casa?**
    - Justificativa: Verificar se há uma diferença significativa no desempenho do time campeão jogando como mandante e como visitante pode ajudar a entender a importância do fator casa para conquistar o título.

3. **Quantos pontos um time tem que conseguir ganhar fora de casa para poder ter maiores chances de ser campeão?**
    - Justificativa: Determinar o número mínimo de pontos que um time precisa conquistar fora de casa para aumentar suas chances de ser campeão pode ajudar a estabelecer metas estratégicas para os times.

4. **Qual é a média de pontos conquistados pelo campeão em cada temporada?**
    - Justificativa: Analisar a média de pontos dos campeões ao longo dos anos pode oferecer insights sobre o nível de competição e as mudanças na exigência de desempenho para se tornar campeão.



```sql 
select count(*) from jogos
```