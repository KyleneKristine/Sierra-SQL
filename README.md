SIERRA-SQL
======
A collection of SQL queries I have found useful for Sierra Direct. :+1:
<br>
<br> 
<br>
<br>

## Bib Record Range
Returns the current beginning and ending BIB Numbers.
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
Returns BIB Numbers that have more than one field in the record.
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
<br>

## Electronic Records without a Proxy
Returns BIB Numbers of electonic records that do not have the Brown Proxy url in the 856 field.
```sql
SELECT record_metadata.record_type_code || record_metadata.record_num || 'a' as BibNumber
FROM sierra_view.record_metadata, sierra_view.varfield, sierra_view.leader_field
WHERE varfield.marc_tag = '856' 
AND varfield.field_content not like '%https://login.revproxy.brown.edu/login?url=%'
AND leader_field.record_type_code = 'm'
AND varfield.record_id = record_metadata.id
AND leader_field.record_id = record_metadata.id
```
<br>

## Bib Location
Returns BIB Numbers of records whose BIB Location is not set to a BIB Location code.
```sql

```
