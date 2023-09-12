[![npm version](https://badge.fury.io/js/%40mapbox%2Fspritezero.svg)](https://badge.fury.io/js/%40mapbox%2Fspritezero)
[![Build Status](https://travis-ci.com/mapbox/spritezero.svg?branch=main)](https://travis-ci.com/mapbox/spritezero)

Forked from @mapbox/spritezero

### WTF? (Why The Fork?)

The mapbox-gl-js [`Map.addImage` method](https://docs.mapbox.com/mapbox-gl-js/api/#map#addimage) supports loading icons with SDF (signed distance fields). The [`icon-color`](https://docs.mapbox.com/mapbox-gl-js/style-spec/#paint-symbol-icon-color) and [`icon-halo-color`](https://docs.mapbox.com/mapbox-gl-js/style-spec/#paint-symbol-icon-halo-color) properties of the Mapbox Style Specification are only supported on SDF icons.

This fork of the spritezero library adds an `options.sdf` parameter to the spritezero functions. SDF icons are generated from SVGs using the `pathToSDF` function in the [elastic/fontnik](https://github.com/elastic/fontnik) library.

You can see a demo of SDF sprites [here](http://nickpeihl.github.io/maki-sdf-sprites/).

### Usage

```js
var spritezero = require("@elastic/spritezero");
var fs = require("fs");
var glob = require("glob");
var path = require("path");

[1, 2, 4].forEach(function (pxRatio) {
  var svgs = glob
    .sync(path.resolve(path.join(__dirname, "input/*.svg")))
    .map(function (f) {
      return {
        svg: fs.readFileSync(f),
        id: path.basename(f).replace(".svg", ""),
      };
    });
  var pngPath = path.resolve(
    path.join(__dirname, "output/sprite@" + pxRatio + ".png")
  );
  var jsonPath = path.resolve(
    path.join(__dirname, "output/sprite@" + pxRatio + ".json")
  );

  // Pass `true` in the layout parameter to generate a data layout
  // suitable for exporting to a JSON sprite manifest file.
  spritezero.generateLayout(
    { imgs: svgs, pixelRatio: pxRatio, sdf: true, format: true },
    function (err, dataLayout) {
      if (err) return;
      fs.writeFileSync(jsonPath, JSON.stringify(dataLayout));
    }
  );

  // Pass `false` in the layout parameter to generate an image layout
  // suitable for exporting to a PNG sprite image file.
  spritezero.generateLayout(
    { imgs: svgs, pixelRatio: pxRatio, sdf: true, format: false },
    function (err, imageLayout) {
      spritezero.generateImage(imageLayout, function (err, image) {
        if (err) return;
        fs.writeFileSync(pngPath, image);
      });
    }
  );
});
```

### Installation

Requires [nodejs](http://nodejs.org/) v10.0.0 or greater.

```bash
$ npm install @elastic/spritezero
```

### Executable

[@elastic/spritezero-cli](https://github.com/elastic/spritezero-cli) is an executable for bundling and creating your own sprites from a folder of svg's:

```bash
$ npm install -g @elastic/spritezero-cli
$ spritezero --help

Usage:
spritezero [output filename] [input directory]
  --retina      shorthand for --ratio=2
  --ratio=[n]   pixel ratio
  --sdf         generate sdf sprites
```
