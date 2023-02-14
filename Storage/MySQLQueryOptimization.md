# MySQL Query Optimization

## Explain

The EXPLAIN statement provides information about how MySQL executes statements. EXPLAIN works with SELECT, DELETE, INSERT, REPLACE, and UPDATE statements.

EXPLAIN returns a row of information for each table used in the SELECT statement. It lists the tables in the output **in the order that MySQL would read them while processing the statement.** This means that MySQL reads a row from the first table, then finds a matching row in the second table, and then in the third table, and so on. When all tables are processed, MySQL outputs the selected columns and backtracks through the table list until a table is found for which there are more matching rows. The next row is read from this table and the process continues with the next table.



## EXPLAIN Output Columns

**Table 8.1 EXPLAIN Output Columns**

| Column                                                                                               | JSON Name       | Meaning                                        |
| ---------------------------------------------------------------------------------------------------- | --------------- | ---------------------------------------------- |
| [`id`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_id)                       | `select_id`     | The `SELECT` identifier                        |
| [`select_type`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_select_type)     | None            | The `SELECT` type                              |
| [`table`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_table)                 | `table_name`    | The table for the output row                   |
| [`partitions`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_partitions)       | `partitions`    | The matching partitions                        |
| [`type`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_type)                   | `access_type`   | The join type                                  |
| [`possible_keys`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_possible_keys) | `possible_keys` | The possible indexes to choose                 |
| [`key`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_key)                     | `key`           | The index actually chosen                      |
| [`key_len`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_key_len)             | `key_length`    | The length of the chosen key                   |
| [`ref`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_ref)                     | `ref`           | The columns compared to the index              |
| [`rows`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_rows)                   | `rows`          | Estimate of rows to be examined                |
| [`filtered`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_filtered)           | `filtered`      | Percentage of rows filtered by table condition |
| [`Extra`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_extra)                 | None            | Additional information                         |



### 各字段含义

### id (JSON name: select_id)

