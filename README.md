# dotcms-htmx
Using dotCMS to manage 

## Overview

You will want two browser windows open for this - one logged into the DotCMS backend and one not logged in.

## Running DotCMS with demo site content

DotCMS provides a "demo" site to allow you to test and explore DotCMS features. Run DotCMS locally using the `single-node-demo-site` from our [docker compose examples](https://github.com/dotCMS/core/tree/master/docker/docker-compose-examples)

### Run DotCMS from Docker
On current OS X:

Install and run [Docker Desktop](https://www.docker.com/products/docker-desktop)

In Docker Desktop -> Preferences -> Resources -> Advanced -> increase Memory to at least 6G

Open `Terminal` application and run:
```sh
curl -o docker-compose.yml https://raw.githubusercontent.com/dotCMS/core/master/docker/docker-compose-examples/single-node-demo-site/docker-compose.yml
docker compose up -d 
docker ps
```
You should see docker containers for DotCMS, Postgres, and Elasticsearch.

### Open browsers to DotCMS front-end and back-end 
DotCMS app is listening on two ports - either is fine, we'll use 8080.

`http:  8080`

`https: 8443` - you must accept the SSL cert 

Open a broweser and log in to DotCMS backend:

[http://localhost:8080/dotAdmin/](http://localhost:8080/dotAdmin/) 

```
username: admin@dotcms.com
password admin
```

Open a different browser or incognito window and browse to the Search page - search results won't work until you create a Site Search index.

[http://localhost:8080/search/](http://localhost:8080/search/) 

 A version of this site is always publically available at [https://demo.dotcms.com](https://demo.dotcms.com) but it is refreshed multiple times/day so your changes a) will not persist and b) can be changed by others using the site.

## Create Site Search Index
[DotCMS Site Search Docs](https://dotcms.com/docs/latest/site-search)

DotCMS requires a Site Search index to support searches from front-end sites. 

Create a Site Search index in the back-end:

`Dev Tools -> Site Search -> Job Scheduler -> create a job:`
```
Run = Now
Select Sites to Index = Index All Sites
Alias Name = default
Language = English, Espanol
```
Click EXECUTE 

The "View All Jobs" tab shows the progress. It should take only a couple minutes for dotCMS to crawl the front-end pages and create an index.

Once complete, the "Indices" tab should show a healthy index.

### Note the query strings used by static search page
In the front-end browser, search for "ski"

You should see search results. Note the uri for the first page of results:

`/search/index?q=ski&search=Search`

Scroll down and click **2** for the next page of results and note the uri ads `&p=` for the pages which are zero-indexed:

`/search/index?q=ski&p=1`
[http://localhost:8080/search/index?q=ski&p=1](http://localhost:8080/search/index?q=ski&p=1) 

## Add HTMX javascript to Theme footer
In the back-end go to:

`Site -> Browser -> application -> themes -> travel`

Double-click `footer.vtl` - scroll to the bottom of file editor and add HTMX to the Javascript section:
```html
<!-- Javascript-->
<script src="${dotTheme.path}js/core.min.js"></script>
<script src="${dotTheme.path}js/script.js"></script>
<script src="https://unpkg.com/htmx.org@1.7.0"></script>

#end
```
click Publish

In the other browser, "view source" on the [/search/](http://localhost:8080/search/) page and confirm HTMX is included at the bottom.

## Simplify static /search/ page
Let's see the current behavior of the "static" search page which we are going to refactor.

On [/search/](http://localhost:8080/search/) submit a search for "ski"

The search results are paginated with 10 results page - scroll down to see links to the other pages.

Also note the "By Type" and "Month Modified" search results aggregations.

### Locate the VTL code in DotCMS
To simplify the vtl for this example, let's remove the aggregations from the results page. 

How do we find this code? In the backend,

Site -> Browser -> search -> double-click "index" -> click EDIT ->

mouse over the "What are you looking for?" search form container -> click edit (pencil icon) ->

click INFO next to the site-search.vtl file to find it's location: `/application/vtl/site-search/site-search.vtl`

Close the edit modals to get back to the Site Browser and open this vtl file:

application -> vtl -> site-search -> site-search.vtl

We're going to refactor this code so peruse it. 

[Lines 1-126](https://github.com/yolabingo/dotcms-htmx/blob/e2d2dc774f2ac042048019393a68ef0306621a13/site-search.vtl#L1-L126) handle query-related tasks. 

[Lines 128-318](https://github.com/yolabingo/dotcms-htmx/blob/e2d2dc774f2ac042048019393a68ef0306621a13/site-search.vtl#L128-L318) generate HTML including the search form, search results, and search aggregation results.


### Remove the aggregations-related code from site-search.vtl

To simplify the VTL for this example, remove the three blocks of code related to aggregations.

The result is [/2-remove-search-aggregations/site-search.vtl](https://github.com/yolabingo/dotcms-htmx/blob/2-remove-search-aggregations/site-search.vtl)

## Create /search/results "partial" Page

## Refactor /search/ form to us HTMX Active Search

## Refactor Pagination to use HTMX Infinite Scroll