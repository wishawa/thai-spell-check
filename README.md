# Thai-Spell-Check
JavaScript-WebAssembly to check spelling of Thai text

# Usage
## The Function
This library provides a function ```checkThaiSpelling```.

**input**: a javascript string

**return**: a 2D integer array of n rows and 2 columns, where n is the number of incorrectly spelled words.

Column 1: the index of the first character of a misspelled word.

Column 2: the index of the first character *after* the misspelled word.

**Example**:

```javascript
> checkThaiSpelling("ไข่ใก่ฟองนี้มีขะหนาดไหญ่")
[[ 3,  6], [14, 24]]
//ใก่ and ขะหนาดไหญ่ are marked as incorrect
//note that ขะหนาด and ไหญ่ are grouped together
//3 -> ใ in ใก่, 6 -> ฟ in ฟอง after ใก่
//14 -> ข in ขะหนาด, 24 -> emptiness after the string (the string is 24 letters long)

```


## Files
You only need
1. checker.js
2. thbrk.js
3. thbrk.wasm
4. thbrk.data



## Using With Dynamic Import
In your JavaScript:

```javascript
import('./checker.js').then(function(module) {
    //loadThaiSpellchecker() returns a promise that resolves to the checker function.
    module.loadThaiSpellchecker().then(function(checkThaiSpelling) {
        //At this point the function checkThaiSpelling is ready.
        console.log(checkThaiSpelling("ไข่ใก่ฟองนี้มีขะหนาดไหญ่"));
        //You probably want to save the function to a global variable so it can be called from outside this scope.
    });
});
```
Usage example:

```javascript
var checkerFunction;//any name you want
var thaiSpellcheckerReady = false;
var someButton;
var someTextArea;
import('./checker.js').then(function(module) {
    module.loadThaiSpellchecker().then(function(r) {
        checkerFunction = r;//save the function to global var
        thaiSpellcheckerReady = true;
    });
});
window.onload = function() {
    someButton = document.getElementById("myButton");
    someTextArea= document.getElementById("myTextArea");
    someButton.onclick = function() {
    if(!thaiSpellcheckerReady) {
        alert("not ready");
        return;
    }
    
    var text = someTextArea.value;
    var checkResult = checkerFunction(text);//check the text
    if(checkResult.length > 0) {//result length is 0 if no misspelling is found
        alert("you made mistakes");
        
        //alert every word marked as incorrect
        for(var i=0; i<checkResult.length; i++) {
            alert("incorrectly spelled word: " + text.slice(checkResult[i][0], checkResult[i][1]));
        }
        return;
    }
    else {
        alert("no mistake!");
    }
}
```


## Using Without Dynamic Import
In your JavaScript:

```javascript
import {loadThaiSpellchecker} from './checker.js';
//loadThaiSpellchecker() returns a promise that resolves to the checker function.
loadThaiSpellchecker().then(function(checkThaiSpelling) {
    //At this point the function checkThaiSpelling is ready.
    console.log(checkThaiSpelling("ไข่ใก่ฟองนี้มีขะหนาดไหญ่"));
    //You probably want to save the function to a global variable so it can be called from outside this scope.
});
```

## Using Without Import
Remember that if the browser does not support WebAssembly, this won't work anyway.
If you want to use this in browsers that support WebAssembly, but not ```import```, use

```html
<script src="thbrk.no_import.js"></script>
<script src="checker.no_import.js"></script>
<script>
    loadThaiSpellchecker().then(function(checkThaiSpelling) {
        //At this point the function checkThaiSpelling is ready.
        console.log(checkThaiSpelling("ไข่ใก่ฟองนี้มีขะหนาดไหญ่"));
    });
</script>
```
But be careful of name collision.

# Notes
Typically, checking should take no more than 10 milliseconds. Still, you should call the function asynchronously if possible.

Checking is not always correct. Currently, the checker does not take in account word usage frequency. A word can be misspelled to a less popular, but correct, word (ex. มาก -> มมาก).

libthai, the word breaking backend used, needs 'recovery space' of 3 correct words after an incorrect word.

```
<incorrect><correct><correct>
//all marked as incorrect

<incorrect><correct><correct><correct>
//only first word marked as incorrect

<incorrect><correct><correct><incorrect><correct>
//all marked as incorrect

<incorrect><correct><correct><correct><incorrect><correct>
//first and last two words marked as incorrect
```


Word suggestion coming in the future.


# License
GPL v3

# Included works
[libthai](https://github.com/tlwg/libthai)

[libdatrie](https://github.com/tlwg/libdatrie)
