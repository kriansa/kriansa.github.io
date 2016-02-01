---
layout: post
title: H2P - PHP library to convert HTML files to PDF
date: 2013-03-24 20:44:34 BRT
tags: open-source, php, pdf
categories: projects
permalink: "/:categories/:title"
redirect_from: /blog/2013/03/h2p-convert-html-files-to-pdf/
---
__H2P__ is a simple library to convert HTML files with images, JS and CSS into PDF files, using the awesome [PhantomJS](http://phantomjs.org/){:target="_blank"}.

<a href="https://github.com/kriansa/h2p"><img style="border-radius: none; box-shadow: none; position: fixed; top: 0; left: 0; border: 0;" alt="Fork me on GitHub" src="https://s3.amazonaws.com/github/ribbons/forkme_left_darkblue_121621.png" /></a>

## Usage
If you're using composer (I strongly recommend that), just add "_kriansa/h2p_" into your require attribute, like so:

{% highlight json %}
{
    "name": "acme/blog",
    "require": {
        "kriansa/h2p": "dev-master"
    }
}
{% endhighlight %}

Then, you must download the PhantomJS binary on its website. Extract and move the _bin/phantomjs_ (or _bin/phantomjs.exe_, if you're using Windows) to H2P/bin/. These bins are not shipped with the main package because they're too big (about 40mb).

That's all, it should work fine. Take a look at the sample code below:

{% highlight php %}
<?php

use H2P\Converter;
use H2P\Adapter\PhantomJS;
use H2P\TempFile;

// Set the input content to convert
$input = new TempFile($htmlString);
$input = 'http://www.google.com/';
$input = '/path/to/file.html';

// Then do the conversion
$output = new TempFile();
$instance = new Converter(new PhantomJS(), $input, $output);
$instance->convert();

// Save it somewhere
$output->save('/another/path/to/file.pdf');
// or output it
header('Pragma: public');
header('Expires: 0');
header('Cache-Control: must-revalidate, post-check=0, pre-check=0');
header('Content-Type: application/pdf');
header('Content-Transfer-Encoding: binary');
header('Content-Length: ' . filesize($output->getFileName()));
echo $output->getContent();
{% endhighlight %}

Note that the **$input** is defined multiple times. That's just to show you how many URIs are accepted by the <em>Converter</em>.

## Advanced usage
The Converter constructor accepts 6 params, see below:

> Converter::*__construct*(AdapterAbstract **$adapter**, string\|TempFile **$uri**, string\|TempFile **$destination**, [string **$format**, [string **$orientation**, [string **$border**]]])

* _AdapterAbstract_ **$adapter**

> The adapter to convert the file. You can write your own adapter if you don't like PhantomJS

* _string\|TempFile_ **$uri**

>The input URI to be converted. The package includes a TempFile class which can be used here

* _string\|TempFile_ **$destination**

> The output file. You can use the TempFile here too :)

* _string_ **$format**

> Converter::FORMAT_A4 - see below a list of page formats that you can use here.

* _string_ **$orientation**

> Converter::ORIENTATION_PORTRAIT or Converter::ORIENTATION_LANDSCAPE

* _string_ **$border**

> A string, which can be '1cm', '2in', '20px' or so.

In the file _Converter.php_, you can see a list of formats (page-sizes) available to convert to. When not specified, the default used is A4.

Format list (from <strong>Converter.php</strong>)

{% highlight php startinline=true %}
    const FORMAT_A0 = 'A0';
    const FORMAT_A1 = 'A1';
    const FORMAT_A2 = 'A2';
    const FORMAT_A3 = 'A3';
    const FORMAT_A4 = 'A4';
    const FORMAT_A5 = 'A5';
    const FORMAT_A6 = 'A6';
    const FORMAT_A7 = 'A7';
    const FORMAT_A8 = 'A8';
    const FORMAT_A9 = 'A9';
    const FORMAT_B0 = 'B0';
    const FORMAT_B1 = 'B1';
    const FORMAT_B2 = 'B2';
    const FORMAT_B3 = 'B3';
    const FORMAT_B4 = 'B4';
    const FORMAT_B5 = 'B5';
    const FORMAT_B6 = 'B6';
    const FORMAT_B7 = 'B7';
    const FORMAT_B8 = 'B8';
    const FORMAT_B9 = 'B9';
    const FORMAT_B10 = 'B10';
    const FORMAT_C5E = 'C5E';
    const FORMAT_COMM10E = 'Comm10E';
    const FORMAT_DLE = 'DLE';
    const FORMAT_EXECUTIVE = 'Executive';
    const FORMAT_FOLIO = 'Folio';
    const FORMAT_LEDGER = 'Ledger';
    const FORMAT_LEGAL = 'Legal';
    const FORMAT_LETTER = 'Letter';
    const FORMAT_TABLOID = 'Tabloid';
{% endhighlight %}

## License
The MIT License (MIT)
Copyright (c) 2013 Daniel Garajau Pereira

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
