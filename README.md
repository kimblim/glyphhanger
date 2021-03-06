# glyphhanger

Utility to automatically subset fonts for serving on the web. Can generates a combined list of every glyph used on a list of sample files or urls.

## Installation

```
npm install -g glyphhanger
```

## Usage

### Find the glyphs in a local file or url

```
# local file
> glyphhanger ./test.html
> glyphhanger ./test.txt

# output Unicode code points (this will be the default mode in v2.0)
> glyphhanger ./test.html --unicodes

# remote URL
> glyphhanger http://example.com

# multiple URLs, optionally using HTTPS
> glyphhanger https://google.com https://www.filamentgroup.com

```

### Subset font files automatically

Use `--subset=*.ttf` to select some font files for subsetting. Note that you can also [subset yourself manually with `pyftsubset`](docs/manual-subset.md), although I’m not sure why you would.

```
> glyphhanger ./test.html --subset=*.ttf

Subsetting LatoLatin-Regular.ttf to LatoLatin-Regular-subset.ttf (was 145.06 KB, now 24 KB)
Subsetting LatoLatin-Regular.ttf to LatoLatin-Regular-subset.zopfli.woff (was 145.06 KB, now 14.34 KB)
Subsetting LatoLatin-Regular.ttf to LatoLatin-Regular-subset.woff2 (was 145.06 KB, now 11.37 KB)

# Subset to specific characters only (no URLs)
> glyphhanger --whitelist=ABCD --subset=*.ttf

Subsetting LatoLatin-Regular.ttf to LatoLatin-Regular-subset.ttf (was 145.06 KB, now 4.42 KB)
Subsetting LatoLatin-Regular.ttf to LatoLatin-Regular-subset.zopfli.woff (was 145.06 KB, now 2.84 KB)
Subsetting LatoLatin-Regular.ttf to LatoLatin-Regular-subset.woff2 (was 145.06 KB, now 2.24 KB)

# Specify the formats to create (Available: ttf,woff,woff-zopfli,woff2)
> glyphhanger --whitelist=ABCD --formats=woff2,woff --subset=*.ttf

Subsetting LatoLatin-Regular.ttf to LatoLatin-Regular-subset.woff (was 145.06 KB, now 2.88 KB)
Subsetting LatoLatin-Regular.ttf to LatoLatin-Regular-subset.woff2 (was 145.06 KB, now 2.24 KB)
```

### Whitelist Characters

```
# whitelist specific characters
> glyphhanger https://google.com --whitelist=abcdefgh

# shortcut to whitelist all of US-ASCII
> glyphhanger https://google.com --US_ASCII

# use both together
> glyphhanger https://google.com --whitelist=™ --US_ASCII

# use without a URL (useful for manual subsetting)
> glyphhanger --whitelist=ABCD --subset=*.ttf
```

### Use the spider to gather URLs from links

Finds all the `<a href>` elements on the page with *local* (not external) links and adds those to the glyphhanger URLs. If you specify `--spider-limit`, `--spider` is assumed.

```
> glyphhanger ./test.html --spider
> glyphhanger ./test.html --spider-limit=10

# No limit
> glyphhanger ./test.html --spider-limit=0
```

Default `--spider-limit` is 10. Set to `0` for no limit. This will greatly affect how long the task takes.

### Additional options

```
# verbose mode
> glyphhanger https://google.com --verbose

# version
> glyphhanger --version
```

### Installing `pyftsubset`

See [https://github.com/fonttools/fonttools](https://github.com/fonttools/fonttools).

```
pip install fonttools

# Additional installation for --flavor=woff2
git clone https://github.com/google/brotli
cd brotli
python setup.py install

# Additional installation for --flavor=woff --with-zopfli
git clone https://github.com/anthrotype/py-zopfli
cd py-zopfli
git submodule update --init --recursive
python setup.py install
```


## Testing

GlyphHanger uses Mocha for testing.

`npm test` will run the tests.

Or, alternatively:

`./node_modules/mocha/bin/mocha` (or just `mocha` if you already have it installed globally with `npm install -g mocha`).

## Limitations

* Pseudo-element CSS `content` is not parsed.

## Example using Unicode code points

```
> glyphhanger https://www.zachleat.com/web/ --unicodes --spider --spider-limit=5 > glyphhanger_zachleat_output

> cat glyphhanger_zachleat_output
 !$&()+,-./0123456789:?@ABCDEFGHIJLMNOPQRSTUVWXYZ[]abcdefghijklmnopqrstuvwxyz»é–—’“”→★

> pyftsubset sourcesanspro-regular.ttf --unicodes-file=glyphhanger_zachleat_output --flavor=woff
# Reduced the 166KB .ttf font file to an 8KB .woff webfont file.
``` 

You can exclude `--unicodes` to use String values instead, but first _(read [Issue #4](https://github.com/filamentgroup/glyphhanger/issues/4)  on why Unicode code points are better)_
