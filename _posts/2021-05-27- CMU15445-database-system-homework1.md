---
layout: post
title: "CMU 15-445/645 - Homework assignment #1 SQL"
description: "doing 15445 #1 homework - sql query"
date: 2021-05-27
tags: [cmu, database]
categories: [cmu]
comments: true
---

* table of contents
{:toc .toc}

## 0x00. Desciption

execute sql queies in SQLite in IMDB database.  

## 0x01. Evn setup

I'm using Ubuntu20.04 VM for this homework. The VM is installed on VMware Workstation Pro 15. There're several steps to prepare:  

1. install SQLite, `sudo apt-get install sqlite3 libsqlite3-dev`

2. Download dataset, `wget https://15445.courses.cs.cmu.edu/fall2019/files/imdb-cmudb2019.db.gz`

3. verify dataset, `md5 imdb-cmudb2019.db.gz`

4. Unzip and execute it, `gunzip imdb-cmudb2019.db.gz` and `sqlite3 imdb-cmudb2019.db`

## 0x02. Q1_SAMPLE

Sample, skip.

## 0x03. Q2_UNCOMMON_TYPE

List the longest title of each type along with the runtime minutes.  

```sql
select type, primary_title, max(runtime_minutes) FROM titles GROUP BY type ORDER BY type ASC, primary_title ASC;
```

output:  

```output
movie|Logistics|51420
short|Kuriocity|461
tvEpisode|Téléthon 2012|1800
tvMiniSeries|Kôya no yôjinbô|1755
tvMovie|ArtQuench Presents Spirit Art|2112
tvSeries|The Sharing Circle|8400
tvShort|Paul McCartney Backstage at Super Bowl XXXIX|60
tvSpecial|Katy Perry Live: Witness World Wide|5760
video|Midnight Movie Madness: 50 Movie Mega Pack|5135
videoGame|Flushy Fish VR: Just Squidding Around|1500
```

This is almost correct, however, in `tvShort` type, a tie exeist.  

```sql
sqlite> select title_id,type, runtime_minutes FROM titles WHERE type = 'tvShort'  ORDER BY runtime_minutes DESC LIMIT 5;
tt2292857|tvShort|60
tt5613498|tvShort|60
tt10622020|tvShort|53
tt0353158|tvShort|49
tt5452108|tvShort|48
```

Now the problem is the tie, I need to figure out a way to handle it(show both tuples when tie). So, I take the output of this sql as a temp table.

```sql
select type, primary_title, max(runtime_minutes) FROM titles GROUP BY type ORDER BY type ASC, primary_title ASC;
```

And then, join it with `titles` table, on the same runtime_minutes and type, which means it will give me:  

```text
if max without tie, the join will output the same tuple,
if max WITH tie, the join will output the tuples that has the same attributes* like the max, in this case, has the same runtime and type, which indicates it's a tie.
```

As for the answer:

```sql
select  titles.type, titles.primary_title, titles.runtime_minutes from titles 
join (select type, primary_title, max(runtime_minutes) as maxLength FROM titles GROUP BY type) as maxType
on maxType.maxLength = titles.runtime_minutes and maxType.type = titles.type
order by titles.type ASC, titles.primary_title ASC;
```

```output
movie|Logistics|51420
short|Kuriocity|461
tvEpisode|Téléthon 2012|1800
tvMiniSeries|Kôya no yôjinbô|1755
tvMovie|ArtQuench Presents Spirit Art|2112
tvSeries|The Sharing Circle|8400
tvShort|Paul McCartney Backstage at Super Bowl XXXIX|60
tvShort|The People Next Door|60
tvSpecial|Katy Perry Live: Witness World Wide|5760
video|Midnight Movie Madness: 50 Movie Mega Pack|5135
videoGame|Flushy Fish VR: Just Squidding Around|1500
```

## 0x04. Q3_TV_VS_MOVIE

List all types of titles along with the number of associated titles.

```sql
SELECT type, count(distinct title_id) as number from titles GROUP BY type ORDER BY number;
```

```output
tvShort|4075
videoGame|9044
tvSpecial|9107
tvMiniSeries|10291
tvMovie|45431
tvSeries|63631
video|90069
movie|197957
short|262038
tvEpisode|1603076
```

Answer from prof:

```sql
SELECT type, count(*) AS title_count FROM titles GROUP BY type ORDER BY title_count ASC;
```

So, at this time that I figured title_id is primary key and it's unique, so distinct is redundant. It could be replaced by `count(title_id)` or simplily `count(*)`.

## 0x05. Q4_OLD_IS_NOT_GOLD

Which decades saw the most number of titles getting premiered? List the number of titles in every decade. Like `2010s|2789741`.

From what I could do, I got this:

```sql
SELECT CAST(premiered/10 AS INT)*10 as upTime, count(*) FROM titles WHERE premiered is not NULL GROUP BY upTime ORDER BY upTime DESC;
```

```output
2020|2492
2010|1050732
2000|494639
1990|211453
1980|119258
1970|99707
1960|75237
1950|39554
1940|10011
1930|11492
1920|13153
1910|26596
1900|9586
1890|2286
1880|22
1870|1
```

I have no idea how to append 's' at the end of each decade. So, I look for the answer.

```sql
SELECT 
  CAST(premiered/10*10 AS TEXT) || 's' AS decade,
  COUNT(*) AS num_movies
  FROM titles
  WHERE premiered is not null
  GROUP BY decade
  ORDER BY num_movies DESC
  ;
```

```output
2010s|1050732
2000s|494639
1990s|211453
1980s|119258
1970s|99707
1960s|75237
1950s|39554
1910s|26596
1920s|13153
1930s|11492
1940s|10011
1900s|9586
2020s|2492
1890s|2286
1880s|22
1870s|1
```

I cast the decades into `INT` instead of `TEXT` which could not use `||` to append someting else. And sort this by number of movies(I sorted by the wrong collum before) Now I change my sql to this:

```sql
SELECT CAST(premiered/10*10 AS TEXT) || 's' as upTime, count(*) as numbers FROM titles WHERE premiered is not NULL GROUP BY upTime ORDER BY numbers DESC;
```

```output
2010s|1050732
2000s|494639
1990s|211453
1980s|119258
1970s|99707
1960s|75237
1950s|39554
1910s|26596
1920s|13153
1930s|11492
1940s|10011
1900s|9586
2020s|2492
1890s|2286
1880s|22
1870s|1
```

And now, the output is exactly the same.

## 0x06. Q5_PERCENTAGE

List the decades and the percentage of titles which premiered in the corresponding decade. Display like : `2010s|45.7042`.

```sql
SELECT CAST(premiered/10*10 AS TEXT) || 's' as upTime, ROUND(CAST(count(*) AS FLOAD) / (SELECT COUNT(*) FROM titles) * 100, 4) as numbers 
FROM titles WHERE premiered is not NULL GROUP BY upTime ORDER BY numbers DESC;
```

```output
2010s|45.7891
2000s|21.5555
1990s|9.2148
1980s|5.1971
1970s|4.3451
1960s|3.2787
1950s|1.7237
1910s|1.159
1920s|0.5732
1930s|0.5008
1940s|0.4363
1900s|0.4177
2020s|0.1086
1890s|0.0996
1880s|0.001
1870s|0.0
```

This is done based on the previous question, just cast the number to fload and divide. The answer cast it into real. They are both proximity data types, so in this question, there no difference. 

If you need accurate data, `numeric` and `decimal` should be better.

