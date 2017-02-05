---
layout: post
title: Pixel fonts for Chinese characters
category: posts
---

This lovely pixelized Jekyll site remind me of my early-days programming experience with Q Basic on a MS-DOS machine. But one thing missing here is a proper Chinese fonts to mimic the fully old-school  computer style.

After a bit of search, I found a 8-bit Chinese fonts, [Founder's Feng](http://ifont.foundertype.com/showsortpic.php?sid=1106) Dot Fonts, in a good match with my flavor.

To display non-ASCII characters 像素字体, font resources are necessary to support browser to render the right Chinese characters.

The issue is Chinese, Japanese, Korean and some other Asian languages have a huge characters set. Font resources file to support Chinese characters is much bigger than the ACSII one.

![](../images/Screenshot 2017-02-05 10.54.01.png)

So we need a subset our font of all the Chinese characters that is actually used in the posts. There are many open-source tools to do this kind of job: `fonttools`,`fontmin`,`font-spider`

Basic idea of those tools are similar, the major difference is their implementation languages. Here I use `fontmin` because I can have a subset fonts family (`ttf`,`svg`,`eot`,`woff`) that support major browsers and also a handy `css` configuration file.

```js
var Fontmin = require('fontmin');

var srcPath = './css/fonts/founderfeng.ttf'; // Source of ttf fonts
var destPath = './css/fonts/additional/';    // Output path

var glob = require("glob");
var texts = '';

var fs = require('fs');

function extractchinese(files){
fs.readFile(files, 'utf-8', function read(err, data) {
    if (err) {
        throw err;
    }
    var chinesecontents;
    chinesecontents = data.replace(/[^\u4e00-\u9fa5]/gi,'');
    // console.log(chinesecontents);
    texts = texts + chinesecontents;
});
};

glob('./_posts/*.markdown', 'utf-8', function (er, files) {
  filename = files;
  filename.forEach(extractchinese);
});

console.log("Text list to be process is " + texts);

var fontmin = new Fontmin()
    .src(srcPath)
    .use(Fontmin.glyph({        // subset plugin
        text: texts           // subset fonts
    }))
    .use(Fontmin.ttf2eot())     // eot plugin
    .use(Fontmin.ttf2woff())    // woff plugin
    .use(Fontmin.ttf2svg())     // svg plugin
    .use(Fontmin.css())         // css plugin
    .dest(destPath);            // Output configuration

fontmin.run(function (err, files, stream) {

    if (err) {                  // Catch error if there is any
        console.error(err);
    }
    console.log('done');        // Notification
});

```

This piece of code search all markdown files under `_posts` folder and extract Chinese characters to subset the main `ttf` font into into four subset fonts in`ttf`,`svg`,`eot` and `woff` format.

```css
@font-face {
    font-family: "FZBITMAPFENGJW";
    src: url("founderfeng.eot"); /* IE9 */
    src: url("founderfeng.eot?#iefix") format("embedded-opentype"), /* IE6-IE8 */
    url("founderfeng.woff") format("woff"), /* chrome, firefox */
    url("founderfeng.ttf") format("truetype"), /* chrome, firefox, opera, Safari, Android, iOS 4.2+ */
    url("founderfeng.svg#FZBITMAPFENGJW") format("svg"); /* iOS 4.1- */
    font-style: normal;
    font-weight: normal;
}
```

Since `css` support weighted `font-face` for a single font-family, it is easy to define an extra web font for non-ASCII characters. Here we can simply use the same font-family name `DOS` for our subset Chinese characters. To avoid the overwrite of ASCII font style, we define the range `unicode-range: U+00-024F;` in English fonts.

```css
@font-face {
    font-family: "DOS";
    src: url("fonts/additional/founderfeng.eot"); /* IE9 */
    src: url("fonts/additional/founderfeng.eot?#iefix") format("embedded-opentype"), /* IE6-IE8 */
    url("fonts/additional/founderfeng.woff") format("woff"), /* chrome, firefox */
    url("fonts/additional/founderfeng.ttf") format("truetype"), /* chrome, firefox, opera, Safari, Android, iOS 4.2+ */
    url("fonts/additional/founderfeng.svg#FZBITMAPFENGJW") format("svg"); /* iOS 4.1- */
    font-style: normal;
    font-weight: normal;
}

@font-face {
  font-family: 'DOS';
  unicode-range: U+00-024F;
  src: url('fonts/Fixedsys500c.eot');
  src: local('☺'), url('fonts/Fixedsys500c.woff') format('woff'), url('fonts/Fixedsys500c.ttf') format('truetype'), url('fonts/Fixedsys500c.svg') format('svg');
  font-weight: normal;
  font-style: normal; }
```  

The result page is what you can see here. And the load speed is much quicker than before. 

![](../images/Screenshot 2017-02-05 11.49.44.png)



License disclaimer: Founder's Dot Feng is not free font. The personal non-commercial use license is priced at 2 CNY, but since it was not possible to purchase a license on this legacy online shop, I paid the license fee by ordered  [another fonts](https://item.taobao.com/item.htm?spm=a1z09.2.0.0.sCErJX&id=17990674587&_u=5lefkln2b6c) at the same price.
