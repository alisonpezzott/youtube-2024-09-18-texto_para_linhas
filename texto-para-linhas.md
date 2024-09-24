### Tabela inicial

| a   | b   | c   |
|-----|-----|-----|
| 124/456/789 | ABC/DEF/GHI | JKL/MNO/PQR |

### Código M

```m
let
    // Cria a tabela inicial com os valores concatenados em cada coluna
    tabelaInicial = Table.FromRecords({[
        a = "124/456/789",
        b = "ABC/DEF/GHI",
        c = "JKL/MNO/PQR"
    ]}),
    
    // Extrai os nomes das colunas da tabela inicial para reutilizar depois
    nomesColunas = Table.ColumnNames(tabelaInicial),
    
    // Converte os valores de cada coluna de texto para uma lista, separando pelo delimitador "/"
    // Aqui, a função Text.Split divide os textos e cria uma lista para cada célula
    textoParaLista = Table.TransformColumns(
        tabelaInicial, {
            {"a", each Text.Split(_, "/"), type {text}},
            {"b", each Text.Split(_, "/"), type {text}},
            {"c", each Text.Split(_, "/"), type {text}}
        }
    ),

    // Cria uma nova coluna "Registros" que combina os valores das colunas 'a', 'b' e 'c' em uma lista de listas
    // A função List.Zip combina os elementos de cada lista (a, b e c) em uma única lista, preservando a ordem
    listaZipada = Table.AddColumn(
        textoParaLista, 
        "Registros", 
        each List.Zip({[a], [b], [c]})
    )[[Registros]],  // Mantém apenas a coluna "Registros", removendo as demais

    // Expande a coluna "Registros", que contém listas, transformando cada item da lista em uma nova linha
    registrosExpandido = Table.ExpandListColumn(listaZipada, "Registros"), 

    // Converte cada lista de valores em um registro, onde cada elemento da lista é associado a um nome de coluna
    registrosApartirDasListas = Table.TransformColumns(
        registrosExpandido, {
            {"Registros", each Record.FromList(_, nomesColunas), type [a=text, b=text, c=text]}
        }
    ), 

    // Expande a coluna de registros (que agora tem colunas a, b, c) de volta em colunas individuais
    resultado = Table.ExpandRecordColumn(
        registrosApartirDasListas, 
        "Registros", 
        nomesColunas
    ) 
    
in
    resultado
```

### Explicação geral:
1. **`tabelaInicial`**: Você define a tabela de entrada, onde os valores de cada coluna são textos que contêm múltiplos valores separados por `/`.
   
2. **`nomesColunas`**: Extraímos os nomes das colunas da tabela inicial para reutilizá-los mais tarde, especialmente ao reconstruir os registros.

3. **`textoParaLista`**: Convertemos os valores das colunas de texto para listas, separando pelos delimitadores `/`. Cada coluna é transformada em uma lista com os valores que antes estavam concatenados.

4. **`listaZipada`**: Criamos uma nova coluna chamada `"Registros"`, que combina os elementos de cada coluna (agora listas) em uma lista de listas. A função `List.Zip` é usada para garantir que os valores de cada coluna permaneçam sincronizados por linha.

5. **`registrosExpandido`**: Expandimos a coluna `"Registros"`, convertendo cada lista em uma nova linha. Isso transforma as listas em múltiplas linhas, uma para cada conjunto de valores.

6. **`registrosApartirDasListas`**: Cada lista de valores é convertida em um registro (um objeto tipo chave-valor), onde os valores são mapeados para os nomes das colunas. Isso recria a estrutura tabular original, mas agora com as linhas individuais.

7. **`resultado`**: Finalmente, expandimos os registros de volta em colunas separadas para cada nome de coluna (a, b, c). Assim, obtemos a tabela final, com cada valor distribuído corretamente nas suas respectivas colunas.

### Resultado:

| a   | b   | c   |
|-----|-----|-----|
| 124 | ABC | JKL |
| 456 | DEF | MNO |
| 789 | GHI | PQR |

Esse código está bem organizado e utiliza boas práticas do M, como evitar produtos cartesianos e garantir que os valores estejam alinhados entre as colunas.
