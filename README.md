# clue/zlib-react [![Build Status](https://travis-ci.org/clue/php-zlib-react.svg?branch=master)](https://travis-ci.org/clue/php-zlib-react)

Streaming zlib compressor and decompressor, built on top of [React PHP](http://reactphp.org/),
supporting compression and decompression of the following formats:

* [RFC 1952](https://tools.ietf.org/html/rfc1952) (GZIP compressed format)
* [RFC 1951](https://tools.ietf.org/html/rfc1951) (raw DEFLATE compressed format)
* [RFC 1950](https://tools.ietf.org/html/rfc1950) (ZLIB compressed format)

> Note: This project is in beta stage! Feel free to report any issues you encounter.

**Table of contents**

* [Quickstart example](#quickstart-example)
* [Formats](#formats)
  * [GZIP format](#gzip-format)
  * [Raw DEFLATE format](#raw-deflate-format)
  * [ZLIB format](#zlib-format)
* [Usage](#usage)
  * [ZlibFilterStream](#zlibfilterstream)
    * [createCompressor()](#createcompressor)
    * [createDecompressor()](#createdecompressor)
    * [Inconsistencies](#inconsistencies)
* [Install](#install)
* [License](#license)
* [More](#more)

## Quickstart example

Once [installed](#install), you can use the following code to pipe a readable
gzip file stream into an decompressor which emits decompressed data events for
each individual file chunk:

```php
$loop = React\EventLoop\Factory::create();
$stream = new Stream(fopen('access.log.gz', 'r'), $loop);

$decompressor = ZlibFilterStream::createGzipDecompressor();

$decompressor->on('data', function ($data) {
    echo $data;
});

$stream->pipe($decompressor);

$loop->run();
```

See also the [examples](examples).

## Formats

This library is a lightweight wrapper around the underlying zlib library.
The zlib library offers a number of different formats (sometimes referred to as *encodings*) detailled below.

### GZIP format

This library supports the GZIP compression format as defined in [RFC 1952](https://tools.ietf.org/html/rfc1952).
This is one of the more common compression formats and is used in several places:

* PHP: `gzdecode()` (PHP 5.4+ only) and `gzencode()`
* Files with `.gz` file extension, e.g. `.tar.gz` or `.tgz` archives (also known as "tarballs")
* `gzip` and `gunzip` (and family) command line tools
* [HTTP compression](https://en.wikipedia.org/wiki/HTTP_compression) with `Content-Encoding: gzip` header
* Java: `GZIPOutputStream`

Technically, this format uses [raw DEFLATE compression](#raw-deflate-format) wrapped in a GZIP header and footer:

```
10 bytes header (+ optional headers) + raw DEFLATE body + 8 bytes footer
```

### Raw DEFLATE format

This library supports the raw DEFLATE compression format as defined in [RFC 1951](https://tools.ietf.org/html/rfc1951).
The DEFLATE compression algorithm returns what we refer to as "raw DEFLATE format".
This raw DEFLATE format is commonly wrapped in container formats instead of being used directly:

* PHP: `gzdeflate()` and `gzinflate()`
* Wrapped in [GZIP format](#gzip-format)
* Wrapped in [ZLIB format](#zlib-format)

> Note: This format is not the confused with what some people call "deflate format" or "deflate encoding".
These names are commonly used to refer to what we call [ZLIB format](#zlib-format).

### ZLIB format

This library supports the ZLIB compression format as defined in [RFC 1950](https://tools.ietf.org/html/rfc1950).
This format is commonly used in a streaming context:

* PHP: `gzcompress()` and `gzuncompress()`
* [HTTP compression](https://en.wikipedia.org/wiki/HTTP_compression) with `Content-Encoding: deflate` header
* Java: `DeflaterOutputStream`

Technically, this format uses [raw DEFLATE compression](#raw-deflate-format) wrapped in a ZLIB header and footer:

```
2 bytes header (+ optional headers) + raw DEFLATE body + 4 bytes footer
```

> Note: This format is often referred to as the "deflate format" or "deflate encoding".
This documentation avoids this name in order to avoid confusion with the [raw DEFLATE format](#raw-deflate-format).

## Usage

All classes use the `Clue\React\Zlib` namespace.

### ZlibFilterStream

The `ZlibFilterStream` is a small wrapper around the underlying `zlib.deflate` and `zlib.inflate`
stream compression filters offered via `ext-zlib`.

#### createCompressor()

The following methods can be used to create a compressor instance:

```php
$compressor = ZlibFilterStream::createGzipCompressor();
$compressor = ZlibFilterStream::createDeflateCompressor();
$compressor = ZlibFilterStream::createZlibCompressor();
```

#### createDecompressor()

The following methods can be used to create a decompressor instance:

```php
$decompressor = ZlibFilterStream::createGzipDecompressor();
$decompressor = ZlibFilterStream::createDeflateDecompressor();
$decompressor = ZlibFilterStream::createZlibDecompressor();
```

#### Inconsistencies

The stream compression filters are not exactly the most commonly used features of PHP.
As such, we've spotted several inconsistencies (or *bugs*) between different PHP versions and HHVM.
These inconsistencies exist in the underlying PHP engines and there's little we can do about this in this library.

* All Zend PHP versions: Decompressing invalid data does not emit any data (and does not raise an error)
* PHP 7 only: Compressing an empty string does not emit any data (not a valid compression stream)
* HHVM only: does not currently support the GZIP and ZLIB format at all (and does not raise an error)
* HHVM only: The [`zlib.deflate` filter function](https://github.com/facebook/hhvm/blob/fee8ae39ce395c7b9b8910dfde6f22a7745aea83/hphp/system/php/stream/default-filters.php#L77) buffers the whole string. This means that compressing a stream of 100 MB actually stores the whole string in memory before invoking the underlying compression algorithm.
* PHP 5.3 only: Tends to SEGFAULT occasionally on shutdown?

Our test suite contains several test cases that exhibit these issues.
If you feel some test case is missing or outdated, we're happy to accept PRs! :)

## Install

The recommended way to install this library is [through composer](https://getcomposer.org).
[New to composer?](https://getcomposer.org/doc/00-intro.md)

```bash
$ composer require clue/zlib-react:~0.1.0
```

## License

MIT

## More

* If you want to learn more about processing streams of data, refer to the documentation of
  the underlying [react/stream](https://github.com/reactphp/stream) component
* If you want to process compressed tarballs (`.tar.gz` and `.tgz` file extension), you may
  want to use [clue/tar-react](https://github.com/clue/php-tar-react) on the decompressed stream.
