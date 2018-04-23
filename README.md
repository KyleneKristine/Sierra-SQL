Sierra-SQL
======

##Bib Record Range
...SELECT MIN(bib_view.record_num) as StartingBib, MAX(bib_view.record_num) as EndingBib..
...FROM sierra_view.bib_view..
...GROUP BY bib_view.record_num;

