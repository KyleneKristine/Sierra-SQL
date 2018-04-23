SIERRA-SQL
======
A collection of SQL queries I have found useful for Sierra Direct. :+1:
<br>
<br> 
<br>
<br>

## Bib Record Range
#### Code
```sql
SELECT MIN(bib_view.record_num) as StartingBib, MAX(bib_view.record_num) as EndingBib
FROM sierra_view.bib_view;
```

#### Result
startingbib integer | endingbib integer
--------------------|------------------
1000001|8181155
<br> 

## More Than One Field

###### Control Fields
```sql
SELECT record_metadata.record_type_code || record_metadata.record_num || 'a' as BibNumber
FROM sierra_view.record_metadata, sierra_view.control_field
WHERE control_field.control_num = '7'
AND control_field.record_id = record_metadata.id
GROUP BY BibNumber HAVING COUNT(*) > 1;
```

###### Variable Fields
```sql
SELECT record_metadata.record_type_code || record_metadata.record_num || 'a' as BibNumber
FROM sierra_view.record_metadata, sierra_view.varfield
WHERE varfield.marc_tag = '245'
AND varfield.record_id = record_metadata.id
GROUP BY BibNumber HAVING COUNT(*) > 1;
```


