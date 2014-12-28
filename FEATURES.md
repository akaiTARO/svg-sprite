Old features
============

* Support custom output formats along with some default ones (CSS, Sass, LESS, Stylus)
* For the inline sprite variants: Provide optional example HTML files


Changes
=======

* Split out all file access methods so that they need not necessarily be used, making the module a good basis for 3rd party tools (like Grunt & Gulp plugins)
* Make the definition of output locations and templates more intuitive
* Drop `libxmljs` in favour of `cheerio` to get rid of the Python dependency (for Windows users)
* Add symlink support ([Issue #25](https://github.com/jkphl/svg-sprite/issues/25))
* Add cache busting ([Pull request #29](https://github.com/jkphl/svg-sprite/pull/29))
* Plugin-like selection of the SVG cleaner(s) to be used


Enhancements
============

* Support several sprite styles, output variants
	* __sprite__: Vertically, horizontally, diagonally or [packed](http://codeincomplete.com/posts/2011/5/7/bin_packing/) (NEW) `<svg>` elements within the sprite
	* __defs__: Using `<defs>` elements for inline embedding (see [CSS Tricks](http://css-tricks.com/svg-sprites-use-better-icon-fonts/))
	* __symbol__: Using `<symbol>` elements for inline embedding (see [CSS Tricks](http://css-tricks.com/svg-symbol-good-choice-icons/))
	* __stack__: Using [fragment identifiers plus JavaScript](http://hofmannsven.com/2013/laboratory/svg-stacking/) ([Support](http://caniuse.com/#feat=svg-fragment))
	* __view__: Using [fragment identifiers to identify predefined views](http://24ways.org/2014/an-overview-of-svg-sprite-creation-techniques/) ([Support](http://caniuse.com/#feat=svg-fragment))
	* __picture__: Using the new [picture element](http://css-tricks.com/svg-fallbacks/) ([Support](http://caniuse.com/#search=picture))
* Let people customize the CSS icon and dimension class names respectively the icon IDs when in inline mode (`defs` / `symbol`)
* Support [accessibility features](http://www.sitepoint.com/tips-accessible-svg/)
	* Support the `<title id="xxx-title">` element + `aria-labelledby="xxx-title"`
	* Support the `<desc id="xxx-desc">` element + `aria-describedby="xxx-desc"` (or `aria-labelledby="xxx-title xxx-desc"`)
	* Support the `role="image"` attribute on inline `<svg>` elements
	* Support for an external file (e.g. `sprite.json` or `icons.json`) to define these accessibility values
* Support an inline fallback (see below)
* Maybe add [`margin` support](https://github.com/jkphl/svg-sprite/pull/24)?
* Make the background sprites being [positioned percentually](https://github.com/jkphl/grunt-svg-sprite/issues/23)? (See the [example page](examples/html/sprite/weather-svg.html))
* Maybe add [coloring feature](https://github.com/jkphl/grunt-svg-sprite/issues/22)?
* Make `padding` (and optionally `margin` as well) [twofold values](https://github.com/jkphl/grunt-svg-sprite/issues/20#issue-41484517)
* Support Windows (several bugs like [#14](https://github.com/jkphl/grunt-svg-sprite/issues/14))
* For real sprites, add an [alignment option](https://github.com/jkphl/grunt-svg-sprite/issues/16)
* Add pre-, main and post-transformation hooks for SVG files
* [WebP](https://developers.google.com/speed/webp/docs/using) fallbacks for capable browsers (Chrome & Opera)?


Processing steps
================
1. *Read in source files (not part of main functionality)*
2. Read in configuration options
3. Instanciate and configure delegate components
4. Process & prepare all source files
	1. Custom post-transformation
	2. Clean & optimize SVG file (with possibly several optimizers)
	3. Determine / sanitize SVG dimensions
	4. Add padding
	5. Namespace all IDs inside an SVG
	6. Custom post-transformation
5. Compose sprite(s) depending on activated modes
6. Generate stylesheet resources depending on activated modes
7. Return all generated files (optimized SVG files, SVG sprites, stylesheet resources, example resources)
8. *Create the necessary directory structure & write out all generated files (not part of main functionality)*


Module components
=================
* Main component
* Logging subcomponent
* SVG Pre- & post-transformation delegates
* SVG Optimizer delegates
* Sprite composer
* Stylesheet renderer
* Binary wrapper


PNG fallback for inline SVG (iconizr)
=====================================
See [this CSS tricks article](http://css-tricks.com/svg-fallbacks/) for an overview of different fallback approaches, and see this [`<use>` specific fallback](http://developersdev.blogspot.ru/2014/10/svg-fallback-pure-css-can-i-use-part-5.html) and [this](http://davidensinger.com/2013/04/inline-svg-with-png-fallback/) for inspiration:

```html
<svg class="icon">
	<switch>
		<use xlink:href="#shape-icon-1" />
		<foreignObject>
			<div class="fallback"></div>
		</foreignObject>
	</switch>
</svg>
```

Fallback logic for inline SVG
-----------------------------
There are several aspects influencing the most appropriate fallback:
* JavaScript support
* [SVG support](http://caniuse.com/#feat=svg)
* [Inline SVG support](http://caniuse.com/#feat=svg-html5)

General and inline SVG support is roughly the same across all major browsers (only Opera Mini doesn't support inline SVG in contrast to it's general SVG support).

Fact is, that without JavaScript it's not possible to determine a client's exact extent of SVG support. Thus the **ultimate fallback** should work **without JavaScript** and **not rely on SVG support** to display images. Furthermore, a proper fallback depends on the way the SVG images are to be used.

### Background mode

In this mode, images are being **set by CSS**, so the fallback consists of serving the appropriate CSS leveraging the client's SVG and / or dataURI capabilities. This is basically what iconizr does. As feature detection relies on JavaScript, the ultimate fallback consists of a `<noscript>` element wrapping the very simplest and most foolproof CSS flavour (PNG sprite).

### Defs / symbols mode

Both `<defs>` and `<symbols>` mode rely on **inline SVG** in conjunction with the `<use>` element and a special "SVG library sprite" (the sprites for the two modes differ from each other). Accordingly, a fallback has to be embedded inline as well and

* must be used whenever a client doesn't support (inline) SVG,
* must not be used when the client supports SVG but doesn't support JavaScript (which could help disabling the fallback),
* must not be downloaded in addition to the SVG sprite.

A possible fallback could combine [this](http://developersdev.blogspot.ru/2014/10/svg-fallback-pure-css-can-i-use-part-5.html) approach with the standard background mode. (?) 

### Stacking mode

SVG stacking uses fragment identifiers to display a specific portion of a special "SVG library sprite" (another special variant). The SVG sprite could be referenced by both `<img>` as well as `<object>` elements. At least in the latter case there should be a native fallback by wrapping a regular `<img>` element inside the `<object>` (not sure about double downloads then though).

What about `<picture>`?
-----------------------

The new `<picture>` element allows for content-type fallbacks like this:

```html
<picture>
  <source srcset="graph.svg" type="image/svg+xml">
  <img srcset="graph-small.png, graph-medium.png 400, graph-large.png 800" alt="A lovely graph.”>
</picture>
```

However, `<picture>` support is still rather unusual, so one would need a JavaScript based polyfill for that ...