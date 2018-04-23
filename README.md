SIERRA-SQL
======
A collection of SQL queries I have found useful for Sierra Direct. :+1:
#
#
## Bib Record Range
#### Code
```
SELECT MIN(bib_view.record_num) as StartingBib, MAX(bib_view.record_num) as EndingBib
FROM sierra_view.bib_view;
```

#### Result
startingbib integer | endingbib integer
--------------------|------------------
1000001|8181155
#
#

## More Than One Field

###### Control Fields
```
SELECT bib_view.record_num as BibNumber
FROM sierra_view.bib_view, sierra_view.control_field
WHERE control_field.control_num = '7'
AND control_field.record_id = bib_view.id
GROUP BY BibNumber HAVING COUNT(*) > 1;
```

###### Variable Fields
```
SELECT bib_view.record_num as BibNumber
FROM sierra_view.bib_view, sierra_view.varfield
WHERE varfield.marc_tag = '245'
AND varfield.record_id = bib_view.id
GROUP BY BibNumber HAVING COUNT(*) > 1;
```
