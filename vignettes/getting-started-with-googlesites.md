# Getting Started with googlesites
Steven Mortimer  
`r Sys.Date()`  



First, we load `googlesites` and specify the domain and site name we'll be working 
with. Creating a completely new site is only available to GSuite users (formerly 
known as Google Apps), so you should have already created a classic site using the 
web UI and know the domain and site name to plug into the package options. These 
options allow you to edit the site without specifying it explicitly on each request.


```r
library(googlesites)
options(googlesites.site_domain = "site")
options(googlesites.site_name = "sitesrtest")
```

Second, you need to use the `googleAuthR` package to get your OAuth2.0 token 
to make edits to your site from the R script. If it's your first time authenticating 
you RStudio will pop open your browser and ask you to authenticate interactively. 
After you've authenticated for the first time, a file called `.httr-oauth` will 
appear in your working directory. Every subsequent time that you run `gar_auth()` 
it will use the cached `.httr-oauth` file instead of making you do it interactively.


```r
library(googleAuthR)
options(googleAuthR.scopes.selected = "https://sites.google.com/feeds/")
gar_auth()
```



### Adding a Page

You can add a page based on a local, static HTML document. Webpages in Google Sites 
are based on HTML. You can specify new content with HTML that runs inside the 
template of your site. The HTML may not have some JavaScript and other potential 
security risks, so more advanced documents might need tweaking before deploying. Also, 
the data is transferred via XML, so your HTML must be XHTML compliant. This means that 
every tag should have a correpsonding closing tag. Do not write `<img src="test.png">`. 
Instead you should write `<img src="test.png" />`.


```r
test_html <- system.file("extdata", "example-site", "test.html", package="googlesites")
add_html_page(page_xhtml_source = test_html,
              page_title = 'API Test',
              page_custom_url = 'api-test',
              overwrite=TRUE)
#> Creating New Page
#> Successfully Created Page Skeleton
#> Replacing Links to Local Files
#> Successfully Added All Page Content
#> $page_id
#> [1] "https://sites.google.com/feeds/content/site/sitesrtest/4618599740961579025"
#> 
#> $page_url
#> [1] "https://sites.google.com/site/sitesrtest/api-test"
```

### Finding Content

The package comes with a function to find content on your site. You can specify which 
field to search on (title, id, url), then find content matching your desired value 
against that field. You can also restrict to certain types of content like webpages, 
announcements, templates. Here is an example where we find the page we just created 
by its title.


```r
find_content(value_to_match='API Test', 
             field_to_match='title', 
             content_category='webpage')
#>                                                                           id
#> 1 https://sites.google.com/feeds/content/site/sitesrtest/4618599740961579025
#>                                                 url category    title
#> 1 https://sites.google.com/site/sitesrtest/api-test  webpage API Test
#>   parent_page_id content_type
#> 1                       xhtml
```

You can also find this page by its URL or it's entry ID.


```r
page <- find_content(value_to_match='https://sites.google.com/site/sitesrtest/api-test', 
                     field_to_match='url', 
                     content_category='webpage')

find_content(value_to_match=page$id, 
             field_to_match='id', 
             content_category='webpage')
#>                                                                           id
#> 1 https://sites.google.com/feeds/content/site/sitesrtest/4618599740961579025
#>                                                 url category    title
#> 1 https://sites.google.com/site/sitesrtest/api-test  webpage API Test
#>   parent_page_id content_type
#> 1                       xhtml
```

The `find_content()` function supports regex-style matching via `grep()`. If you 
want to search for all templates on your site, you can just search for "*". 


```r
templates <- find_content(value_to_match='*', 
                          field_to_match='title', 
                          use_regex_matching=TRUE,
                          content_category='template')
templates
#>                                                                           id
#> 1 https://sites.google.com/feeds/content/site/sitesrtest/7471740061815467959
#>                                                                                  url
#> 1 https://sites.google.com/site/sitesrtest/config/pagetemplates/no-title-no-sublinks
#>           category                title parent_page_id content_type
#> 1 webpage|template no-title-no-sublinks                       xhtml
```

