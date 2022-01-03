# DFD Hugo Module for Metadata for Pages

### Description

Basic documentation for metadata-mod-hugo-dfd, a Hugo module by Daniel F. Dickinson for handling page metadata

_Note_: The exampleSite displays the gathered/generated metadata but does not modify the theme to use it (except to add the information gathered/generated to the footer).
## Status

![build-and-verify](https://github.com/danielfdickinson/metadata-mod-hugo-dfd/actions/workflows/build-and-verify.yml/badge.svg)

## Features

* Provides a common method of accessing page metadata for use in layouts and shortcodes.
* Has a unified and comprehensive means of finding page and site titles.
* For \<head> metadata, supports either self-closing or 'void style' empty tags (that is either \<meta name="some-meta" content="some-content" /> or \<meta name="some-meta" content="some-content">)
  * **NB**: Only for \<head> tags generated by this module; you'll need to implement the same logic for the rest of the items in your \<head> section if you need consistency.
* Partials are 'shortcode safe' (which means they can work from layouts or shortcodes).

### Page and Site Title Logic

## Basic Usage

### Importing the Module

1. The first step to making use of this module is to add it to your site or theme.  In your configuration file:

   ``config.toml``
   ```toml
   [module]
     [[module.imports]]
       path = "github.com/danielfdickinson/metadata-mod-hugo-dfd"
   ```
   OR
   ``config.yaml``
   ```yaml
   module:
     imports:
       - path: github.com/danielfdickinson/metadata-mod-hugo-dfd
   ```
2. Execute
   ```bash
   hugo mod get github.com/danielfdickinson/metadata-mod-hugo-dfd
   hugo mod tidy
   ```

### \<head> metadata configuration

* If ``emptyElementStyle`` is set to ``self-close`` in params (site or per-page), then empty tags produced by this module use 'self-closing' form (see above), otherwise 'void style' (see above).
  For example, with ``toml`` for the site as
  ```toml
  [params]
      emptyElementStyle = "self-close"
  ```
  you get
  ```html
      <meta name="generator" content="Hugo 0.91.2" />
      <meta name="keywords" content="Placeholder" />
  ```
  while not configuring ``emptyElementStyle`` produces
  ```html
      <meta name="generator" content="Hugo 0.91.2">
      <meta name="keywords" content="Placeholder">
  ```
  for ``exampleSite/docs/placeholder.md`` (which is output as ``public/docs/placeholder/index.html``) in the 'Test of metadata for \<head>' section of the page.

### 'summary' metadata configuration

* If ``summaryRenderMarkdown`` is true in Params (site or per-page) then we allow setting a summary in the frontmatter that contains markdown, which we render.
* If ``summaryPreserveHtml`` is true in Params (site or per-page) then we return a summary that will render any contained HTML. Note that HTML is only preserved using the manual split method of generating a summary (that is using ``<!--more-->`` as described in [the Hugo docs for Content Summaries](https://gohugo.io/content-management/summaries)), or we rendered Markdown (above).
* Also note that if you render markdown but don't preserve HTML the generated HTML is converted to plain text (i.e. will have no formatting).

### Notes on 'description' metadata item

* For a page (including sections, taxonomies, and terms, but not the home page): Uses ``.Description`` if present, otherwise defaults to the summary (if present).
  * For the found description: Turn into plain text, replace newlines and carriage-returns with spaces, and condense all multiple space sequences to single spaces.
* For the home page: defaults to ``.Description`` if present, otherwise the home page frontmatter description, otherwise the site ``.Description``, otherwise summary from ``.Site.Params``
  * For the found description: Render markdown if present, turn into plain text, replace newlines and carriage-returns with spaces, then condense all multiple space sequences to single spaces.

### Notes on 'Date, PublishDate, and Lastmod (dateCreated, datePublished, and dateModified)' metadata items

* For pages that do no exist in the ``content`` directory (e.g. taxonomies, terms, subdirectories/sections with no '_index.md' or 'index.md' and/or a homepage with no '_index.md' in 'content'), then Hugo's ``.PublishDate`` (Open Graph name 'datePublished') is .IsZero, treated as false or not present depending on the method of use. This is true even with the ``config.toml`` ``[frontmatter]`` ``publishDate`` configured.
* With this exception, [the preferences for automatic values for the various date .Page variables can be configured using the ``frontmatter`` section on your Hugo config](https://gohugo.io/getting-started/configuration/#configure-dates).
* In this module, if ``datesPreserveTimezone`` is ``true`` in page or site ``Params``, then the timezone of the date is not adjusted. If this is not the case, all dates are converted to UTC. You can of course use ``.Local`` or other date functions to adjust the output of the date variables.
* Not all themes are consistent in treating ``.Date`` as 'dateCreated' even though Hugo's internal templates (like RSS feed generator) have that expectation. I'd recommend modifying the theme's date handling if this is the case.

### Taxonomies and 'keywords' \<head> metadata

* For all pages except the homepage, any taxonomy terms (but not taxonomies themselves; so not 'tags' and 'categories' for the default taxonomies configuration, but the terms you specify under 'tags' and/or under 'categories' in page frontmatter) in the page frontmatter are used to fill in the 'keywords' \<meta> tag.
* For the homepage all taxonomy terms (but not taxonomies) are used to fill in the 'keywords' \<meta> tag.
* The 'keywords' meta tag uses a comma separated list of terms, with no space added before or after the comma as a flat string as [defined in the HTML5 spec](https://html.spec.whatwg.org/multipage/semantics.html#standard-metadata-names).
* Note that this list is likely not going to get used by search engines due to historical use of this to mislead naive search engines.

### 'type' metadata

* Refers to the item type for feeds and Open Graph. We only use 'website' and 'article'. The homepage is of 'type' 'website' and other pages with a 'publishDate' (that is, all pages which exist under ``content`` and have frontmatter) are considered of type 'article'.
* This can by overridden by a frontmatter ``Open GraphType`` in the page-level ``Params``. See [Object Types in the Open Graph Protocol](https://ogp.me/#types).
