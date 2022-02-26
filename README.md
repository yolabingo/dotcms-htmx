# dotcms-htmx
Using dotCMS to manage 

## Overview

You will want two browser windows open for this - one logged into the DotCMS backend and one not logged in.

## Running DotCMS with demo site content

**tl;dr;**

On current OS X with Docker Desktop
```
curl -o docker-compose.yml https://raw.githubusercontent.com/dotCMS/core/master/docker/docker-compose-examples/single-node-demo-site/docker-compose.yml
docker compose up -d 
```
browse to 

[http://localhost:8080/dotAdmin/](http://localhost:8080/dotAdmin/) or 

[https://localhost:8443/dotAdmin/](https://localhost:8443/dotAdmin) (and accept the SSL cert)

Log in with:
```
username: admin@dotcms.com
password admin
```

DotCMS provides a "demo" site to allow you to test and explore DotCMS features. Run DotCMS locally using the `single-node-demo-site` from our [docker compose examples](https://github.com/dotCMS/core/tree/master/docker/docker-compose-examples)

 A version of this site is always publically available at [https://demo.dotcms.com](https://demo.dotcms.com) but it is refreshed multiple times/day and not a 

## Create Site Search Index
[DotCMS Site Search Docs](https://dotcms.com/docs/latest/site-search)

DotCMS requires a Site Search index to support searches from front-end sites. To create a Site Search index, log into the dotCMS backend at `/dotAdmin/`:
Dev Tools -> Site Search -> Job Scheduler and create a job:
```
Run = Now
Select Sites to Index = Index All Sites
Alias Name = default
Language = English, Espanol
```
Click EXECUTE 

The "View All Jobs" tab shows the progress. It should take only a couple minutes for dotCMS to crawl the front-end pages and create an index.

Once complete, the "Indices" tab should show a healthy index.

## Add HTMX script to footer

In DotCMS backend go to
Site -> Browser -> application -> themes -> travel

Double-click `footer.vtl` - scroll to the bottom of file editor and add HTMX to the Javascript section:
```
<!-- Javascript-->
<script src="${dotTheme.path}js/core.min.js"></script>
<script src="${dotTheme.path}js/script.js"></script>
<script src="https://unpkg.com/htmx.org@1.7.0"></script>

#end
```
click Publish

## Simplify static /search/ page

## Create /search/results "partial" Page

## Refactor /search/ form to us HTMX Active Search

## Refactor Pagination to use HTMX Infinite Scroll