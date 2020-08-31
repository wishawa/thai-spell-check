This is for developers. Want to just spellcheck your work? [Go here](https://wishawa.github.io/thai-spellcheck-extension/).
# Thai-Spell-Check
JavaScript-WebAssembly to check spelling of Thai text

```javascript
> checkThaiSpelling("ไข่ใก่ฟองนี้มีขะหนาดไหญ่")
[[ 3,  6], [14, 24]]
//output explanation available below
```

# Installation / File Placement
## Normal Installation
The files you need are:

1. ```checker.js```

2. ```thbrk.js```

3. ```thbrk.wasm```

4. ```thbrk.data```

**Place all four files in the same directory**

## Custom Files Location
Custom files location requires some manual tweak.

1. Open ```thbrk.js``` in your text editor. You need a text editor that can handle very long text in a single line.

2. To specify ```thbrk.data``` location: search for ```REMOTE_PACKAGE_BASE="thbrk.data"``` replace it with ```REMOTE_PACKAGE_BASE="YOUR/PATH/HERE/thbrk.data"``` where ```YOUR/PATH/HERE/``` is *relative to the page HTML* (can also be absolute ```/YOUR/ABSOLUTE/PATH```).

3. To specify ```thbrk.wasm``` location: search for ```wasmBinaryFile="thbrk.wasm"``` replace it with ```wasmBinaryFile="YOUR/PATH/HERE/thbrk.wasm"``` where ```YOUR/PATH/HERE/``` is *relative to the page HTML* (can also be absolute ```/YOUR/ABSOLUTE/PATH```).

4. Save ```thbrk.js```.

5. To specify ```thbrk.js``` location, edit the first line of ```checker.js``` from ```import thaiSpellcheckerBackend from './thbrk.js';``` to ```import thaiSpellcheckerBackend from './YOUR/PATH/HERE/thbrk.js';```. Path is *relative to ```checker.js```*. Also, you need the ```./``` if your path is relative. Again, absolute path works too.

6. To specify ```checker.js``` location, just specify the path when importing (See Usage section for importing). Ex: ```import('./YOUR/PATH/HERE/checker.js')```. Path is *relative to your JS file*. Also, you need the ```./``` if your path is relative. Again, absolute path works too.


# Usage
## The Function
This library provides a function ```checkThaiSpelling```.

**input**: a javascript string

**return**: a 2D integer array of n rows and 2 columns, where n is the number of incorrectly spelled words.

Column 1: the index of the first character of a misspelt word.

Column 2: the index of the first character *after* the misspelt word.

**Example**:

```javascript
> checkThaiSpelling("ไข่ใก่ฟองนี้มีขะหนาดไหญ่")
[[ 3,  6], [14, 24]]
//ใก่ and ขะหนาดไหญ่ are marked as incorrect
//note that ขะหนาด and ไหญ่ are grouped together. This is because the backend has no knowledge of the misspelt word and cannot guess where it ends.
//3 -> ใ in ใก่, 6 -> ฟ in ฟอง after ใก่
//14 -> ข in ขะหนาด, 24 -> emptiness after the string (the string is 24 letters long)

```

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
Usage example - check spelling of text in a text area and alert misspelt words:

```javascript
var checkerFunction;//any name you want
var thaiSpellcheckerReady = false;//the checker requires some loading, so it is not ready in the beginning

var someButton;
var someTextArea;

import('./checker.js').then(function(module) {//dynamic import the checker module
    module.loadThaiSpellchecker().then(function(r) {//load the checker
        //r is the checker function
        checkerFunction = r;//save the function to a global var so we can call it from elsewhere
        thaiSpellcheckerReady = true;//the checker is ready now
    });
});

window.onload = function() {
    someButton = document.getElementById("myButton");
    someTextArea = document.getElementById("myTextArea");
    
    //on click, check text in the text area and alert errors.
    someButton.onclick = function() {
        if(!thaiSpellcheckerReady) {
            alert("not ready");
            return;
        }
        
        var text = someTextArea.value;
        
        var checkResult = checkerFunction(text);//check the text
        
        if(checkResult.length > 0) {//result length is 0 if no misspelling is found
            alert("mistake(s) found");
            
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
}
```


## Using With Static Import
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
If you want to use this in browsers without ```import``` support, use

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
But keep in mind that most browsers that support WebAssembly support ```import``` too anyway.

Also be careful of name collision.

# Notes
### Performance
Typically, checking should take no more than 10 milliseconds. Still, you should assume the worst (a few hundred milliseconds, probably).

### Correctness
Checking is not always correct. Currently, the checker does not take in account word usage frequency. A word can be misspelt to a less common (but still correct) word (ex. มาก -> มมาก).

### Marking Behavior
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


Word replacements suggestion coming in the future.


# License
GPL v3

# Included works
[libthai](https://github.com/tlwg/libthai)

[libdatrie](https://github.com/tlwg/libdatrie)
