# dotcms-htmx

A tutorial to integrate htmx into dotCMS demo site's search page.

## Overview {#Overview}
Htmx is a library that exposes modern browser features via HTML attributes, rather than using javascript.

dotCMS, in addition to its extensive APIs and headless features, is also a great platform to manage rich websites with server-generated HTML pages.

[Htmx](https://htmx.org/) is a library that allows you to access modern browser features directly from HTML, rather than using Javascript. For some common website features built with htmx, see the following two examples:

- [htmx Active Search](https://htmx.org/examples/active-search/) example — slowly type a name in the demo at the bottom of the page.
- [htmx Infinite Scroll](https://htmx.org/examples/infinite-scroll/) works through an attribute applied to a final item in a list, and then likewise applied to the final item of each list segment the server appends.

Combining dotCMS and htmx allows web developers to incorporate responsive, AJAX-style elements and features to server-generated HTML pages with a minimum of composed Javascript — which is to say, in some estimations, a minimum of fuss.

This article will guide you through the basic setup of an htmx-enabled dotCMS instance from scratch.

## Running dotCMS With Demo Site Content {#RunningDotCMSDemo}

dotCMS provides a Demo site to allow you to test and explore dotCMS features.

### Run dotCMS from Docker {#RunDotCMS}

> Below are a summarized set of steps, but you can also follow the instructions on our [Quick-Start Guide](/docs/latest/quick-start-guide).

- Install and run [Docker Desktop](https://www.docker.com/products/docker-desktop).
- In **Docker Desktop -> Preferences -> Resources -> Advanced**, you may wish to increase the allocated memory to improve performance. Generally, **4 GB** is a good "medium" level of headroom for current versions. However, depending on your needs and implementation, you may be able to run comfortably on less memory; likewise, heavier use may call for more.

To run the dotCMS demo site locally, use the `single-node-demo-site` from our [Docker Compose examples](https://github.com/dotCMS/core/tree/master/docker/docker-compose-examples):

Open your command-line application (Terminal on macOS or Command Prompt on Windows) and run:

```sh
curl -o docker-compose.yml https://raw.githubusercontent.com/dotCMS/core/master/docker/docker-compose-examples/single-node-demo-site/docker-compose.yml
docker compose up -d 
docker ps
```

> (Note: Windows 10 & 11 both come with the `curl` command preinstalled. However, if you're using Windows 7 or 8, you'll need to [install it](https://curl.se/)).)

You should see the software create Docker containers for dotCMS, Postgres, and Elasticsearch. The process may take a few minutes to complete. 

### Open dotCMS Front and Back Ends {#FrontEndBackEnd}

To most easily observe changes in real time, use two browser windows that do not share session data — either separate browsers or one normal and one Private/Incognito window. Sign in with one; leave the other signed out, to emulate a normal site visitor.

The dotCMS application is listening on two ports by default:
- HTTP: `8082`
- HTTPS: `8443` (you must accept the SSL certificate)

Either is fine, but we'll use `8082`. Sign in with the following link: [http://localhost:8082/dotAdmin/](http://localhost:8082/dotAdmin/) 

- username: `admin@dotcms.com`
- password: `admin`

> Note: At the time of writing, the YAML file includes a `DOT_INITIAL_ADMIN_PASSWORD` environment variable, set to the value `admin`. If this is absent, then the default admin password will instead be a randomly generated string that can be found in the server's startup logs. See the [Quick Start Guide](/docs/latest/quick-start-guide) for more information.

A version of the demo site is always publically available at [https://demo.dotcms.com](https://demo.dotcms.com) but it is recreated multiple times/day so your changes will not persist and can be changed by others using the site.

## Load Htmx Javascript in the Theme Footer {#LoadHtmx}

> For the official instructions, see the [htmx installation documentation](https://htmx.org/docs/#installing).

In the back end, go to: **Site -> Browser -> application -> themes -> travel**

Double-click `footer.vtl` — scroll to the bottom of file editor and add htmx to the Javascript section:

```html
<!-- Javascript-->
<script src="${dotTheme.path}js/core.min.js"></script>
<script src="${dotTheme.path}js/script.js"></script>
<script src="https://unpkg.com/htmx.org@1.7.0"></script>
#end
```

Click `Publish`.

On the front-end site, reload and "view source" on the [/search/](http://localhost:8082/search/) page to confirm htmx is included in the footer.

Just like that, **we're up and running with htmx**!

But before we call our work finished, **let's build something with it** over the following sections. We'll be interacting with dotCMS's search functionality, so the first order of business is to build an index.

## Create dotCMS Site Search Index {#CreateSearchIndex}

Open a different browser or incognito window and browse to the Search page: [http://localhost:8082/search/](http://localhost:8082/search/)

As you'll see, the search function is operational, but you won't see any results until you create a Site Search index; dotCMS requires an index to support searches from front-end sites.

To create a Site Search index in the back end, navigate to **Dev Tools -> Site Search -> Job Scheduler**. Next, create a job with the following parameters:

- Run: `Now`
- Select Sites to Index: Select `Index All Sites` checkbox
- Alias Name: `default`
- Language: `English, Espanol`

Finally, click `EXECUTE`.

The "View All Jobs" tab shows your progress. It should take only a couple minutes for dotCMS to crawl the front-end pages and create an index.

Once complete, the "Indices" tab should show a healthy index.

> For more general info about search functionality, see the [dotCMS Site Search Docs](https://dotcms.com/docs/latest/site-search).

### Note the Query Strings Used by the Search Page {#QueryStringsQuery}

Once the index is in place, [returning to the front-end search page](http://localhost:8082/search/), try searching for `ski`.

You should see search results. Note the URI for the first page of results:

`/search/index?q=ski&search=Search`

Scroll down and click **2** for the next page of results and note the URI adds `&p=[page_number]` for the pages (which are zero-indexed):

`/search/index?q=ski&p=1`

[http://localhost:8082/search/index?q=ski&p=1](http://localhost:8082/search/index?q=ski&p=1)

## Simplify VTL in the Static `/search/` Page {#SimplifyVTL}

Let's examine the behavior of the "static" search page which we are going to refactor.

On [/search/](http://localhost:8082/search/) once again submit a search for `ski`.

You will see that search results are paginated with 10 results page. Scroll down to see links to the other pages of results. Also note the "By Type" and "Month Modified" search results aggregations.

### Locate the "Site Search" VTL Code in dotCMS {#TheSearchForSearch}

To simplify the VTL for this example, let's remove the aggregations from the results page. 

How do we find this code? In the back-end browse to **Site -> Browser -> search**. Now double-click `index`, and then click the "Edit" tab.

Mouse over the "What are you looking for?" search form container and you will see a button  containing a pencil icon. Click on that to edit.

Now, click the `INFO` button to the right of the VTL File field to find its location: `/application/vtl/site-search/site-search.vtl`

Close the edit modals and return to the Site Browser. Now navigate to that file, and double click it to open.

Feel free to peruse the code to get a basic lay of the land. To summarize:
- [Lines 1-126](https://github.com/yolabingo/dotcms-htmx/blob/e2d2dc774f2ac042048019393a68ef0306621a13/site-search.vtl#L1-L126) handle query-related tasks. 
- [Lines 128-318](https://github.com/yolabingo/dotcms-htmx/blob/e2d2dc774f2ac042048019393a68ef0306621a13/site-search.vtl#L128-L318) generate HTML including the search form, search results, and search aggregation results.

### Remove the Aggregations-Related Code {#SimplifySearchSurgery}

To simplify the VTL for this example, we will remove the three blocks of code related to aggregations.

Replace the current VTL code with this code: [/2-remove-search-aggregations/site-search.vtl](https://raw.githubusercontent.com/yolabingo/dotcms-htmx/2-remove-search-aggregations/site-search.vtl)

Now click `Publish`.

## Create `/search/results` "Partial" Page {#SearchResultsPartial}

We'll need to create a new Page on the site that returns "html over the wire" containing only the results of search queries. This must not include our header or footer files for "normal" HTML Pages.

### Create Search-Results Container {#SearchResultsPartialContainer}

First, create a new Container.

Go to **Site -> Containers**. Click the Blue `+` button and then select `Add Container` and input the following:
- Title: `search-results-htmx`
- Code: (paste in the `site-search.vtl` code)

This code must now be refactored — it need not generate the search form any more. It returns only the HTML for search results — basically a `<ul>` with an  `<li>` for each result.

Don't worry, we've got you covered: Just paste [this refactored version of our VTL](https://raw.githubusercontent.com/yolabingo/dotcms-htmx/3-split-search-form-and-results/search-results-htmx.vtl) in the "Code" section.

Click `Save and Publish`.

### Create Search-Results Template {#SearchResultsPartialTemplate}

Next, make a new Template that includes only this container. In the back end, navigate to **Site -> Templates**. Right-click "Blank," and then pick Copy. Now double-click the copied "Blank - 1" and click the `EDIT` button beside the title.

- Title: `search-results-htmx` 
- Description: `htmx "html-over-the-wire" search results`

Now `Save`. 

Then click the "Add a Container" dropdown, select the `search-results-htmx - demo.dotcms.com` item. Click the activated `Save` button.

Go back to Templates list to Publish the Template: **Site -> Templates**, then right-click `search-results-htmx` and select `Publish`.

### Create Search-Results Page {#SearchResultsPartialsPage}

Create a new Page using the new Template. Visit **Site -> Browser**. Click on the `search` folder, and then click the blue `+` button and select `Page` (type = "Page"). Set the following values:
- Host or Folder: `search`
- Title: `search-results`
- Template: `search-results-htmx (demo.dotcms.com)`

Click `Publish`.

### Test Search-Results Page {#SearchResultsPartialsTest}

Try out a couple of tests:
- [http://localhost:8082/search/search-results?q=ski](http://localhost:8082/search/search-results?q=ski)
- [http://localhost:8082/search/search-results?q=ski&p=2](http://localhost:8082/search/search-results?q=ski&p=2)

Looks good! There is some pagination cruft to clean up, but the functionality is there.

### Final Version of `/search/search-results` Page {#SearchResultsPartialsFinal}

[This final version of search-results-htmx.vtl](https://github.com/yolabingo/dotcms-htmx/blob/main/search-results-htmx.vtl) adds some finishing touches. Copy this VTL ([raw link](https://raw.githubusercontent.com/yolabingo/dotcms-htmx/main/search-results-htmx.vtl)) and replace the code: Navigate to **Site -> Containers**, double-click `search-results-htmx`, paste it into the "Code" field, and then `Save and Publish`.

## Refactor Pagination Logic With Htmx Infinite Scroll {#InfiniteScroll}

> More detailed information on this functionality can be found in the [htmx Infinte Scroll documentation](https://htmx.org/examples/infinite-scroll/).

We can use the underlying pagination logic in the VTL with htmx's "infinite scroll" capabilities.

When last `<li>` in the search results is visible on the screen, htmx will fetch 10 more results and append them to the `<ul>`.

This is achieved simply by adding a few htmx attributes to the final `<li>` on each page:

```html
<li hx-get="/search/search-results?q=ski&p=[next_page]"
    hx-trigger="revealed"
    hx-swap="afterend">
```

## Refactor `/search/` Form to Use Htmx Active Search {#ActiveSearch}

> For more on this, see the [htmx Active Search docs](https://htmx.org/examples/active-search/).

We're at the finish line! 

Let's go back to **Site -> Browser**, then navigate to `application/vtl/site-search/site-search.vtl`. 

Replace the file contents with [this final version of site-search.vtl](https://github.com/yolabingo/dotcms-htmx/blob/main/site-search.vtl).

All that needs to remain of the original code is the search form and associated markup; from there, we add htmx attributes and disable the default submit behavior. A stripped-down version demonstrating the htmx elemets is:

```html
<form action="javascript:void(0);">
    <input  name="q"
            type="text"
            hx-get="/search/search-results"
            hx-target="#search-results-htmx"
            hx-indicator=".htmx-indicator"
            hx-trigger="keyup changed delay:500ms, search">
</form>
<span class="htmx-indicator"><img src="/application/themes/travel/images/spinner.gif"/> Searching...</span>
<span id="search-results-htmx"></span>
```

Go to [http://localhost:8082/search/](http://localhost:8082/search/) and try a few searches. For searches with more than 10 results, such as:
- `store`
- `ski`
- `blog`

Now scroll down to see the "infinite scroll" behavior.

As you can see, the actual code requirements of this were vastly simplified by htmx. If you prefer working in a pure HTML/CSS paradigm over working with Javascript — recognizing, of course, the minor irony of htmx being a Javascript library — you may find many more elegant little time-savers at your disposal with htmx and dotCMS.
