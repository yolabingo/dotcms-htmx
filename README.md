# dotcms-htmx
Using dotCMS to manage 

## Overview

## Running DotCMS with demo site content

**tl;dr;**

On current OS X with Docker Desktop
```
curl -o docker-compose.yml https://raw.githubusercontent.com/dotCMS/core/master/docker/docker-compose-examples/single-node-demo-site/docker-compose.yml
docker compose up -d 
```
browse to 

[http://localhost:8080](http://localhost:8080/dotAdmin/) or 
[https://localhost:8443](https://localhost:8443/dotAdmin) (and accept the SSL cert)

Log in with:
```
username: admin@dotcms.com
password admin
```

## Create Site Search Index
[DotCMS Site Search Docs](https://dotcms.com/docs/latest/site-search)

DotCMS requires a Site Search index to support front-end searches. To create a Site Search index, log into the backend:
Dev Tools -> Site Search -> Job Scheduler, then
```
Run = Now
Select Sites to Index = "Index All Sites"
Alias Name = default
Language = English, Espanol
```
Click EXECUTE 

The "View All Jobs" tab shows the progress. It should take only a couple minutes for dotCMS to crawl the front-end pages and create an index.

Once complete, the "Indices" tab should show a healthy index.


## Simplify /search/ page


## Create /search/results "partial" Page

## Add HTMX to footer

## Refactor /search/ form to us HTMX Active Search

## Refactor Pagination to use HTMX Infinite Scroll