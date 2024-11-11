---
title: "Browser Forensics: Firefox Bookmark Analysis"
datePublished: Mon Nov 11 2024 11:43:56 GMT+0000 (Coordinated Universal Time)
cuid: cm3cygolb001409l25d7o1b7z
slug: browser-forensics-firefox-bookmark-analysis
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1726305989974/1cb26bcc-7b53-4a23-8fb7-b654b96879f7.png
tags: firefox, cyberexam, browser-forensics

---

## Mission Statement

It is required that needs to be done Firefox data analysis manually from bookmarks data.

## Solution

Start the lab and access the machine. Mozilla.zip file can be found on desktop. Unzip the file and analyze.

### **How many bookmarks are there in the Firefox added by user?**

`places.sqlite` file includes bookmarks and history URL for Firefox browser. The directory of the file is `.mozilla/firefox/PROFILE.default-release/`under home directory commonly. But this scenerio, the directory and files are given to us.

`sqlite3 places.sqlite` run command to open the places.sqlite database. `.tables` command displays the tables of the database. Type `.help` to see other commands in a detail.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726302096812/67e0e34c-eeaa-437e-84a2-7cf63ad5d46e.png align="center")

`SELECT * FROM moz_bookmarks` command lists the bookmarked URLs

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1731323542329/2dfcc34c-0385-4610-80bc-f803c8be6f88.png align="center")

Run the command `PRAGMA table_info(<TABLE_NAME>);` or `.schema TABLE_NAME` to learn about the schema of the table.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726302926485/83286beb-24f3-4118-a997-d06a7c8bdc47.png align="center")

### **What is the URL of the shopping site added to Favorites by target user?**

To list Favorites sites with help of chatgpt for the SQL query, run the command

```sql
SELECT moz_places.url, moz_bookmarks.title, moz_places.description
FROM moz_bookmarks
JOIN moz_places ON moz_bookmarks.fk = moz_places.id
WHERE moz_bookmarks.title IS NOT NULL
ORDER BY moz_bookmarks.dateAdded DESC;
```

This query lists the URL and its title from the bookmarks and ordered by the added date. You can filter shopping keywords to search in description text or other column values.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1731325185943/076e9c9c-9368-4d2a-b3d4-ee5535cb1cab.png align="center")

### **When was the Youtube entry added DD-MM-YYYY?**

This command can be used to find the date of the added Youtube in bookmarks with format DD-MM-YYYY. moz\_places table includes history and bookmarks with url information, so there is a relation between moz\_places and moz\_bookmarks tables with column fk (foreign key). If controls with url, use this command

`SELECT moz_places.url, strftime('%d-%m-%Y', dateAdded/1000000, 'unixepoch') AS added_date FROM moz_bookmarks JOIN moz_places ON moz_bookmarks.fk = moz_places.id WHERE moz_places.url LIKE '%youtube%'`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726306134904/25b966b5-c49f-467e-9116-8f07ef5cf0eb.png align="center")

If condition which title contains youtube is checked with title value, use the command `SELECT title, strftime('%d-%m-%Y', dateAdded/1000000, 'unixepoch') AS added_date FROM moz_bookmarks WHERE title LIKE '%youtube%';`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726306193176/f2bdff7d-b1e8-4628-ae67-780bac569731.png align="center")

The column of dateAdded value represents the date in microsecond format. Linux `date` command can be used for the time conversion from microseconds to DD-MM-YYYY format with the help of ChatGPT, `date -d "@$(printf "%.0f" $(awk 'BEGIN {print MICROSECONDS/ 1000000}'))" +"%d-%m-%Y"` . Note that MICROSECONDS value will be the dateAdded value in the table.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726307016246/220f09b4-e749-43af-80fa-025b40faa6c6.png align="center")