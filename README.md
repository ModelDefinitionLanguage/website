# website
Community resources for the DDMoRe Model Description Language (MDL)

The website is built using [RStudio's blogdown package](https://github.com/rstudio/blogdown).

Content is created in the /content/ folder
  - Blog posts under /content/post
  - Pages under /content/page
  
Content can be authored by creating .Rmd files which will then be rendered as HTML and organised by using blogdown::build_site().

The site is built into the /docs/ folder on the master branch which is then used as the basis for the Github webpage.
