---
date: 2020-07-05T13:59:15-04:00
title: "Building a Website With Hugo and the Personal Web Theme"
draft: false
categories:
  - static-site-gen
---

This is a sample project report provided as guidance for students in CSC 324 at Georgetown College.
<!--more-->

## Introduction

This article describes __pweb__, a personal website constructed using the [Hugo](https://gohugo.io/) static-site generator.  Currently __pweb__ is deployed at [https://netlify.pweb.com](https://netlify.pweb.com).

The repository is [on Github](https://github.com/homerhanumat/pweb).  A number of branches have been created at key stages of the development of the project, so the reader can check out the branches to examine the code-base at each step.  The branch `13-begin-report` gives the state of the repository at the time when this article was written.

## Choice of Theme

I decided that my website should be about myself as a professional:

* I wanted a site where I could describe my recent work in computer programming.
* I also wanted a blog, or at least some support for a section of articles.
* Finallly, I needed a fixed homepage with menu links to the the programming work and to the articles.

Beyond all that, I simply wanted the theme to be easy to understand and to customize.

Poking around a bit on the Hugo Themes site, I discovered [Personal Web](https://themes.gohugo.io/personal-web/).  It appeared to fit the bill nicely.

## Initial Customization

I began with the example site provided along with the theme, and then proceeded to personalize the `config.toml` file.

I changed the main site title and sub-title, replacing the `:wave:` emoji with a japanese goblin, which seems a better fit for me:
```toml
[params.intro]
  main = "Homer White"
  sub = "Professor of Mathematics and Computer Science.:japanese_goblin:"
```

I substituted my own logo image, and sidebar background:

```toml
[params.sidebar]
  backgroundImage = '/images/fm_sky.jpg'
  gradientOverlay = '' # default: rgba(0,0,0,0.4),rgba(0,0,0,0.4)
  logo = "/images/homer_white.jpg" 
```

Not being a great user of social media, I cut the social media links down to just Facebook, Github and my YouTube channel.  For the contact link on the sidebar I used my personal email address.

Initial changes to `config.toml` may be seen at commit [f039e6ac7](https://github.com/homerhanumat/pweb/commit/f039e6ac76450fe2d8a5068cfe212679f804b3a9).

The theme author provides a 404.html image, a cartoon gopher:

```toml
[params.notFound]
  gopher = "/images/gopher.png" # checkout https://gopherize.me/
  h1 = 'Bummer!'
  p = "It seems that this page doesn't exist."
```

I took the author up on the invitation to check out [https://gopherize.me/](https://gopherize.me/), and decided to make and use my own gopher image.  You can see it at work, if you like:

>[https://pweb.netlify.app/nothing-here](https://pweb.netlify.app/nothing-here)

## Modifications to the Theme

I made quite a few modifications to the theme.  Because I installed the theme as a git submodule so as to be able to pull updates from the theme repository, I left the theme files untouched and instead created over-riding files, e.g., in the `static` and  `layouts` directories at the root of the project.

A few of my favorite modifications are described below.

### Support for Categories

In the theme, the metadata for articles included title, author, date and estimated reading-time.  Since i plan to write all of the articles, I felt that I did not need an author field.  I also wans't interested in the reading-time.  However, I did want to be able to place articles into categories.

To accomplish this, I added a taxonomy for categories in `config.toml`:

```toml
[taxonomies]
  design = "designs"
  tech = "techs"
  category = "categories"
```

In `layouts/partials/post/information.html`, the partial that inserts metadata about each post, I modified the div that once contained reading time:

```html
<div class="reading-time">
    Categories:
  {{ if .Params.Categories }}
    {{ range .Params.Categories }}
      <span><a href="/categories/{{ . }}">{{ . }}</a> </span>
    {{ end }}
  {{ else }}
      None
  {{ end }}
</div>
```

Note that the categories for each post are to be specified in the front matter for the post (see the next section), and are then accessible to the partial as `.Params.Categories`.



### Archetype for Posts

I wanted to start new posts with appropriate front matter filled in, so in my `archetypes` folder I inserted the file `post.md`:

```txt
---
date: {{ .Date }}
title: "{{ replace .Name "-" " " | title }}"
draft: true
categories:
  - general
---

**Replace this text with content that appears in the post-listing.**
<!--more-->

Replace this with the rest of the content.
```

Now the command

```bash
hugo new --kind post  post/my-article.md
```

creates a starter post like this:

```txt
---
date: 2020-07-05T09:21:10-04:00
title: "My Article"
draft: true
categories:
  - general
---

**Replace this text with content that appears in the post-listing.**
<!--more-->

Replace this with the rest of the content.
```

### Custom CSS

I noticed a few styling issues:

* The theme did not ship with styling for tables.
* Unordered lists had a margin-top set to -15px, which caused nested unordered lists to ride up into the parent list item.
* In code fences, long lines of code spilled over the allotted display-area. (I noticed this issue while writing this article!)

I addressed this in a custom CSS file, which at the moment looks like this:

```css
/* theme set margin-top to -15px, looked bad: */
ul {
    margin-top: 0;
}

/* style table */
table {
    margin-bottom: 15px;
}
th, td {
    border-bottom: 1px solid #ddd;
  }
tr:hover {
    background-color: #f5f5f5;
}

/* address code overflow */
pre {
    overflow: scroll;
}
```

I named the file `custom.css` and placed it in my `static/css`directory, then modified the `config.toml` file:

```toml
[params.assets]
  favicon = ""
  customCSS = "/css/custom.css"
```

### Content of Default Single Pages

If one wants to add single page at the top level of the website, the standard procedure is to create a new markdown file at top level, e.g. `yo.md`.  In the theme, the structure this file is contolled by `layouts/_deault/single.html`:

```html
{{ partial "header" .}}

<h1>{{ .Title | markdownify }}</h1>
{{ .Content | markdownify }}

{{ partial "footer" .}}
```

Now `.Content` is the text that appears in the Markdown file after the front matter, rendered into HTML.  If one applies the Hugo function `markdownify` to `.Content`, nothing is returned, so the resulting `yo.html` will appear to be empty.

I corrected the layout file to:

```html
{{ partial "header" .}}

<h1>{{ .Title | markdownify }}</h1>
{{ .Content }}

{{ partial "footer" .}}
```

To see that the correction worked, look at:

[https://pweb.netlify.app/yo](https://pweb.netlify.app/yo)

You will see some content!

### Deployment

The theme has trouble handling URLs, interfering with some types of deployment, including deployment with GitHub pages in the `docs` folder.  (See [this Issue](https://github.com/bjacquemet/personal-web/issues/15) on GitHub.)

Deployment on Netlify presented no problems.

## Desiderata

There are few further thngs I would like to explore.

* __Further CSS__: As I write articles I'll probably see the need for more custom styling.  (I'm already thinking that tables should fill up the available width of their container.)
* __Investigate Use of Relative URLs__: The theme's deployment issues probably could be corrected by swithcing to a relative URLs for all internal links.  I would like to work on this.
* __Contribute to the Theme__: If I can fix the URL issue then I'll probably contribute that and other fixes to the theme repository, in the form of a pull request.

## Grading Myself

__Note__:  You don't need this section in your report.

How did i do with respect to the specific requirement for the report?  Let's see:

* __Structure__:  The site had all required pages and then some, and t was easy to navigate using the sidebar menu and the "address" field at the top of the main content div.
* __Articles__:  I had engouh sample articles to show off my use of categories, and one of my articles was the required report.
* __Customization__:  I fell badly short by not making the About page really about myself; instead it was just a bunch of Markdown testing.  On the other hand, I did modify `config.toml` extensively, and I made some non-trivial changes to the theme.  I also added a couple of my own projects into the Portfolos.
* __Currency__:  This theme served and compiled sites without any warnings from Hugo. so I was fine there.
* __Posting/Deployment__:  The repository is posted on Github, and the site is deployed on Netlify.

Here's how I score myself:

|Category|Possible|My Score
|:----|----|-----|
|Stucture|4|4|
|Articles|2|1|
|Customization|10|11|
|Currency|4|4|
|Posting/Deployment|4|4|

My total score is 24/25.  :clap: :clap: :clap:


