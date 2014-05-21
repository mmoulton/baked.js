baked.js
========

baked.js is a static website generator, using Node.js, which embed an updated version of the precedent JS script.

## Install

To use it, you first need to install its dependencies

```sh
git clone https://github.com/prismicio/baked.js.git
cd baked.js
npm install
```

## Use

Run baked.js with the command:

```sh
node generate.js <src_dir> <dst_dir>
```

It displays a lot of informations, explaining what it's doing, which file it's trying to render, and logs errors that occur.

**Tips**: you can use the `--no-async` argument to make the displayed informations more readable. It will run slower though.

When it's finished, just copy the generated files into your favorite HTTP server (you can open them directly in your browser if you wish).

## Template

Lodash's template can be used, with syntax “`[% %]`” (and “`[%= %]`”, “`[%- %]`”).

Here are the variables passed to the template:

- `_`: Lodash
- `api`: The main API object
- `bookmarks`: The bookmarks list
- `ref`: The current ref
- `refs`: The refs list
- `types`: The types list
- `tags`: The tags list
- `master`: The master ref

```html
[% _.each(tags, function (tag) { %]
  <span>[%= tag %]</span>
[% }) %]
```

## Queries

Queries (to prismic.io repositories) can be written directly in the template, using specify `<script>` tags (with `type="text/prismic-query"`).

The documents returned by the queries are passed to the templated through the variable whose name is specified with `data-binding` attribute.

Pagination related parameters can be specified using `data-query-<name>` syntax.

```html
<script type="text/prismic-query" data-binding="product" data-query-orderings="[my.product.name]">
  [
    [:d = any(document.type, ["product"])]
    [:d = at(document.tags, ["Featured"])]
  ]
</script>

<h1>[%= featuredProducts.length %] featured products:</h1>
[% _.each(featuredProducts, function(product) { %]
  <div>
    <h2>[%= product.getText('product.name') %]</h2>
    <img data-src="[%= product.getImageView('product.image', 'icon').url %]">
  </div>
[% }) %]
```

## Links

To create links to an other generated page, use the helper `url_to`, and specify the file name (without the “.html” part).

```html
<a href="[%= url_to('search') %]">[%= product.getText('product.name') %]</a>
```

## Page parameters

Some pages (like articles of a blog) can have dynamically generated URL. This can be done by creating one file and specify parameter, like the ID of the article.

The problem is that these parameters are generally not known before parsing other pages linking to them. For instance the main blog page lists some articles (displaying previews) and gives links to the full article pages. That's how we know that there are articles of these specific IDs.

In baked.js, we use linking pages to infer that there are articles to be generated with these specific IDs. In short, we need to know that a page is needed in order to be able to create it!

First, add a `<meta>` tag per parameter in the page's header.

```html
<meta name="prismic-routing-param" content="id">
```

Then use these parameters in your query, by using the syntax `$name` or `${name}.`

```html
<script type="text/prismic-query" data-binding="product">
  [
    [:d = any(document.id, ["$id"])]
  ]
</script>
```

To create links to the above page, use the helper `url_to`, and specify the arguments.

```html
<a href="[%= url_to('product', {id: product.id}) %]">
    [%= product.getText('product.name') %]
</a>
```

**Bonus**: if your only argument is “`id`”, you can give it directly, without wraping it in a “`{id: "xxx"}`” structure.

```html
<a href="[%= url_to('product', product.id) %]">
    [%= product.getText('product.name') %]
</a>
```

You can also use the helper without providing any argument.

```html
<a href="[%= url_to('index' %]">index</a>
```

**Note**: remember: if nobody call a page (using the `url_to` helper) it won't be generated.

### Custom URL

It is possible to customize the URL as well. To do so, just add a `<meta>` tag “`prismic-routing-pattern`” in your page's header.

```html
<meta name="prismic-routing-pattern" content="product/$id">
```

**Warning**: be sure to specify every routing params in the URL! If you don't, we can't guarantee that there won't be URL conflict.

## Dev mode

It's possible to directly open the sources file in your browser. The rendering lib will run as seen in the last blog article (+ the `url_to` helper is working).

## Internals

baked.js is built on top of [Node.js](nodejs.org) and use [dom.js](https://github.com/andreasgal/dom.js/) to emulate the DOM.

It uses [Q](https://github.com/kriskowal/q) and [lodash](http://lodash.com), and let [Gulp](gulpjs.com) and [browserify](browserify.org) handle the generation of the browser library.

## Notes

- page in dev mode don't have the global map of exiting pages, so:
	- the page a URL `http://host/foo/bar.html` could be
		- the file `foo/bar.html` (if the generated files are put at the root of `host`)
		- the file `bar.html` (if the generated files are put in the dir `foo` of `host`)
	- `url_to` have to be relative (how to link to a global file without knowning what is the origin point?)
	- the custom routes can't be honored, even simple rename (how can we know which route should be used without reading the called file?)
    - ⇒ **⚠ we're working to fix these points**
- the server build a DOM structure (using JSDOM) in order to navigate inside it, so:
	- The tags `<% %>` can't be used, because JSDOM doesn't like them
