---
aliases:
- /layout/functions/
date: 2013-07-01
linktitle: Functions
menu:
  main:
    parent: layout
next: /templates/variables
prev: /templates/go-templates
title: Hugo Template Functions
weight: 20
---

Hugo uses the excellent Go html/template library for its template engine.
It is an extremely lightweight engine that provides a very small amount of
logic. In our experience it is just the right amount of logic to be able
to create a good static website.

Go templates are lightweight but extensible. Hugo has added the following
functions to the basic template logic.

(Go itself supplies built-in functions, including comparison operators
and other basic tools; these are listed in the
[Go template documentation](http://golang.org/pkg/text/template/#hdr-Functions).)

## General

### isset
Return true if the parameter is set.
Takes either a slice, array or channel and an index or a map and a key as input.

e.g. `{{ if isset .Params "project_url" }} {{ index .Params "project_url" }}{{ end }}`

### echoParam
If parameter is set, then echo it.

e.g. `{{echoParam .Params "project_url" }}`

### eq
Return true if the parameters are equal.

e.g.

    {{ if eq .Section "blog" }}current{{ end }}

### first
Slices an array to only the first X elements.

Works on [lists](/templates/list/), [taxonomies](/taxonomies/displaying/), [terms](/templates/terms/), [groups](/templates/list/)

e.g.

    {{ range first 10 .Data.Pages }}
        {{ .Render "summary" }}
    {{ end }}

### where
Filters an array to only elements containing a matching value for a given field.

Works on [lists](/templates/list/), [taxonomies](/taxonomies/displaying/), [terms](/templates/terms/), [groups](/templates/list/)

e.g.

    {{ range where .Data.Pages "Section" "post" }}
       {{ .Content}}
    {{ end }}

It can be used with dot chaining second argument to refer a nested element of a value.

e.g.

    // Front matter on some pages
    +++
    series: golang
    +++

    {{ range where .Site.Recent "Params.series" "golang" }}
       {{ .Content}}
    {{ end }}

*where and first can be stacked*

e.g.

    {{ range first 5 (where .Data.Pages "Section" "post") }}
       {{ .Content}}
    {{ end }}

### delimit
Loops through any array, slice or map and returns a string of all the values separated by the delimiter. There is an optional third parameter that lets you choose a different delimiter to go between the last two values.
Maps will be sorted by the keys, and only a slice of the values will be returned, keeping a consistent output order.

Works on [lists](/templates/list/), [taxonomies](/taxonomies/displaying/), [terms](/templates/terms/), [groups](/templates/list/)

e.g.
    // Front matter
    +++
    tags: [ "tag1", "tag2", "tag3" ]
    +++

    // Used anywhere in a template
    Tags: {{ delimit .Params.tags ", " }}

    // Outputs Tags: tag1, tag2, tag3

    // Example with the optional "last" parameter
    Tags: {{ delimit .Params.tags ", " " and " }}

    // Outputs Tags: tag1, tag2 and tag3

### sort
Sorts maps, arrays and slices, returning a sorted slice. A sorted array of map values will be returned, with the keys eliminated. There are two optional arguments, which are `sortByField` and `sortAsc`. If left blank, sort will sort by keys (for maps) in ascending order.

Works on [lists](/templates/list/), [taxonomies](/taxonomies/displaying/), [terms](/templates/terms/), [groups](/templates/list/)

e.g.
    // Front matter
    +++
    tags: [ "tag3", "tag1", "tag2" ]
    +++

    // Site config
    +++
    [params.authors]
      [params.authors.Derek]
        "firstName"  = "Derek"
        "lastName"   = "Perkins"
      [params.authors.Joe]
        "firstName"  = "Joe"
        "lastName"   = "Bergevin"
      [params.authors.Tanner]
        "firstName"  = "Tanner"
        "lastName"   = "Linsley"
    +++

    // Use default sort options - sort by key / ascending
    Tags: {{ range sort .Params.tags }}{{ . }} {{ end }}

    // Outputs Tags: tag1 tag2 tag3

    // Sort by value / descending
    Tags: {{ range sort .Params.tags "value" "desc" }}{{ . }} {{ end }}

    // Outputs Tags: tag3 tag2 tag1

    // Use default sort options - sort by value / descending
    Authors: {{ range sort .Site.Params.authors }}{{ .firstName }} {{ end }}

    // Outputs Authors: Derek Joe Tanner

    // Use default sort options - sort by value / descending
    Authors: {{ range sort .Site.Params.authors "lastName" "desc" }}{{ .lastName }} {{ end }}

    // Outputs Authors: Perkins Linsley Bergevin

### in
Checks if an element is in an array (or slice) and returns a boolean.  The elements supported are strings, integers and floats (only float64 will match as expected).  In addition, it can also check if a substring exists in a string.

e.g.

    {{ if in .Params.tags "Git" }}Follow me on GitHub!{{ end }}

or

    {{ if in "this string contains a substring" "substring" }}Substring found!{{ end }}

### intersect
Given two arrays (or slices), this function will return the common elements in the arrays.  The elements supported are strings, integers and floats (only float64).

A useful example of this functionality is a 'similar posts' block.  Create a list of links to posts where any of the tags in the current post match any tags in other posts.

e.g.

    <ul>
    {{ $page_link := .Permalink }}
    {{ $tags := .Params.tags }}
    {{ range .Site.Recent }}
        {{ $page := . }}
        {{ $has_common_tags := intersect $tags .Params.tags | len | lt 0 }}
        {{ if and $has_common_tags (ne $page_link $page.Permalink) }}
            <li><a href="{{ $page.Permalink }}">{{ $page.Title }}</a></li>
        {{ end }}
    {{ end }}
    </ul>


## Math

### add
Adds two integers.

e.g. `{{add 1 2}}` → 3

### sub
Subtracts two integers.

e.g. `{{sub 3 2}}` → 1

### div
Divides two integers.

e.g. `{{div 6 3}}` → 2

### mul
Multiplies two integers.

e.g. `{{mul 2 3}}` → 6

### mod
Modulus of two integers.

e.g. `{{mod 15 3}}` → 0

### modBool
Boolean of modulus of two integers.
true if modulus is 0.

e.g. `{{modBool 15 3}}` → true

## Strings

### urlize
Takes a string and sanitizes it for usage in URLs, converts spaces to "-".

e.g. `<a href="/tags/{{ . | urlize }}">{{ . }}</a>`

### safeHtml
Declares the provided string as "safe" so Go templates will not filter it.

e.g. `{{ .Params.CopyrightHTML | safeHtml }}`


### markdownify

This will run the string through the Markdown processesor. The result will be declared as "safe" so Go templates will not filter it.

e.g. `{{ .Title | markdownify }}`

### lower
Convert all characters in string to lowercase.

e.g. `{{lower "BatMan"}}` → "batman"

### upper
Convert all characters in string to uppercase.

e.g. `{{upper "BatMan"}}` → "BATMAN"

### title
Convert all characters in string to titlecase.

e.g. `{{title "BatMan"}}` → "Batman"

### chomp
Removes any trailing newline characters. Useful in a pipeline to remove newlines added by other processing (including `markdownify`).

e.g., `{{chomp "<p>Blockhead</p>\n"` → `"<p>Blockhead</p>"`

### highlight
Take a string of code and a language, uses Pygments to return the syntax highlighted code in HTML. Used in the [highlight shortcode](/extras/highlighting).

### ref, relref
Looks up a content page by relative path or logical name to return the permalink (`ref`) or relative permalink (`relref`). Requires a Node or Page object (usually satisfied with `.`). Used in the [`ref` and `relref` shortcodes]({{% ref "extras/crossreferences.md" %}}).

e.g. {{ ref . "about.md" }}

## Advanced

### apply

Given a map, array, or slice, returns a new slice with a function applied over it. Expects at least three parameters, depending on the function being applied. The first parameter is the sequence to operate on; the second is the name of the function as a string, which must be in the Hugo function map (generally, it is these functions documented here). After that, the parameters to the applied function are provided, with the string `"."` standing in for each element of the sequence the function is to be applied against. An example is in order:

    +++
    names: [ "Derek Perkins", "Joe Bergevin", "Tanner Linsley" ]
    +++

    {{ apply .Params.names "urlize" "." }} → [ "derek-perkins", "joe-bergevin", "tanner-linsley" ]

This is roughly equivalent to:

    {{ range .Params.names }}{{ . | urlize }}{{ end }}

However, it isn’t possible to provide the output of a range to the `delimit` function, so you need to `apply` it. A more complete example should explain this. Let's say you have two partials for displaying tag links in a post,  "post/tag/list.html" and "post/tag/link.html", as shown below.

    <!-- post/tag/list.html -->
    {{ with .Params.tags }}
    <div class="tags-list">
      Tags:
      {{ $len := len . }}
      {{ if eq $len 1 }}
        {{ partial "post/tag/link" (index . 0) }}
      {{ else }}
        {{ $last := sub $len 1 }}
        {{ range first $last . }}
          {{ partial "post/tag/link" . }},
        {{ end }}
        {{ partial "post/tag/link" (index . $last) }}
      {{ end }}
    </div>
    {{ end }}


    <!-- post/tag/link.html -->
    <a class="post-tag post-tag-{{ . | urlize }}" href="/tags/{{ . | urlize }}">{{ . }}</a>

This works, but the complexity of "post/tag/list.html" is fairly high; the Hugo template needs to perform special behaviour for the case where there’s only one tag, and it has to treat the last tag as special. Additionally, the tag list will be rendered something like "Tags: tag1 , tag2 , tag3" because of the way that the HTML is generated and it is interpreted by a browser.

This is Hugo. We have a better way. If this were your "post/tag/list.html" instead, all of those problems are fixed automatically (this first version separates all of the operations for ease of reading; the combined version will be shown after the explanation).

    <!-- post/tag/list.html -->
    {{ with.Params.tags }}
    <div class="tags-list">
      Tags:
      {{ $sort := sort . }}
      {{ $links := apply $sort "partial" "post/tag/link" "." }}
      {{ $clean := apply $links "chomp" "." }}
      {{ delimit $clean ", " }}
    </div>
    {{ end }}

In this version, we are now sorting the tags, converting them to links with "post/tag/link.html", cleaning off stray newlines, and joining them together in a delimited list for presentation. That can also be written as:

    <!-- post/tag/list.html -->
    {{ with.Params.tags }}
    <div class="tags-list">
      Tags:
      {{ delimit (apply (apply (sort .) "partial" "post/tag/link" ".") "chomp" ".") ", " }}
    </div>
    {{ end }}

`apply` does not work when receiving the sequence as an argument through a pipeline.
