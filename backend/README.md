#backend
The codes here are reduced and modified from [LibThai](https://github.com/tlwg/libthai)

The ThBrk module, which provides word breaking functionality (finding where each word ends), has been modified to also give the position of words that are not present in the dictionary.

Compile to WebAssembly to use.

#Building
Requires [Emscripten](https://emscripten.org/).

To build, simply run
```
make module
```

if you want to link in HTML instead of using import, run
```
make js
```
