```sql
SELECT owner, table_name, column_name, data_type
FROM all_tab_columns
WHERE owner = 'HR';
```