A nice feature about `add_html_page()` is that you can specify a template that 
you would like the page to be created with by providing the template ID. In this example 
we'll add the page using the first template we found. This one hides the title and 
turns off sublinks at the bottom of the page.


```r
sample_html <- system.file("extdata", "example-site", "sample-doc.html", package="googlesites")
add_html_page(page_xhtml_source = sample_html,
              page_title = 'Post with GIF',
              page_custom_url = 'post-with-gif',
              page_template_id = templates$id[1])
#> Creating New Page
#> Successfully Created Page Skeleton
#> Replacing Links to Local Files
#> Uploading Local Files to Sites Page
#> 
  |                                                                       
  |                                                                 |   0%
  |                                                                       
  |======================                                           |  33%
  |                                                                       
  |===========================================                      |  67%
  |                                                                       
  |=================================================================| 100%
#> Successfully Added All Page Content
#> $page_id
#> [1] "https://sites.google.com/feeds/content/site/sitesrtest/2544131551914011893"
#> 
#> $page_url
#> [1] "https://sites.google.com/site/sitesrtest/post-with-gif"
```

### Adding an Attachment

Once you've got pages up and running, it's easy to programmatically attach content 
to those pages. In this example, we're pulling down the RMarkdown cheatsheet from 
RStudio and attaching it to our "Post with GIF" page. You should see the attachment at the 
bottom of the page if you have page settings that show attachments. The nice thing about 
Google Sites is that if you re-upload something with the same name to the same page, 
it keeps track of it's revision number so you can always go back and get old versions of 
a file that you upload.


```r
pdf_cheatsheet <- 'https://www.rstudio.com/wp-content/uploads/2016/03/rmarkdown-cheatsheet-2.0.pdf'
download.file(pdf_cheatsheet, 'rmarkdown-cheatsheet-2.0.pdf', mode="wb")

upload_file_to_site(local_file_path = "rmarkdown-cheatsheet-2,0.pdf", 
                    file_summary = 'R Markdown is an authoring format that makes it 
                    easy to write reusable reports with R. You combine your R code 
                    with narration written in markdown (an easy-to-write plain text format) 
                    and then export the results as an html, pdf, or Word file. You can 
                    even use R Markdown to build interactive documents and slideshows. 
                    Updated 02/16.',
                    parent_page_id = find_content(value_to_match='Post with GIF', 
                                                  field_to_match='title')$id)
```


```
#> $file_id
#> [1] "https://sites.google.com/feeds/content/site/sitesrtest/8473038229992847045"
#> 
#> $file_url
#> [1] "https://sites.google.com/site/sitesrtest/post-with-gif/rmarkdown-cheatsheet-2.0.pdf"
```

### Deleting Content

Of course you can delete content from your site as well. If you don't want that 
PDF attachment anymore, then just find it's ID and remove it. In this example, 
note the use of the `parent_page_id` argument so that we don't accidentally 
remove the PDF if it exists somewhere else on the site with the same name.


```r
# deleting the attachment
delete_content(id = find_content(value_to_match='rmarkdown-cheatsheet-2.0.pdf',
                                 field_to_match='title', 
                                 parent_page_id=find_content(value_to_match='Post with GIF', 
                                                  field_to_match='title')$id,
                                 content_category='attachment')$id)
#> Content Succesfully Deleted.
```

### Deleting Page

You can even delete the first sample page we created. We recommend pausing your 
script in between deletions and insertions because the Sites API is sometimes 
slow at receiving and reflecting many rapid API calls to change content.


```r
# deleting page
delete_content(id = find_content(value_to_match='API Test',
                                 field_to_match='title')$id)
#> Content Succesfully Deleted.
```

Thats it! You can use these functions to automatically upload content and manage 
your Google Site. The beautiful thing is that you can version control the contents 
and repush any content as needed without fussing with the Web UI. It's recommended 
that you keep your site files all in one folder so you can keep track of them all. 
There is an example site folder bundled with the package and available [here](https://github.com/ReportMort/googlesites/tree/master/inst/extdata/example-site).
