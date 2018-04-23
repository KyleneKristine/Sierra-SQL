SIERRA-SQL
======
A collection of SQL expressions I have found useful for Sierra Direct. :+1:


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
