SIERRA-SQL
======
A collection of SQL queries I have found useful for Sierra Direct. :+1:

<br>

![Coding Cat](https://camo.githubusercontent.com/a8eac700157bdf9da9a404855c5d91a68c59b360/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6669742f742f323430302f313030382f302a6e2d32625738325a366d36553262696a2e6a706567) 
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
AND NOT varfield.field_content like '%https://login.revproxy.brown.edu/login?url=%'
AND leader_field.record_type_code = 'm'
AND varfield.record_id = record_metadata.id
AND leader_field.record_id = record_metadata.id;
```
<br>

## Transactions by Date
Returns Item Numbers of items that circulated during specified time frame.
```sql
SELECT record_metadata.record_type_code || record_metadata.record_num || 'a' as ItemNumber, 
  circ_trans.transaction_gmt as TransTime
FROM sierra_view.record_metadata, sierra_view.circ_trans
WHERE circ_trans.item_record_id = record_metadata.id
AND circ_trans.transaction_gmt BETWEEN '2018/04/09' AND '2018/04/22' --enter date range
GROUP BY ItemNumber, TransTime;
```
###### By Patron Code
```sql
SELECT record_metadata.record_type_code || record_metadata.record_num || 'a' as ItemNumber, 
  circ_trans.ptype_code as Ptype, circ_trans.transaction_gmt as TransTime
FROM sierra_view.record_metadata, sierra_view.circ_trans
WHERE circ_trans.item_record_id = record_metadata.id
AND circ_trans.ptype_code = '21' -- enter ptype
AND circ_trans.transaction_gmt BETWEEN '2018/04/09' AND '2018/04/22' --enter date range
GROUP BY Ptype, ItemNumber, TransTime
ORDER BY Ptype;
```
<br>

## Records with No Titles
Returns BIB numbers of BIB records that have and empty or no 245.
```sql
SELECT bib_view.record_type_code || bib_view.record_num || 'a' as BibNumber
FROM sierra_view.bib_view
WHERE bib_view.title is NULL;
```
<br>

## Deleted Records
Returns Record Numbers that have been deleted and their deletion date
```sql
SELECT record_metadata.record_type_code || record_metadata.record_num || 'a' as RecordNumber,
	record_metadata.deletion_date_gmt as DeletionDate
FROM sierra_view.record_metadata
WHERE record_metadata.deletion_date_gmt is not NULL
AND record_metadata.record_type_code = 'b'; --enter record code
```
<br>

## Number of Records Deleted
Returns the number of records deleted on a specific day
```sql
SELECT record_metadata.deletion_date_gmt as DeletionDate, 
  COUNT(record_metadata.deletion_date_gmt) as DeletedCount
FROM sierra_view.record_metadata
WHERE record_metadata.deletion_date_gmt BETWEEN '2018/01/01' AND '2018/05/01' --enter date range
GROUP BY DeletionDate
ORDER BY DeletionDate;
```
<br>

## Distinct Tag Counts
Query pulls unique tags used in the 71020 field and counts the number of records using the tag
```sql
SELECT distinct(varfield.field_content) as FContent, COUNT(varfield.field_content)
FROM sierra_view.varfield
WHERE varfield.marc_tag = '710' --enter marc tag
AND varfield.marc_ind1 = '2' --enter first indicator
AND varfield.marc_ind2 = '0' --enter second indicator
AND varfield.field_content like '%5RPB%' --use to search specific subfields, enter between %s
GROUP BY FContent
ORDER BY FContent ASC;
```
<br>

## Review List Word Count
Query counts the number of records that contain a word.
```sql
SELECT count(distinct(varfield_view.record_id)) as FContent
FROM sierra_view.varfield_view, sierra_view.bool_set
WHERE bool_set.bool_info_id = '257' --enter Review File Number
AND varfield_view.field_content like '%data%' --enter word between %s
AND bool_set.record_metadata_id = varfield_view.record_id;
```
<br>

## Broken BIB Locations
Query looks for BIBs with incorrect codes in the location field.
```sql
SELECT bib_view.record_type_code || bib_view.record_num || 'a' as BibNumber
FROM sierra_view.bib_view, sierra_view.bib_record_location
WHERE NOT bib_record_location.location_code like '%001%' --removes correct bib location codes
AND NOT bib_record_location.location_code like '%multi%' --Not sure how to get around this. If there is more than one bib location it only appears as multi
AND NOT bib_record_location.location_code like '%nnnnn%' --removes in-process
AND NOT bib_record_location.location_code like '%fffff%' --removes on the fly
AND NOT bib_record_location.location_code like '%xxxxx%' --removes reserves
AND bib_record_location.bib_record_id = bib_view.id
AND bib_view.record_type_code = 'b';
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
<br>

## Flags Table
Pulls bib and item information on records contained in a specific list. We then use CSS to arrange the pulled table into a printable flag format. Pulls Order Numbers, Call number, copy number, barcode, icode2, item status code, bib record number, item record number, 910 tags containing Hathi (but only the last 3 characters e.g. 'SPM'), title, edition, publisher, description, local notes, volume, and finally whether the item record is bound with more than one bib record. 
```sql
DROP TABLE IF EXISTS temp_item_data;
DROP TABLE IF EXISTS temp_dupe;
CREATE TEMP TABLE temp_item_data AS
SELECT
bo.display_order,
(SELECT o.record_type_code || o.record_num || 'a'
 FROM sierra_view.order_view as o, sierra_view.bib_record_order_record_link as ol
 WHERE o.id = ol.order_record_id AND ol.bib_record_id=l.bib_record_id
LIMIT 1) as ordernum,
p.call_number_norm,
i.copy_num,
(SELECT 
 	vv.field_content 
 FROM 
 	sierra_view.varfield as vv 
 WHERE 
 	vv.record_id = l.item_record_id 
 	AND vv.varfield_type_code = 'v' 
 LIMIT 1) as volume,
p.barcode,
i.icode2,
i2.item_status_code,
rb.record_type_code || rb.record_num || 'a' as bib_record_num,
ri.record_type_code || ri.record_num || 'a' as item_record_num,
(SELECT
	regexp_replace(trim(v.field_content), '(\|[a-z]{1}Hathi Trust Report)', '', 'ig')
	FROM
	sierra_view.varfield as v
	WHERE
	v.record_id = l.bib_record_id
	AND v.marc_tag='910'
  	AND v.field_content like '%Hathi%'
	ORDER BY
	v.occ_num
	LIMIT 1) as localtag,
(SELECT
	(sv.content)
	FROM
	sierra_view.subfield_view as sv
	WHERE
	sv.record_id = l.bib_record_id
	AND sv.marc_tag='245'
 	AND sv.tag='a'
	ORDER BY
	sv.occ_num
	LIMIT 1) as title,
(SELECT
	regexp_replace(trim(v.field_content), '(\|[a-z]{1})', '', 'ig')
	FROM
	sierra_view.varfield as v
	WHERE
	v.record_id = l.bib_record_id
	AND v.marc_tag='250'
	ORDER BY
	v.occ_num
	LIMIT 1) as edition,
(SELECT
	regexp_replace(trim(v.field_content), '(\|[a-z]{1})', '', 'ig')
	FROM
	sierra_view.varfield as v
	WHERE
	v.record_id = l.bib_record_id
	AND v.marc_tag='260'
	ORDER BY
	v.occ_num
	LIMIT 1) as publisher,
(SELECT
	regexp_replace(trim(v.field_content), '(\|[a-z]{1})', '', 'ig')
	FROM
	sierra_view.varfield as v
	WHERE
	v.record_id = l.bib_record_id
	AND v.marc_tag='300'
	ORDER BY
	v.occ_num
	LIMIT 1) as description,
i.location_code iloc,
(SELECT
	regexp_replace(trim(v.field_content), '(\|[a-z]{1})', '', 'ig')
	FROM
	sierra_view.varfield as v
	WHERE
	v.record_id = l.bib_record_id
	AND v.marc_tag='590'
	ORDER BY
	v.occ_num
	LIMIT 1) as localnotes

FROM sierra_view.item_record as i

JOIN sierra_view.bib_record_item_record_link as l
ON (l.item_record_id = i.record_id)

JOIN sierra_view.item_view as i2
ON (i2.id=i.record_id)

JOIN sierra_view.item_record_property as p
ON (p.item_record_id=i.record_id)

JOIN sierra_view.bool_set as bo
ON (bo.record_metadata_id=i.record_id)

JOIN sierra_view.record_metadata as ri
ON (ri.id = i.record_id)

JOIN sierra_view.record_metadata as rb
ON (rb.id = l.bib_record_id) AND (rb.campus_code = '')

WHERE bo.bool_info_id=171; --Change number to whatever list you are using in Sierra

CREATE TEMP TABLE temp_dupe AS
SELECT
count(l.bib_record_id)>1 as BNDWITH, bo.display_order
FROM
sierra_view.bib_record_item_record_link as l
JOIN sierra_view.bool_set as bo ON (bo.record_metadata_id=l.item_record_id)
WHERE bo.bool_info_id=171 --Change number to whatever list you are using in Sierra
GROUP BY l.item_record_id, bo.display_order;

SELECT
t.*, du.BNDWITH
FROM
temp_item_data as t
JOIN temp_dupe as du ON (du.display_order = t.display_order)
ORDER BY t.display_order;
```
## Duplicate DDAs
We have several DDA plans from different vendors. Normally we use Sierra's heading reports but we get a lot of matches for physical and electronic records which eats up a lot of time investigating. This allows me to pull only records from our JSTOR and GOBI DDAs and look for duplicate isbns to investigate.
```sql
DROP TABLE IF EXISTS DDAs;
CREATE TEMP TABLE DDAs AS
SELECT
pe.index_entry as ocns, pe.record_id, z.index_entry as isbns, z.id, z.index_tag
FROM
sierra_view.phrase_entry as pe, sierra_view.phrase_entry as z
WHERE pe.index_tag='o' AND z.record_id=pe.record_id AND z.index_tag='i' AND (pe.index_entry LIKE '%jstor' OR pe.index_entry LIKE 'ybp%');

SELECT
d.record_id, d.ocns, d.isbns, d.id
FROM
DDAs as d
WHERE d.isbns IN (SELECT d.isbns FROM DDAs as d WHERE d.index_tag='i' GROUP BY d.isbns HAVING count(d.id)>1)
ORDER BY d.isbns
```
## Mismatching Call numbers between Serials and Mono Series
We noticed that our bibs for serials and the bibs for individual monographs in those series sometimes didn't match up or were slightly off. This code looks for Serials and Monographs that share an item record but have different call numbers.
```sql
DROP TABLE IF EXISTS LCS;
DROP TABLE IF EXISTS LCM;

CREATE TEMP TABLE LCS AS
SELECT 'b' || VVS.record_num || 'a' as serial_num, VVS.field_content as serial_call, VVS.record_id as serialid, BWD.item_record_id as serial_item_id
FROM sierra_view.varfield_view as VVS JOIN sierra_view.bib_record_item_record_link as BWD ON BWD.bib_record_id=VVS.record_id
WHERE BWD.item_record_id IN (SELECT BWD.item_record_id as serial_id
FROM sierra_view.leader_field as LF JOIN sierra_view.bib_record_item_record_link as BWD ON BWD.bib_record_id=LF.record_id
WHERE LF.bib_level_code = 's' AND BWD.item_record_id IN (SELECT BL.item_record_id
FROM sierra_view.bib_record_item_record_link as BL
GROUP BY BL.item_record_id
HAVING count(BL.bib_record_id)>1))
AND VVS.marc_tag='090';

CREATE TEMP TABLE LCM AS
SELECT 'b' || VVS.record_num || 'a' as mono_num, VVS.field_content as mono_call, VVS.record_id as monoid, BWD.item_record_id as mono_item_id
FROM sierra_view.varfield_view as VVS JOIN sierra_view.bib_record_item_record_link as BWD ON BWD.bib_record_id=VVS.record_id
WHERE BWD.item_record_id IN (SELECT BWD.item_record_id as serial_id
FROM sierra_view.leader_field as LF JOIN sierra_view.bib_record_item_record_link as BWD ON BWD.bib_record_id=LF.record_id
WHERE LF.bib_level_code = 'm' AND BWD.item_record_id IN (SELECT BL.item_record_id
FROM sierra_view.bib_record_item_record_link as BL
GROUP BY BL.item_record_id
HAVING count(BL.bib_record_id)>1))
AND VVS.marc_tag='090';

SELECT LCS.serial_num, LCS.serial_call, LCM.mono_num, LCM.mono_call
FROM LCS JOIN LCM ON LCM.mono_item_id = LCS.serial_item_id
WHERE LCS.serial_call != LCM.mono_call
```