This is the sequential number of the [`SELECT`](https://dev.mysql.com/doc/refman/8.0/en/select.html "13.2.13 SELECT Statement") within the query. The value can be `NULL` if the row refers to the union result of other rows. In this case, the `table` column shows a value like ``<union*`M`*,*`N`*>`` to indicate that the row refers to the union of the rows with `id` values of *`M`* and *`N`*.



### select_type (JSON name: none)

The type of SELECT, which can be any of those shown in the following table. A JSON-formatted EXPLAIN exposes the SELECT type as a property of a query_block, unless it is SIMPLE or PRIMARY. The JSON names (where applicable) are also shown in the table.

| `select_type` Value                                                                                                            | JSON Name                    | Meaning                                                                                                                                                                                                                                  |
| ------------------------------------------------------------------------------------------------------------------------------ | ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `SIMPLE`                                                                                                                       | None                         | Simple [`SELECT`](https://dev.mysql.com/doc/refman/8.0/en/select.html "13.2.13 SELECT Statement") (not using [`UNION`](https://dev.mysql.com/doc/refman/8.0/en/union.html "13.2.18 UNION Clause") or subqueries)                         |
| `PRIMARY`                                                                                                                      | None                         | Outermost [`SELECT`](https://dev.mysql.com/doc/refman/8.0/en/select.html "13.2.13 SELECT Statement")                                                                                                                                     |
| [`UNION`](https://dev.mysql.com/doc/refman/8.0/en/union.html "13.2.18 UNION Clause")                                           | None                         | Second or later [`SELECT`](https://dev.mysql.com/doc/refman/8.0/en/select.html "13.2.13 SELECT Statement") statement in a [`UNION`](https://dev.mysql.com/doc/refman/8.0/en/union.html "13.2.18 UNION Clause")                           |
| `DEPENDENT UNION`                                                                                                              | `dependent` (`true`)         | Second or later [`SELECT`](https://dev.mysql.com/doc/refman/8.0/en/select.html "13.2.13 SELECT Statement") statement in a [`UNION`](https://dev.mysql.com/doc/refman/8.0/en/union.html "13.2.18 UNION Clause"), dependent on outer query |
| `UNION RESULT`                                                                                                                 | `union_result`               | Result of a [`UNION`](https://dev.mysql.com/doc/refman/8.0/en/union.html "13.2.18 UNION Clause").                                                                                                                                        |
| [`SUBQUERY`](https://dev.mysql.com/doc/refman/8.0/en/optimizer-hints.html#optimizer-hints-subquery "Subquery Optimizer Hints") | None                         | First [`SELECT`](https://dev.mysql.com/doc/refman/8.0/en/select.html "13.2.13 SELECT Statement") in subquery                                                                                                                             |
| `DEPENDENT SUBQUERY`                                                                                                           | `dependent` (`true`)         | First [`SELECT`](https://dev.mysql.com/doc/refman/8.0/en/select.html "13.2.13 SELECT Statement") in subquery, dependent on outer query                                                                                                   |
| `DERIVED`                                                                                                                      | None                         | Derived table                                                                                                                                                                                                                            |
| `DEPENDENT DERIVED`                                                                                                            | `dependent` (`true`)         | Derived table dependent on another table                                                                                                                                                                                                 |
| `MATERIALIZED`                                                                                                                 | `materialized_from_subquery` | Materialized subquery                                                                                                                                                                                                                    |
| `UNCACHEABLE SUBQUERY`                                                                                                         | `cacheable` (`false`)        | A subquery for which the result cannot be cached and must be re-evaluated for each row of the outer query                                                                                                                                |
| `UNCACHEABLE UNION`                                                                                                            | `cacheable` (`false`)        | The second or later select in a [`UNION`](https://dev.mysql.com/doc/refman/8.0/en/union.html "13.2.18 UNION Clause") that belongs to an uncacheable subquery (see `UNCACHEABLE SUBQUERY`)                                                |



### table (JSON name: table_name)

The name of the table to which the row of output refers.



### partitions (JSON name: partitions)

The partitions from which records would be matched by the query. The value is NULL for nonpartitioned tables. See Section 24.3.5, “Obtaining Information About Partitions”.



### type (JSON name: access_type)

The join type. For descriptions of the different types, see EXPLAIN Join Types.



### key_len (JSON name: key_length)

The key_len column indicates the length of the key that MySQL decided to use. The value of key_len enables you to determine how many parts of a multiple-part key MySQL actually uses. If the key column says NULL, the key_len column also says NULL.



### ref (JSON name: ref)

The `ref` column shows which columns or constants are compared to the index named in the key column to select rows from the table.



### rows (JSON name: rows)

The rows column indicates the number of rows MySQL believes it must examine to execute the query.

For InnoDB tables, this number is an estimate, and may not always be exact.



### filtered (JSON name: filtered)

The filtered column indicates an estimated percentage of table rows that are filtered by the table condition. The maximum value is 100, which means no filtering of rows occurred. Values decreasing from 100 indicate increasing amounts of filtering. rows shows the estimated number of rows examined and rows × filtered shows the number of rows that are joined with the following table. For example, if rows is 1000 and filtered is 50.00 (50%), the number of rows to be joined with the following table is 1000 × 50% = 500.



### Extra (JSON name: none)

This column contains additional information about how MySQL resolves the query. For descriptions of the different values, see EXPLAIN Extra Information.

There is no single JSON property corresponding to the Extra column; however, values that can occur in this column are exposed as JSON properties, or as the text of the message property.



## EXPLAIN Join Types

### system

The table has only one row (= system table). This is a special case of the const join type.

### const

The table has at most one matching row, which is read at the start of the query. Because there is only one row, values from the column in this row can be regarded as constants by the rest of the optimizer. const tables are very fast because they are read only once.

## eq_ref

One row is read from this table for each combination of rows from the previous tables. Other than the system and const types, this is the best possible join type. It is used when all parts of an index are used by the join and the index is a PRIMARY KEY or UNIQUE NOT NULL index.

eq_ref can be used for indexed columns that are compared using the = operator. The comparison value can be a constant or an expression that uses columns from tables that are read before this table. 

### ref

All rows with matching index values are read from this table for each combination of rows from the previous tables. ref is used if the join uses only a leftmost prefix of the key or if the key is not a PRIMARY KEY or UNIQUE index (in other words, if the join cannot select a single row based on the key value). If the key that is used matches only a few rows, this is a good join type.

ref can be used for indexed columns that are compared using the = or <=> operator. 



### index_merge

This join type indicates that the Index Merge optimization is used. In this case, the key column in the output row contains a list of indexes used, and key_len contains a list of the longest key parts for the indexes used. For more information, see Section 8.2.1.3, “Index Merge Optimization”.    

### range

Only rows that are in a given range are retrieved, using an index to select the rows. The key column in the output row indicates which index is used. The key_len contains the longest key part that was used. The ref column is NULL for this type.

range can be used when a key column is compared to a constant using any of the =, <>, >, >=, <, <=, IS NULL, <=>, BETWEEN, LIKE, or IN() operators

### index

The index join type is the same as ALL, except that the index tree is scanned. This occurs two ways:

If the index is a covering index for the queries and can be used to satisfy all data required from the table, only the index tree is scanned. In this case, the Extra column says Using index. An index-only scan usually is faster than ALL because the size of the index usually is smaller than the table data.

A full table scan is performed using reads from the index to look up data rows in index order. Uses index does not appear in the Extra column.

MySQL can use this join type when the query uses only columns that are part of a single index.

### ALL

A full table scan is done for each combination of rows from the previous tables. This is normally not good if the table is the first table not marked const, and usually very bad in all other cases. Normally, you can avoid ALL by adding indexes that enable row retrieval from the table based on constant values or column values from earlier tables.

### fulltext

The join is performed using a FULLTEXT index.

### ref_or_null

This join type is like ref, but with the addition that MySQL does an extra search for rows that contain NULL values. This join type optimization is used most often in resolving subqueries.

## Reference

[MySQL :: MySQL 8.0 Reference Manual :: 8.8.2 EXPLAIN Output Format](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html)
