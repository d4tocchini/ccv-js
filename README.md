# ccv.js

A port of [ccv](http://libccv.org/) (a minimalistic computer vision library written in C) to the browser using [emscripten](http://kripken.github.io/emscripten-site/).

## Demos

[Demos](https://fta2012.github.io/ccv-js/).

## Usage

Given some example ccv C code you want to translate:

```C
ccv_dense_matrix_t* image = 0;
ccv_read("someFile.png", &image, CCV_IO_GRAY | CCV_IO_ANY_FILE);
ccv_array_t* rects = ccv_swt_detect_words(image, ccv_swt_default_params);
// Do stuff with rects
ccv_array_free(rects);
ccv_matrix_free(image);
```

The equivalent javascript is:

```javascript
CCV = CCVLib({}); // initialize

var image = new CCV.ccv_dense_matrix_t();
CCV.ccv_read(document.querySelector('#some-img-element'), image, CCV.CCV_IO_GRAY);
var rects = CCV.ccv_swt_detect_words(image, CCV.ccv_swt_default_params);
// console.log(rects.toJS());
rects.delete();
image.delete();
```

Tips for converting:

- `ccv_read` now takes in a [CanvasImageSource](https://developer.mozilla.org/en-US/docs/Web/API/CanvasImageSource) (which is either \<img\>, \<video\> or \<canvas\>) or [ImageData](https://developer.mozilla.org/en-US/docs/Web/API/ImageData). Only valid flags are `CCV.CCV_IO_GRAY` or `CCV.CCV_IO_RGB_COLOR`.
- `ccv_write` now outputs to either a \<img\>, \<canvas\>, \<div\>, or [ImageData](https://developer.mozilla.org/en-US/docs/Web/API/ImageData). If outputing to a div it will append a new canvas with the contents. Can only be used with matrices with datatype `CCV_8U` (use the `getData()` method on `ccv_dense_matrix_t` otherwise).
- Only [ImageData](https://developer.mozilla.org/en-US/docs/Web/API/ImageData/ImageData) version will work for reading/writing if using webworkers.
- Raw pointers are wrapped in a `shared_ptr` with corresponding free function as the deleter.
- No destructors so you still need to call delete on those `shared_ptr`s (including the ones returned to you).
- Functions that took in a `T*` now take in a `const shared_ptr<T>&`.
- Functions that took in a `T**` now take in a `shared_ptr<T>&` which will hold the output value.
- Functions that took in a `T*[]` now take in a js array of `shared_ptr<T>`.
- Functions that took in a struct param by value now take in a JS object with the same shape (grep for `value_object`). `CCV.ccv_swt_default_params` for example is just a JS object.
- `ccv_array_t` needs to know element type (e.g., `ccv_rect_array`, `ccv_comp_array`). There is a `fromJS()` and `toJS()` method on them to convert from/to js arrays.

See `examples/index.js` for source code of the demos (which are mostly ported from the C demos in `external/ccv/bin/`). The bindings were added by hand so if it's not in the demos it probably doesn't have bindings (but they are easy to add yourself).

## Install/Build

You should be able to just drop in the js file found at `build/ccv.js`(~9MB, 2MB gzipped):

```html
<script src="//cdn.rawgit.com/fta2012/ccv-js/f63c56c8/build/ccv.js"></script>
```

There is a smaller file at `build/ccv_without_filesystem.js`(~2.3MB, 400KB gzipped) but at the cost of removing emscripten filesystem support and the model files required for SCD, ICF, and DPM.

If you want rebuild to include your own trained files or add new bindings:

1. Install [emscripten](http://kripken.github.io/emscripten-site/docs/getting_started/index.html)
2. Run make: `emmake make`

The emscripten tutorial you should read is [this](https://kripken.github.io/emscripten-site/docs/porting/connecting_cpp_and_javascript/embind.html).
The C++ bindings are all in `ccv_bindings.cpp`. JS helpers used in the bindings is in `ccv_pre.js`.

## TODOs / Limitations

Grep for TODO in the code. If it wasn't demoed, it probably doesn't work (notably, convnet doesn't have bindings since it requires a huge sqlite file). Feel free to contribute!

## License

MIT for the bindings and BSD 3-clause for ccv (which can be found in ./external/ccv/COPYING)
