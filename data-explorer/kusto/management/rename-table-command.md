---
title: .rename table and .rename tables - Azure Data Explorer
description: This article describes .rename table and .rename tables in Azure Data Explorer.
ms.reviewer: orspodek
ms.topic: reference
ms.date: 02/04/2020
---
# .rename table and .rename tables

Changes the name of an existing table.

The `.rename tables` command changes the name of a number of tables in the database as a single transaction.

Requires [Database  admin permission](../management/access-control/role-based-authorization.md).

**Syntax**

`.rename` `table` *OldName* `to` *NewName*

`.rename` `tables` *NewName* = *OldName* [`ifexists`] [`,` ...]

> [!NOTE]
> * *OldName* is the name of an existing table. An error is raised and
  the whole command fails (has no effect) if *OldName* does not name
  an existing table, unless `ifexists` is specified (in which case
  this part of the rename command is ignored).
> * *NewName* is the new name of the existing table that used to be called
  *OldName*.
> * If `ifexists` is specified, it modifies the behavior of the command to
  ignore renaming parts of non-existent tables.

**Remarks**

This command operates on tables of the database in scope only.
Table names cannot be qualified with cluster or database names.

This command doesn't create new tables, nor does it remove existing tables.
The transformation described by the command must be such that the number
of tables in the database does not change.

The command **does** support swapping table names, or more complex
permutations, as long as they adhere to the rules above. For example, ingest data into multiple staging tables,
and then swap them with existing tables in a single transaction.

**Examples**

Imagine a database with the following tables: `A`, `B`, `C`, and `A_TEMP`.
The following command will swap `A` and `A_TEMP` (so that the `A_TEMP` table will now be called `A`, and the other way around), rename
`B` to `NEWB`, and keep `C` as-is. 

```kusto
.rename tables A=A_TEMP, NEWB=B, A_TEMP=A
``` 

The following sequence of commands:
1. Creates a new temporary table
1. Replaces an existing or non-existing table with the new table

```kusto
// Drop the temporary table if it exists
.drop table TempTable ifexists

// Create a new table
.set TempTable <| ...

// Swap the two tables
.rename tables TempTable=Table ifexists, Table=TempTable

// Drop the temporary table (which used to be Table) if it exists
.drop table TempTable ifexists
```

**Rename source table of a materialized view**

If the table being renamed is the source table of a [materialized view](materialized-views/materialized-view-overview.md), you can specify the following property as part of the `.rename` command:

`.rename` `table` *OldName* `to` *NewName* `with (updateMaterializedViews=true)`

The table will be renamed and all materialized views referencing *OldName* will be updated to point to *NewName*, in a transactional way.

> [!NOTE]
> The command will only work if the source table is referenced directly in the materialized view query. If the source table is referenced from a stored function invoked by the view query, the command will fail, since the command cannot update the stored function.
