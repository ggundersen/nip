# nip - Node Input/output Piper
`nip` is a command line utility for performing any type of processing to and from files and pipes

# Install
With [node.js](http://nodejs.org/) and [npm](http://github.com/isaacs/npm):

    npm install npipe -g
    
If you omit the `-g` then make sure to add the local npm module path to your dafault path

You should now be able to call `nip` from the command line.

### Usage: `nip js-function [options] [files]`

The js-function can be one of three syntaxes:

1. `function(line, index, lines, cols) { /* code here */ return value; }`
2. `return line.substr(0, 10) + index`
3. `/* code */ return function(line, i, lines) { /* ... */ return value; }`

The names `line`, `index`, `lines`, and `cols` can be changed in the first and third style syntaxes

If the return value is `false` or `undefined` nothing is sent to output stream  
If the return value is `true` then the line will be sent to the output stream  
else the return value will be sent to the output stream (including an empty string)

### options

`-f js-file` or  `--file=js-file`
>use the js-file as the function to execute on the input instead of the `js-function` argument
you must supply either this option or the `js-function` argument

----

`-1` or `--first-line-only`
>only execute once per file, not for each line  
this is useful if you plan on proccessing the file as a whole, namely through the `lines` variable  
for examaple (not a useful one): `nip 'return lines.length' -1 file.txt`

----

`-c` or `--cols`
>tell `nip` to pass in an array of values split from each line as the fourth argument

----

`-s string-or-regex`, `--col-splitter=string-or-regex`
>the splitter for --cols, can be regex or string format, by default it's `/\s+/`

----

`-n string-or-regex`, `--line-splitter=string-or-regex`
>the line separator, can be regex or string format, by default we're splitting on lines so it's `\n`

---

## Examples

Only output lines that begin with the word `var`:
```    
nip 'function(l) { return /^var/.test(l); }' lines-that-start-with-var.txt
```

Output every second line only in uppercase in a file:

```
nip 'function(line, i) { return i % 2 ? line.toUpperCase() : false; }' every-2nd-line.txt
```

Trim whitesplace from a file:

```
nip 'return line.replace(/^s*|s*$/g, "");' trim-lines.txt
```

Run the contents of `jsfile.js` on `file.txt`:

```
nip -f jsfile.js file.txt
```

Print out a file in an easy to read format

```
nip 'return index % 2 ? line.red.whiteHiBg : line.yellow.blueBg.bold' file.txt
```
[read more about `ccolors`](https://github.com/kolodny/ccolors)

Like most unix commands, you can pipe the input and/or output:

generate a script file to rename files recursively and sequentiality
`find . -type f |  nip 'return "mv " + line + " " + line.replace(/\/[^/]*$/, "") + "/" + index;' > rename-script`

rename files recursively and sequentiality
`find . -type f |  nip 'return "mv " + line + " " + line.replace(/\/[^/]*$/, "") + "/" + index;' | sh`

find the biggest number from all files in a directory:

    nip '
      var biggest = 0;
      this.on("end", function() { print(biggest); });
      return function(_,i,lines) {
        biggest = Math.max(biggest,
          Math.max.apply(Math, lines.match(/(?:\s|^)[\d]+(?:\.\d*)?(?:\s|$)/g))
        )
      }' -1 *

---

By default there are `start`, `end`, `fileStart`, and `fileEnd` events you can register, you can also register in  
`this.onend = function() {/* code */}` format

The context inside the main function can be used as a global store, and has a `filename` property

---

### Why
This is for people who aren't "devops" and who can't crank out a fancy piped shell script using `awk`, `sed,` and `grep`. Also most programmers who have node installed can write a quick javascript one+ liner to do what the oldschoolers would make a shell script out of.

https://github.com/kolodny/nip