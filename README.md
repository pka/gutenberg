# Gutenberg
[![Build Status](https://travis-ci.org/Keats/gutenberg.svg?branch=master)](https://travis-ci.org/Keats/gutenberg)
[![Build status](https://ci.appveyor.com/api/projects/status/h4t9r6h5gom839q0/branch/master?svg=true)](https://ci.appveyor.com/project/Keats/gutenberg/branch/master)
[![Chat](https://img.shields.io/gitter/room/gitterHQ/gitter.svg)](https://gitter.im/gutenberg-rs/Lobby#)

An opinionated static site generator written in Rust.

## Installation
You can get the latest release by going to the [Release page](https://github.com/Keats/gutenberg/releases).
Alternatively, if you have the rust toolchain on your computer, you can also install it
through Cargo: `cargo install gutenberg`.

## Usage

### Creating a new site
Use `gutenberg init <a_directory_name>`. 
This will create a folder with the name given and the base structure of a gutenberg site.

### Working on a site
Use `gutenberg serve` to spin up a server that will automatically live reload any changes to the 
content, templates or static files.

### Building a site
Use `gutenberg build` to generate the site in the `public/` directory.

### Gutenberg terms
Some words are going to be repeated in the docs so let's make sure they are clear.

- Page: a markdown file in the `content` directory that has a name different from `_index.md`
- Section: a group of pages in the `content` directory that has `_index.md` in the same folder

### Configuration
Configuration is using the [TOML](https://github.com/toml-lang/toml) language.
Only 2 parameters are required: `title` and `base_url`.
The other options are:

- `highlight_code`: Whether to highlight all code blocks found in markdown files. Defaults to false
- `highlight_theme`: Which themes to use for code highlighting. Defaults to "base16-ocean-dark"
- `language_code`: The language used in the site. Defaults to "en"
- `generate_rss`: Whether to generate RSS, defaults to false
- `generate_tags_pages`: Whether to generate tags and individual tag pages if some pages have them. Defaults to true
- `generate_categories_pages`: Whether to generate categories and individual category categories if some pages have them. Defaults to true

If you want to add some of your own variables, you will need to put them in the `[extra]` table in `config.toml` or
they will be silently ignored.

### Templates
Templates live in the `templates/` directory and the files need to end by `.html`.
Only [Tera](https://github.com/Keats/tera) templates are supported.

Each kind of page get their own variables:

// TODO: detail the schema of the variables

- index.html: gets `section` representing the index section and all `sections`
- page.html: gets `page` that contains the data for that page 
- section.html: gets `section` that contains the data for pages in it and its subsections
- tags.html: gets `tags`
- tag.html: gets `tag` and `pages`
- categories.html: gets `categories`
- category.html: gets `category` and `pages`

Additionally, all pages get a `config` variable representing the data in `config.toml`, `current_url` that represent
the absolute URL of the current page and `current_path` that represents the path of the URL of the current page, starting with `/`.
If you want to know all the data present in a template content, simply put `{{ __tera_context }}`
in the templates and it will print it.

### Static files
Everything in the `static` folder will be copied into the output directory as-is.

### Pages
Pages have to start with a front-matter enclosed in `+++`. Here is a minimal example:

```md
+++
title = "My page"
description = "Some meta info"
+++

A simple page with fixed url
```

A front-matter has only optional variables:

- title
- description
- date: a YYYY-MM-DD or RFC339 formatted date
- slug: what slug to use in the url
- url: this overrides the slug and make this page accessible at `{config.base_url}/{url}`
- tags: an array of strings
- category: only one category is allowed
- draft: whether the post is a draft or not
- template: if you want to change the template used to render that specific page

Even if your front-matter is empty, you will need to put the `+++`.
You can also, like in the config, add your own variables in a `[extra]` table.
The front-matter will be accessible in templates at the `page.extra` field.

By default, the URL of a page will follow the filesystem paths. For example, if you have
a page at `content/posts/python3.md`, it will be available at `{config.base_url}/posts/python3/`.
You can override the slug created from the filename by setting the `slug` variable in the front-matter.

Quite often, a page will have assets and you might want to co-locate them with the markdown file.
Gutenberg supports that pattern out of the box: you can create a folder, put a file named `index.md` and any number of files
along with it that are NOT markdown.
Those assets will be copied in the same folder when building so you can just use a relative path to use them.

A summary is only defined if you put `<!-- more -->` in the content. If present in a page, the summary will be from
the start up to that tag.

### Sections
Sections represent a group of pages, for example a `tutorials` section of your site.
Sections are only created in Gutenberg when a file named `_index.md` is found in the `content` directory.

This `_index.md` file needs to include a front-matter as well, but won't have content:

```md
+++
title = "Tutorials"
+++
```
You can also set the `template` variable to change which template will be used to render that section.

Sections will also automatically pick up their subsections, allowing you to make some complex pages layout and
table of contents.

You can define how a section pages are sorted using the `sort_by` key in the front-matter. The choices are `date`, `order`
and `none` (default). Pages that can't be sorted will currently be silently dropped: the final page will be rendered but it will not appear in 
the `pages` variable in the section template.

A special case is the `_index.md` at the root of the `content` directory which represents the homepage. It is only there
to control pagination and sorting of the homepage.

You can also paginate section, including the index by setting the `paginate_by` field in the front matter to an integer. 
This represents the number of pages for each pager of the paginator. 
You will need to access pages through the `paginator` object. (TODO: document that).

### Code highlighting themes
Code highlighting can be turned on by setting `highlight_code = true` in `config.toml`.

When turned on, all text between backticks will be highlighted, like the example below.

```rust
let site = Site::new();
```

If the name of the language is not given, it will default to plain-text highlighting.

Gutenberg uses Sublime Text themes for syntax highlighting. It comes with the following theme
built-in:

- base16-ocean-dark
- base16-ocean-light
- gruvbox-dark
- gruvbox-light
- inspired-github
- kronuz
- material-dark
- material-light
- monokai
- solarized-dark
- solarized-light

### Internal links
You can have internal links in your markdown that will be replaced with the full URL when rendering.
To do so, use the normal markdown link syntax, start the link with `./` and point to the `.md` file you want
to link to. The path to the file starts from the `content` directory.

For example, linking to a file located at `content/pages/about.md` would be `[my link](./pages/about.md)`.

### Anchors
Headers get an automatic id from their content in order to be able to add deep links. By default no links are actually created but
the `insert_anchor_links` option in `config.toml` can be set to `true` to link tags. The default template is very ugly and will need
CSS tweaks in your projet to look decent. The default template can also be easily overwritten by creating a `anchor-link.html` file in
the `templates` directory.

### Shortcodes
Gutenberg uses markdown for content but sometimes you want to insert some HTML, for example for a YouTube video.
Rather than copy/pasting the HTML around, Gutenberg supports shortcodes, allowing you to define templates using Tera and call those templates inside markdown.

#### Using a shortcode
There are 2 kinds of shortcodes: simple ones and those that take some content as body. All shortcodes need to be preceded by a blank line or they
will be contained in a paragraph.

Simple shortcodes are called the following way:

```markdown
{{ youtube(id="my_youtube_id") }}
```

Shortcodes with a body are called like so:

```markdown
{% quote(author="Me", link="https://google.com") %}
My quote
{% end %}
```

The shortcodes names are taken from the files they are defined in, for example a shortcode with the name youtube will try to render
the template at `templates/shortcodes/youtube.html`.

#### Built-in shortcodes
Gutenberg comes with a few built-in shortcodes:

- YouTube: embeds a YouTube player for the given YouTube `id`. Also takes an optional `autoplay` argument that can be set to `true`
if wanted
- Vimeo: embeds a Vimeo player for the given Vimeo `id`
- Gist: embeds a Github gist from the `url` given. Also takes an optional `file` argument if you only want to show one of the files.

#### Defining a shortcode
All shortcodes need to be in the `templates/shortcodes` folder and their files to end with `.html`.
Shortcodes templates are simple Tera templates, with all the args being directly accessible in the template.

In case of shortcodes with a body, the body will be passed as the `body` variable.


## Example sites

- [vincent.is](https://vincent.is): https://gitlab.com/Keats/vincent.is


## Adding syntax highlighting languages and themes
### Adding a syntax
Syntax highlighting depends on submodules so ensure you load them first:
```bash
$ git submodule update --init 
```
Gutenberg only works with syntaxes in the `.sublime-syntax` format. If your syntax
is in `.tmLanguage` format, open it in Sublime Text and convert it to `sublime-syntax` by clicking on
Tools > Developer > New Syntax from ... and put it at the root of `sublime_syntaxes`.

You can also add a submodule to the repository of the wanted syntax:

```bash
$ cd sublime_syntaxes
$ git submodule add https://github.com/elm-community/Elm.tmLanguage.git
```

Note that you can also only copy manually the updated syntax definition file but this means
Gutenberg won't be able to automatically update it.

You can check for any updates to the current packages by running:

```bash
$ git submodule update --remote --merge
```

And finally from the root of the repository run the following command:

```bash
$ cargo run --example generate_sublime synpack sublime_syntaxes sublime_syntaxes/newlines.packdump sublime_syntaxes/nonewlines.packdump
```

### Adding a theme
A gallery containing lots of themes at https://tmtheme-editor.herokuapp.com/#!/editor/theme/Agola%20Dark.
More themes can be easily added to gutenberg, just make a PR with the wanted theme added in the `sublime_themes` directory
and run the following command from the repository root:

```bash
$ cargo run --example generate_sublime themepack sublime_themes sublime_themes/all.themedump
```

You should see the list of themes being added.
