
Example:

```js
const inliner = new CSSInliner({
  directory: 'stylesheets'
});

inliner.inlineAsync(html)
  .then(function(result) {
    console.log(result.inlined);
  });
```


## Configuring

First, create a new `CSSInliner` object.  The inliner object serves as a cache
for processed stylesheets, inlined documents, and other assets.  You don't have
to keep it around, but it is useful to speed up processing.

You configure the inliner with the following options:

`directory` - This is the base directory from which stylesheets are loaded.

`plugins`   - Array of PostCSS plugins.

`resolve`   - This is a function that resolves a stylesheet reference (path or
URL) into the filename of the stylesheet to load.

If the source document links to any external stylesheet with with a relative URL
(path only), then the inliner will attempt to resolve this stylesheet using the
supplied `resolve` function.

The most common configuration is for external stylesheets to exist in a known
location in the file system, and this can be set using the `directory` option
(instead, not in addition to, the `resolve` options).  Only files in that
directory or sub-directory are accessible.

If you don't specify either options, inliner will not be able to resolve
external stylesheets.  However, it can still process `style` elements appearing
in the document.


## How It Works

The inliner extracts any `style` elements appearing in the document, with all
their rules.  It also extracts any external stylesheets that use relative URLs.
Both stylesheets are processed using PostCSS and any plugins you opt to use, and
cached in memory.

The `style` element and `link` element to known stylesheet are then removed from
the document.  These rules will be inlined or added back into the document
later.

External stylesheets that reference absolute URLs (anything with a hostname,
such as `//example.com/`) are retained in the document.  These are expected to
resolve when the document is rendered in the browser or email client.

Styles are processed in document order, that is, the order in which the `style`
or `link` elements appear in the document, such that the early rules take
precedence (for the same specificity).

Rules that can be applied to the `style` attribute of an element (inlined), are
applied to any matching element found in the document, if there is one.  They
are always discarded.

Rules that cannot be applied, are kept as is, and added back to the document
inside a new `style` elements.  These become accessible to browsers or email
clients that refuse to load external stylesheets, but will still process `style`
elements included in the document (e.g. GMail).

Rules that cannot be inlined include pseudo selectors such as `:hover` and
`::after`, as well as all media queries such as `@media screen`.  These can only
be applied to a live document when rendered by a browser.

If a rule has multiple selectors, it may be inlined using one selector, and
included in the document with another selector.




## Working with Less/SASS/Stylus/etc

If you're working with a language that compiles to CSS, you need to use the
`precompile` option.

A precompiled for Less is included by default, and you can use it like this:

```js
const precompile  = CSSInliner.less;
const inliner     = new CSSInliner({ precompile });
```

(Less is an optional dependency, so you need to add it in your `package.json` if
you want to use it.)

The `precompile` option takes a function that will be called with two arguments:
the pathname, and the stylesheet.  You can use the pathname to determine the
file type based on its extension (e.g. does it end with `.less`?)

The function should return the compiled CSS in the form of a string or a Buffer,
or a promise that resolves to a string or Buffer.


## Working with templates (Handlebars, etc)

Inlining requires parsing the HTML document, and when there are non-HTML tags in
the document, they are often parsed incorrectly.  Many templating languages use
non-HTML tags.

Use the `template` option with an appropriate template parser.  For example,
when working with Handlebar templates:

```js
const template = CSSInliner.handlebars;
const inliner  = new CSSInliner({ template });
```

(Handlebars is an optional dependency, so you need to add it in your
`package.json` if you want to use it.)

The template handler is a function that will be called with the source template,
and must return an array of all template tags found there.

These tags are then replaced with markers, before parsing the HTML and inlining,
and are restored before resolving to the final HTML.


## References

[CSS 3: Calculating a selector's specificity](http://www.w3.org/TR/css3-selectors/#specificity)

[PostCSS API](https://github.com/postcss/postcss/blob/master/docs/api.md)

[PostCSS Selector Parser API](https://github.com/postcss/postcss-selector-parser/blob/master/API.md)

