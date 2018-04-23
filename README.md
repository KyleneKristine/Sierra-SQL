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
WHERE control_field.control_num = '7' --6, 7, or 8 which equal 006, 007, or 008 respectively.
AND control_field.record_id = record_metadata.id
GROUP BY BibNumber HAVING COUNT(*) > 1;
```

###### Variable Fields
```sql
SELECT record_metadata.record_type_code || record_metadata.record_num || 'a' as BibNumber
FROM sierra_view.record_metadata, sierra_view.varfield
WHERE varfield.marc_tag = '245' --enter marc tag
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

## Transactions by Date
Returns Item Numbers of items that circulated during specified time frame.
```sql
SELECT record_metadata.record_type_code || record_metadata.record_num || 'a' as BibNumber, 
  circ_trans.transaction_gmt as TransTime
FROM sierra_view.record_metadata, sierra_view.circ_trans
WHERE circ_trans.item_record_id = record_metadata.id
AND circ_trans.transaction_gmt BETWEEN '2018/04/09' AND '2018/04/22' --enter dates
GROUP BY BibNumber, TransTime
```
###### By Patron Code
```sql
SELECT record_metadata.record_type_code || record_metadata.record_num || 'a' as BibNumber, 
  circ_trans.ptype_code as Ptype, circ_trans.transaction_gmt as TransTime
FROM sierra_view.record_metadata, sierra_view.circ_trans
WHERE circ_trans.item_record_id = record_metadata.id
AND circ_trans.transaction_gmt BETWEEN '2018/04/09' AND '2018/04/22' --enter dates
GROUP BY Ptype, BibNumber, TransTime
ORDER BY Ptype
```
<br>

## Create Lists Difference
Query that compares two create lists and finds BIBs not found in both. Created by [UNC Library](https://github.com/UNC-Libraries/III-Sierra-SQL/wiki#diff-two-create-lists--review-files).
```sql
with lists as (select 222 as list1   --enter list numbers here
                     , 242 as list2   --enter list numbers here
      ),
      diff as (
        select bs.record_metadata_id
            , case when bs2.id is null then bs.bool_info_id::varchar
                   else 'both'
              end as appears
            , case when bs2.id is null then bsi.name
                   else 'both'
              end as name
        from sierra_view.bool_set bs
        cross join lists
        left join sierra_view.bool_set bs2 on bs2.record_metadata_id = bs.record_metadata_id
          and bs2.bool_info_id = lists.list2
        inner join sierra_view.bool_info bsi on bsi.id = lists.list1
        where bs.bool_info_id = lists.list1
        -- 
        UNION
        --
        select bs2.record_metadata_id
             , bs2.bool_info_id::varchar as appears
             , bsi.name
        from sierra_view.bool_set bs2
        cross join lists
        inner join sierra_view.bool_info bsi on bsi.id = lists.list2
        where bs2.bool_info_id = lists.list2
          and not exists (
            select *
            from sierra_view.bool_set
            where record_metadata_id = bs2.record_metadata_id
              and bool_info_id = lists.list1
          )
      ) --end diff
 select rm.record_type_code || rm.record_num || 'a' as rnum, diff.appears, diff.name
 from diff
 inner join sierra_view.record_metadata rm on rm.id = diff.record_metadata_id
 order by diff.appears
```
