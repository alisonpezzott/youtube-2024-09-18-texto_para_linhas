### Initial table

| a   | b   | c   |
|-----|-----|-----|
| 124/456/789 | ABC/DEF/GHI | JKL/MNO/PQR |

### M Code

```m
let
    // Create the initial table with concatenated values in each column
    initialTable = Table.FromRecords({[
        a = "124/456/789",
        b = "ABC/DEF/GHI",
        c = "JKL/MNO/PQR"
    ]}),
    
    // Extract the column names from the initial table for reuse later
    columnNames = Table.ColumnNames(initialTable),
    
    // Convert the values in each column from text to a list, splitting by the delimiter "/"
    // Here, the Text.Split function divides the text and creates a list for each cell
    textToList = Table.TransformColumns(
        initialTable, {
            {"a", each Text.Split(_, "/"), type {text}},
            {"b", each Text.Split(_, "/"), type {text}},
            {"c", each Text.Split(_, "/"), type {text}}
        }
    ),

    // Create a new column "Records" that combines the values from columns 'a', 'b', and 'c' into a list of lists
    // The List.Zip function combines the elements from each list (a, b, and c) into a single list, preserving order
    zippedList = Table.AddColumn(
        textToList, 
        "Records", 
        each List.Zip({[a], [b], [c]})
    )[[Records]],  // Keeps only the "Records" column, removing the others

    // Expands the "Records" column, which contains lists, transforming each list item into a new row
    expandedRecords = Table.ExpandListColumn(zippedList, "Records"), 

    // Converts each list of values into a record, where each element of the list is associated with a column name
    recordsFromLists = Table.TransformColumns(
        expandedRecords, {
            {"Records", each Record.FromList(_, columnNames), type [a=text, b=text, c=text]}
        }
    ), 

    // Expands the record column (which now has columns a, b, c) back into individual columns
    finalResult = Table.ExpandRecordColumn(
        recordsFromLists, 
        "Records", 
        columnNames
    ) 
    
in
    finalResult
```

### General explanation:
1. **`initialTable`**: You define the input table, where each column contains concatenated text values separated by `/`.
   
2. **`columnNames`**: We extract the column names from the initial table to reuse them later, especially when reconstructing the records.

3. **`textToList`**: Converts the column values from text into lists by splitting them at the `/` delimiter. Each column is transformed into a list of values that were previously concatenated.

4. **`zippedList`**: Creates a new column called `"Records"`, which combines the elements from each column (now lists) into a list of lists. The `List.Zip` function ensures the values from each column remain synchronized by row.

5. **`expandedRecords`**: Expands the `"Records"` column, converting each list into multiple rows. This step turns the lists into new rows.

6. **`recordsFromLists`**: Each list of values is converted into a record (a key-value object), where the values are mapped to the corresponding column names. This recreates the tabular structure, now with individual rows.

7. **`finalResult`**: Finally, we expand the records back into separate columns for each column name (a, b, c), yielding the final result.

### Output:

| a   | b   | c   |
|-----|-----|-----|
| 124 | ABC | JKL |
| 456 | DEF | MNO |
| 789 | GHI | PQR |

This code is well-organized, using good practices in M, ensuring no Cartesian product and maintaining the correct alignment of values across columns.
