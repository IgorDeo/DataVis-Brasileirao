
# Integrantes

## Logan Miranda e Igor Deo Alves

<hr>

## DataSet Completo

link do dataset [clique aqui](https://basedosdados.org/dataset/c861330e-bca2-474d-9073-bc70744a1b23?table=18835b0d-233e-4857-b454-1fa34a81b4fa)

```js
const arquivo = await FileAttachment("./data/jogos.csv").csv({typed: true});

arquivo.sort((a,b) => {
   
      return b.ano_campeonato - a.ano_campeonato
    
  })
view(Inputs.table(arquivo));
```
