---
title: Scraping Photo Metadata
layout: post
date: 2015-05-29
summary: Scraping photo exif data using node js for web mapping
categories: data scraping, node js, web-mapping
---

I ended up finding a Node JS library that worked pretty well called Exif. I'm basically cleaning up a mess by a bunch of students at the Urban Ecologies program. Back in the fall of 2014 they took photos of vacant lots, abandoned buildings, new construction etc in Bushwick for a participatory mapping survey but did a horrible horrible job storing the data and photos. 

I'm the one stuck with actually making sense of and using the data for the Bushwick Community Map. They sent me a KML file with 700 features for roughly 1000+ photos which is wack. Not only that but the file names of the photos in the KML file were some how changed so there is no way to link to the photos, at least that I'm aware of.

Anyhow I was able to scrape the lat lon from the Exif data for like 99% of the photos and make a Geojson that also has urls for the photos I uploaded to Flickr. I grabbed the urls with the Flickr API then used the Node joiner module (thanks to you for pointing that one out to me) to create the final GeoJSON. 

## Notes:
extract file name substring without the extension [reference](http://stackoverflow.com/questions/624870/regex-get-filename-without-extension-in-one-shot)  

	SELECT substring(file_name_column, '(.+?)(\.[^.]*$|$)') FROM table_name;
