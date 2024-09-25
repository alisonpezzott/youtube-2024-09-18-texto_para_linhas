## Entrada

| a   | b   | c   |
|-----|-----|-----|
| 124/456/789 | ABC/DEF/GHI | JKL/MNO/PQR |

## Tratamento - Power Query M

```pq
let
    // Cria a tabela inicial com os valores concatenados em cada coluna
    tabelaInicial = Table.FromRecords({[
        a = "124/456/789",
        b = "ABC/DEF/GHI",
        c = "JKL/MNO/PQR"
    ]}),

    // Transpõe toda a tabela e transforma em uma lista
    tabelaTransposta = Table.Transpose(tabelaInicial)[Column1],

    // Divide cada item da lista pelo separador
    textoDividido = List.Transform(tabelaTransposta, (linhaAtual) => Text.Split(linhaAtual, "/")),
    
    // Transforma as listas em colunas da tabela
    paraTabela = Table.FromColumns(textoDividido, Table.ColumnNames(tabelaInicial)),
    
    // Altera o tipo de dados
    tipoAlterado = Table.TransformColumnTypes(paraTabela,{{"a", Int64.Type}, {"b", type text}, {"c", type text}})
in
    tipoAlterado
```

## Saída

| a   | b   | c   |
|-----|-----|-----|
| 124 | ABC | JKL |
| 456 | DEF | MNO |
| 789 | GHI | PQR |
