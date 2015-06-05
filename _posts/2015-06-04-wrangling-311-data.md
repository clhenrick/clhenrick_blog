---
title: Wrangling NYC's 311 data
layout: post
date: 2015-06-03
summary: Battling the task of downloading 311 data and importing it into PostgreSQL for analysis.
categories: data postgresql open-data
---

I've been lucky to have the opportunity to work with the incredibly talented author [Rebecca Solnit](#) on her new NYC Atlas. Rebecca has previously completed atlases for two other cities, they're titled [Infinite City, a San Francisco Atlas](#); and [Unfathomable City, a New Orleans Atlas](#); and they are anything but traditional city planning maps. While Rebecca is not a cartographer by trade (she hires others to make the maps in her books) she has a deep appreciation and in-depth knowledge of the field and thus draws inspiration from the rich history of printed maps. Rebecca has added her own unique touch and brought life to the maps in her books; they range from being quirky, mystifying, poignant, politically charged, or celebratory. Her maps offer a refreshing perspective on map making in the current deluge of maps that are bound to the web mercator projection, algorithmically rendered, and at times just plain ugly.

As part of the map I'm working on for the NYC Atlas (I can't disclose its name yet!), Rebecca and her publishing partner [Joshua Jelly Schapiro](#) were very interested in looking into NYC's 311 data. This was an exciting opportunity for me to revisit the data, as it had been several years since I last used it when I made the [NYC Rats, Graffiti, and Wifi Hot Spots Map](#).

## Downloading 311 data from the NYC Open Data Portal
Because the 311 dataset is so massive, and Socrata's servers aren't necessarily the fastest, it's not too easy to pull this data down from [NYC Open Data](#). The solution I came up with was to filter the data on the open data portal so that I only had to download a subset of the data. I grabbed everything from January 1, 2014 to present, made a new "view", then downloaded that data which still ended up being a roughly 1.5 GB CSV file!

## Importing 311 Data into PostgreSQL
I prefer postgresql for a bunch of reasons, but probably the main is that the map / GIS geek in me really likes to use [PostGIS](#), which is an extension for PostgreSQL that allows for wrangling spatial data & geoprocessing.

### Problem: importing a 1.5 GB CSV file into postgresql
Typically I use csvkit's csvsql command line tool to import CSV data into Postgres. This normally works fairly well, but not in the case of a 1.5 GB CSV file. Basically, attempting to do this row by row *is not the answer!*  
The following Allen Iverson poster comes to mind:

![war is not the answer]({{site.url}}/assets/warisnottheanswer1.jpg)

What we need is a way to bulk load the data.

### Solution: PGloader 

PGloader is a tool that allows for programatically bulk loading of data into PostgreSQL. Using this method ended up taking a whole 5 minutes 2.127 seconds, instead of hanging up and eventually crashing my computer. Wow!

PGloader can be used one of two ways; by either writing a script or as a command line utility. Because I had yet to create the table in my `nyc_311` database, I decided to go with using a script. My friend [John Krauss](#) refered me to a pgloader script he used which helped me get going. The syntax for pgloader is PLpgSQL which is a bit strange when first encountered. 

Prior to writing the script I first wanted to pull out the 30 or so field names and make them database friendly. A quick n dirty bash script did the job:  

```
FILE_IN="311_requests_2014_to_present.csv"
FILE_OUT="311_field_list.txt"
head -n 1 $FILE_IN  | tr ',' '\n'| sed -e 's/ /_/g' | tr '[A-Z]' '[a-z]' > $FILE_OUT
```


### pgloader code:

```
LOAD CSV
  FROM '/Users/clhenrick/Downloads/311_requests_2014_to_present.csv'
  INTO postgresql:///nyc_311?requests
  WITH skip header = 1,
       fields terminated by ','
  BEFORE LOAD DO  
     $$ drop table if exists requests; $$,
     $$ create table if not exists requests (  
          unique_key bigint,
          created_date DATE,
          closed_date DATE,
          agency TEXT,
          agency_name TEXT,
          complaint_type TEXT,
          descriptor TEXT,
          location_type TEXT,
          incident_zip TEXT,
          incident_address TEXT,
          street_name TEXT,
          cross_street_1 TEXT,
          cross_street_2 TEXT,
          intersection_street_1 TEXT,
          intersection_street_2 TEXT,
          address_type TEXT,
          city TEXT,
          landmark TEXT,
          facility_type TEXT,
          status TEXT,
          due_date DATE,
          resolution_action_updated_date DATE,
          community_board TEXT,
          borough TEXT,
          x_coordinate bigint,
          y_coordinate bigint,
          park_facility_name TEXT,
          park_borough TEXT,
          school_name TEXT,
          school_number TEXT,
          school_region TEXT,
          school_code TEXT,
          school_phone_number TEXT,
          school_address TEXT,
          school_city TEXT,
          school_state TEXT,
          school_zip TEXT,
          school_not_found TEXT,
          school_or_citywide_complaint TEXT,
          vehicle_type TEXT,
          taxi_company_borough TEXT,
          taxi_pick_up_location TEXT,
          bridge_highway_name TEXT,
          bridge_highway_direction TEXT,
          road_ramp TEXT,
          bridge_highway_segment TEXT,
          garage_lot_name TEXT,
          ferry_direction TEXT,
          ferry_terminal_name TEXT,
          latitude NUMERIC,
          longitude NUMERIC,
          location TEXT
        );  
   $$; 

LOAD CSV
  FROM '/Users/clhenrick/Downloads/311_requests_2014_to_present.csv'
  INTO postgresql:///nyc_311?requests
  WITH skip header = 1,
       fields optionally enclosed by '"',  
       fields terminated by ','
;
```

## Data Analysis
Now that I had the data in Postgres, I could do some analysis (not before indexing the data, vacuum analyzing and clustering it).

Thus I was able to accomplish the following:

- list all the [distinct complaint types](#)
- pull out subsets of the data, such as [taxi complaints](#)
- geocode data that has latitude & longitude values (see the taxi complaint link above).

All in all, this was a fun time I'd say, and it certainly paid off to have the 311 data in postgres where I can now get all crazy with it.