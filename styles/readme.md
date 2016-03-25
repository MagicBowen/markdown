# markdown-styles

Converts Markdown files to HTML, with over a dozen builtin themes. The new `v2.0` release includes a ton of enhancements!

Looking for something to generate a blog from Markdown files? Check out [ghost-render](https://github.com/mixu/ghost-render).

## Features

- `v2.3` adds one new feature: header hover anchor links. When you hover over a header, a hover anchor link appears to the side of the header. Clicking on that link or copying its URL produces a link to that specific location on the page. All built-in layouts support this feature by default.
- `v2.2` added Windows support (!)
- Includes 15+ ready-made CSS stylesheets for Markdown, see the bottom of the readme for screenshots.
- Reuse the stylesheets or use the `generate-md` tool to convert a folder of Markdown files to HTML using one of the built-in layouts or a custom layout.
- Completely static output is easy to host anywhere.
- Metadata support: Each file can include additional metadata in a header section, such as the page title and author name which can then be used in the layout.

### Layout features

- Built in support for code syntax highlighting via highlight.js; all layouts include a Github-style code highlighting theme by default.
- Built in table of contents generation from Markdown headings, fully customizable by replacing the `{{> toc}}` partial in custom layout.
- Built in header id and anchor generation for headings written in Markdown; all layouts support revealing the URL via header hover links.
- Support for custom logic for rendering code blocks via `--highlight-*`; this can be used to implement custom blocks that render the content of the code block in some interesting way.
- Automatically detects the document title from the first heading in the Markdown markup.

### Features for creating your own layout

- To make it easier to get started, you can export an existing layout using `--exports` and use that as a starting point for your layouts.
- Create your own layout based on an existing layout via `--layout` with:
  - Full [Handlebars](http://handlebarsjs.com/) support for layouts, helpers and partials
  - Fully customizable table of contents template via the `toc` partial
  - Support for relative path generation via the `{{asset 'path'}}` helper
- API support: `markdown-styles` now has a public API

For changes, see [the changelog](changelog.md).

-----

## Quickstart

Install `generate-md` via npm (to get npm, just [install Node.js](http://nodejs.org/download/)):

    sudo npm install -g markdown-styles

Create a markdown file and then convert it to html:

    mkdir input/
    echo "# Hello world\n YOLO" > input/index.md
    generate-md --layout github --input ./input --output ./output
    google-chrome ./output/index.html

Try out different layouts by changing the `--layout` parameter; screenshots are at the bottom of this page.

![montage](https://github.com/mixu/markdown-styles/raw/master/screenshots/montage.png)

## generate-md CLI options

- `--input <path>` specifies the input directory (default: `./input/`).
- `--output <path>` specifies the output directory (default: `./output/`).
- `--layout <path>` specifies the layout. It can be:
  - The name of a builtin layout, such as `github` or `mixu-page`.
  - A path to a layout folder (full path, or a path relative to `process.cwd`).
  - A layout folder consists of:
    - `./page.html`, the template to use in the layout
    - `./assets`, the assets folder to copy over to the output
    - `./partials`, the [partials](#partials) directory
    - `./helpers`, the [helpers](#helpers) directory
- `--export <name>`: Exports a built-in layout to a directory. Use `--output <path>` to specify the location to write the built-in layout. For example, `--export github --output ./custom-layout` will copy the `github` builtin layout to `./custom-layout`.
- `--highlight-<language> <module>`: Specifies a custom highlighter module to use for a specific language. For example, `--highlight-csv mds-csv` will highlight any `csv` code blocks using the `mds-csv` module.
- `--no-header-links`: If this flag is passed, the HTML for header links will not be generated. The hover links are enabled by default.

## The resulting output

The output HTML is fully static and uses relative paths to the asset files, which are also copied into the output folder. This means that you could, for example, point a HTTP server at the output folder and be done with it or push the output folder to Amazon S3.

For example, here is how I deploy one of my books: `aws s3 sync ./output/ s3://some-s3-bucket/some-folder/ --delete --exclude "node_modules/*"  --exclude ".git"` (assuming credentials are in the necessary environment variables and that the AWS CLI is installed).

## Syntax highlighting

`v2.0` has syntax highlighting enabled by default. Every layout has also been updated to include a default [highlight.js](https://highlightjs.org/) syntax highlighting theme, which means everything works out of the box. For more highlighter themes, [check out this demo site](https://highlightjs.org/static/demo/) - you can find the [highlight.js CSS styles here](https://github.com/isagalaev/highlight.js/tree/master/src/styles).

To enable language-specific syntax highlighting, you need to specify the language of the code block, e.g.:

    ```js
    var foo = 'bar';
    ```

`v2.0` also supports additional language specific syntax highlighters - check out [mds-csv](https://github.com/mixu/mds-csv) for an example of a syntax highlighter for a specific language.

To enable additional language-specific syntax highlighters, install the module (e.g. `mds-csv`), then add `--highlight-{languagename} {modulename}` to the command line. For example, `generate-md --highlight-csv mds-csv ...` to enable the CSV highlighter for `csv` code blocks.

## Table of contents

The following built in layouts include the `{{~> toc}}` partial:

- mixu-book
- mixu-bootstrap-2col
- mixu-gray
- mixu-radar

These are mostly templates that have a sensible place to put this table of contents, such as a sidebar. I didn't want to default to putting a table of contents into the layouts that had no sidebar, but you can add it quite easily.

The `{{~> toc}}` partial generates a table of contents list. The list contains links to every header in your Markdown file. In addition, every Markdown header is automatically converted to a linkable anchor (e.g. `#table_of_contents`) when the page is generated. You can customize the table of contents markup by overriding the [./partials/toc.hbs](https://github.com/mixu/markdown-styles/blob/master/builtin/partials/toc.hbs) partial in your custom layout.

## Header hover links (v2.1)

If you are reading this on Github, hover over the header above. You'll see a link appear on the side of the header. The same feature is supported by all of the layouts. The feature is implemented purely with CSS, and you can find the details in `pilcrow.css` in each layout's assets folder. To disable the feature, pass the `--no-header-links` flag.

## Metadata sections

Each markdown file can have metadata associated with it. To set the metadata, start your markdown file with a metadata block that looks like this:

```
title: Page title
---
# Hello world
YOLO
```

There must be at least three `-` characters that separate the header from the rest of the content (on a single line).

You can reference the metadata values in your template by name. The default layouts only make use of the `{{title}}` metadata value, but your custom layouts can refer to any additional fields you want.

`{{title}}` is used as the page title. If you do not set the value explicitly, it is automatically detected from the first heading in the markdown file.

The metadata can also be written using JSON syntax or YAML syntax. This makes it possible to add arrays and hashes in the metadata. Using [handlebars.js](https://github.com/wycats/handlebars.js) you can go even further. For example, you can add a tags array into the metadata section:

```
title: Page title
tags: ["handlebars", "template"]
---
# Hello world
```

... which can then be [iterated over using the standard Handlebars `{{#each}}` iterator](http://handlebarsjs.com/block_helpers.html):

```html
<ul>
{{#each tags}}
    <li>{{ this }}</li>
{{/each}}
</ul>
```

which will result in:

```html
<ul>
    <li>handlebars</li>
    <li>template</li>
</ul>
```

If you take a look at [the `{{~> toc}}` built in partial](https://github.com/mixu/markdown-styles/blob/master/builtin/partials/toc.hbs), you can see that it is actually iterating over a metadata field called `headings` using the same syntax. The `headings` metadata is an array of objects with an `id` field (the HTML anchor id), a `text` field (the heading text) and a `depth` field (the depth of the heading, e.g. the number of `#` characters in the heading).

## Writing your own layout

`v2.0` makes it easier to get started with a custom layout via `--export`, which exports a built in layout as a starting point. Just pick a reasonable built in layout and start customizing. For example:

    generate-md --export github --output ./my-layout

will export the `github` layout to `./my-layout`. To make use of your new layout:

    generate-md --layout ./my-layout --input ./some-input --output ./output

If you look under `./my-layout`, you'll see that a layout folder consists of:

- `./page.html`, the template to use in the layout
- `./assets`, the assets folder to copy over to the output
- `./partials`, the [partials](#partials) directory
- `./helpers`, the [helpers](#helpers) directory

See the next few sections for more details for how these features work.

### Template Evaluation (page.html)

The [handlebars.js](https://github.com/wycats/handlebars.js) template language is used to evaluate both the template and the markdown.

Here is a list of all the built in features:

- `{{~> content}}`: renders the markdown content
- `{{asset 'asset-path'}}`: renders a specific asset path
- `{{~> toc}}`: renders the table of contents
- `{{title}}`: renders the title from the metadata section

Any metadata fields you have defined in the page's metadata section can be referenced in `page.html` by name. For example, `{{title}}` is replaced with the value of the `title` metadata field when the template is rendered.

You can include your own helpers and partials in your custom layout as shown below.

### Assets folder (./assets)

All files in the assets folder are copied from the layout folder to the output folder.

To refer to files in the assets folder, use the `{{asset 'path'}}` helper. For example, `{{asset 'css/style.css'}}` will be replaced with a relative path to the file in `./assets/css/style.css`. Take a look at the built in layouts for some examples.

### Partials

Partials are html files that can be included via handlebars `{{> partialName}}` style. Usually they are .html files. For example, if `footer.html` resides in the partials directory, `{{> footer}}` will be replaced with `footer.html`'s content. For more advanced topics, see [handlebars partials documentation](https://github.com/wycats/handlebars.js#partials). Don't use `content.html`, it is reserved to the html generated from the markdown. You can override the `toc` partial by adding `./partials/toc.html` as a partial in your custom layout, e.g.

```html
<h1>My Table of Contents</h1>
<ul class="nav nav-list">
  {{#each headings}}
    <li><a href="#{{id}}">{{text}}</a></li>
  {{/each}}
</ul>
```

### Helpers

Helpers are functions that you can use throughout the template. See [handlebars helpers](https://github.com/wycats/handlebars.js#registering-helpers).
For example, add `linkTo.js` to the `./helpers` directory in your custom layout:

```js
var Handlebars = require('handlebars');
module.exports = function(){
  return new Handlebars.SafeString("<a href='" + Handlebars.Utils.escapeExpression(this.url) + "'>" + Handlebars.Utils.escapeExpression(this.body) + "</a>");
};
```

Next, in `./my-layout`, run `npm install handlebars` (since we're requiring handlebars) in the code.

In your metadata heading:

```
links:
  - url: "/hello"
    body: "Hello"
  - url: "/world"
    body: "World!"
---
# Hello world
```
or:

```
links: [ { url: "/hello", body: "Hello"},
         { url: "/world", body: "World!" } ]
---
# Hello world
```
and somewhere in your template:

```html
<ul>{{#links}}<li>{{{linkTo}}}</li>{{/links}}</ul>
```

Note the usage of the "triple-stash", e.g. `{{{` here. The technical reason for this is documented [in this issue in Handlebars](https://github.com/wycats/handlebars.js/issues/886) and will be apparently fixed in Handlebars `3.0`. For now, use triple-stash to invoke any helpers that generate HTML.

... will result in:

```html
<ul>
  <li>
    <a href='/hello'>Hello</a>
  </li>
  <li>
    <a href='/world'>World!</a>
  </li>
</ul>
```

### `meta.json`

If you want to apply additional metadata to all Markdown files in a particular folder, you can add a file named `meta.json` to the input folder. (Note: in `v1.x`, `meta.json` was read from `process.cwd()`, e.g. the folder from which you ran `generate-md`).

`meta.json` will be read, and the values in it are added to the regular metadata. The values from `meta.json` act like default values, e.g. the per-file metadata section values will override the values from `meta.json`.

The metadata is scoped by the top-level directory in `./input`.

For example, a `./input/meta.json` file like this:

````json
{
  "foo": {
    "repoUrl": "https://github.com/mixu/markdown-styles"
  }
}
````

would make the metadata value `{{repoUrl}}` available in the template, for all files that are in the directory `./input/foo`. If any markdown file in `./input/foo/` defines a metadata value called `repoUrl`, then that value will override the value from `meta.json`.

### API

It exists, and uses the same options as `generate-md`. Docs TODO, see `bin/generate-md` and `test/api.test.js` for now.

## Acknowledgments

I'd like to thank @AaronJan for contributing a patch that adds support for Windows (for `v.2.2.0`+) and @joebain for a fix related to using markdown-styles with grunt.

I'd like to thank the following people for either contributing to markdown-styles directly or making CSS stylesheets available with a permissive open source license:

- the `witex` style is based on [AndrewBelt/WiTeX](https://github.com/AndrewBelt/WiTeX)
- @iamdoron for contributing the initial implementation of the Handlebars templating integration
- the `github` style is based on [sindresorhus/github-markdown-css](https://github.com/sindresorhus/github-markdown-css)
- the `bootstrap3` style was contributed by @MrJuliuss
- jasonm23-dark, jasonm23-foghorn, jasonm23-markdown and jasonm23-swiss are based on https://github.com/jasonm23/markdown-css-themes by [jasonm23](https://github.com/jasonm23)
- thomasf-solarizedcssdark and thomasf-solarizedcsslight are based on https://github.com/thomasf/solarized-css by [thomasf](https://github.com/thomasf)
- markedapp-byword is based on the user-contributed stylesheet at http://bywordapp.com/extras/
- roryg-ghostwriter is based on https://github.com/roryg/ghostwriter
- github is based on [sindresorhus/github-markdown-css](https://github.com/sindresorhus/github-markdown-css) (sorry, sindresorhus-github is too long to type as a layout name!)

## Screenshots of the layouts

Note: these screenshots are generated via phantomjs, so they look worse than they do in a real browser because the font rendering is just bad and lacks webfont support. For example, WiTeX actually uses the Latin Modern Roman font from TeX but the screenshots show the fallback font.

### github

![github](https://github.com/mixu/markdown-styles/raw/master/screenshots/github.png)

### witex

![witex](https://github.com/mixu/markdown-styles/raw/master/screenshots/witex.png)

### roryg-ghostwriter

![roryg-ghostwriter](https://github.com/mixu/markdown-styles/raw/master/screenshots/roryg-ghostwriter.png)

### mixu-bootstrap

![mixu-bootstrap](https://github.com/mixu/markdown-styles/raw/master/screenshots/mixu-bootstrap.png)

### mixu-bootstrap-2col

![mixu-bootstrap-2col](https://github.com/mixu/markdown-styles/raw/master/screenshots/mixu-bootstrap-2col.png)

### mixu-gray

![mixu-gray](https://github.com/mixu/markdown-styles/raw/master/screenshots/mixu-gray.png)

### jasonm23-dark

![jasonm23-dark](https://github.com/mixu/markdown-styles/raw/master/screenshots/jasonm23-dark.png)

### jasonm23-foghorn

![jasonm23-foghorn](https://github.com/mixu/markdown-styles/raw/master/screenshots/jasonm23-foghorn.png)

### jasonm23-markdown

![jasonm23-markdown](https://github.com/mixu/markdown-styles/raw/master/screenshots/jasonm23-markdown.png)

### jasonm23-swiss

![jasonm23-swiss](https://github.com/mixu/markdown-styles/raw/master/screenshots/jasonm23-swiss.png)

### markedapp-byword

![markedapp-byword](https://github.com/mixu/markdown-styles/raw/master/screenshots/markedapp-byword.png)

### mixu-book

![mixu-book](https://github.com/mixu/markdown-styles/raw/master/screenshots/mixu-book.png)

### mixu-page

![mixu-page](https://github.com/mixu/markdown-styles/raw/master/screenshots/mixu-page.png)

### mixu-radar

![mixu-radar](https://github.com/mixu/markdown-styles/raw/master/screenshots/mixu-radar.png)

### thomasf-solarizedcssdark

![thomasf-solarizedcssdark](https://github.com/mixu/markdown-styles/raw/master/screenshots/thomasf-solarizedcssdark.png)

### thomasf-solarizedcsslight

![thomasf-solarizedcsslight](https://github.com/mixu/markdown-styles/raw/master/screenshots/thomasf-solarizedcsslight.png)

### bootstrap3

![bootstrap3](https://github.com/mixu/markdown-styles/raw/master/screenshots/bootstrap3.png)

## Contributing new styles to markdown-styles

Add new layouts to `./layouts/name`. To regenerate the pages, you need to run:

    git clone git://github.com/mixu/markdown-styles.git
    npm install
    make build

To regenerate the screenshots, you need cutycapt (or some other Webkit to image tool) and imagemagic. On Ubuntu / Debian, that's:

    sudo aptitude install cutycapt imagemagick

You also need to install the web fonts locally so that cutycapt will find them, run `node font-download.js` to get the commands you need to run (basically a series of wget and fc-cache -fv commands).

Finally, run:

    make screenshots

If you have phantomjs installed, run:

    make phantomjs

which will use a phantomjs script to capture the screenshots.
