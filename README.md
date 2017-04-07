# api documentation for  [lazy (v1.0.11)](https://github.com/pkrumins/node-lazy#readme)  [![npm package](https://img.shields.io/npm/v/npmdoc-lazy.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-lazy) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-lazy.svg)](https://travis-ci.org/npmdoc/node-npmdoc-lazy)
#### Lazy lists for node

[![NPM](https://nodei.co/npm/lazy.png?downloads=true)](https://www.npmjs.com/package/lazy)

[![apidoc](https://npmdoc.github.io/node-npmdoc-lazy/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-lazy_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-lazy/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-lazy/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-lazy/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Peteris Krumins",
        "email": "peteris.krumins@gmail.com",
        "url": "http://www.catonmat.net"
    },
    "bugs": {
        "url": "https://github.com/pkrumins/node-lazy/issues"
    },
    "dependencies": {},
    "description": "Lazy lists for node",
    "devDependencies": {
        "expresso": ">=0.7.5"
    },
    "directories": {},
    "dist": {
        "shasum": "daa068206282542c088288e975c297c1ae77b690",
        "tarball": "https://registry.npmjs.org/lazy/-/lazy-1.0.11.tgz"
    },
    "engines": {
        "node": ">=0.2.0"
    },
    "homepage": "https://github.com/pkrumins/node-lazy#readme",
    "keywords": [
        "lazy lists",
        "functional"
    ],
    "license": {
        "type": "MIT"
    },
    "main": "./lazy.js",
    "maintainers": [
        {
            "name": "pkrumins",
            "email": "peteris.krumins@gmail.com"
        }
    ],
    "name": "lazy",
    "optionalDependencies": {},
    "readme": "Lazy lists for node\n===================\n\n\n# Table of contents:\n\n[Introduction](#Introduction)\n  \n[Documentation](#Documentation)\n\n<a name=\"Introduction\" />\n# Introduction\nLazy comes really handy when you need to treat a stream of events like a list.\nThe best use case currently is returning a lazy list from an asynchronous\nfunction, and having data pumped into it via events. In asynchronous\nprogramming you can't just return a regular list because you don't yet have\ndata for it. The usual solution so far has been to provide a callback that gets\ncalled when the data is available. But doing it this way you lose the power of\nchaining functions and creating pipes, which leads to not that nice interfaces.\n(See the 2nd example below to see how it improved the interface in one of my\nmodules.)\n\nCheck out this toy example, first you create a Lazy object:\n'''javascript\n    var Lazy = require('lazy');\n\n    var lazy = new Lazy;\n    lazy\n      .filter(function (item) {\n        return item % 2 == 0\n      })\n      .take(5)\n      .map(function (item) {\n        return item*2;\n      })\n      .join(function (xs) {\n        console.log(xs);\n      });\n'''\n\nThis code says that 'lazy' is going to be a lazy list that filters even\nnumbers, takes first five of them, then multiplies all of them by 2, and then\ncalls the join function (think of join as in threads) on the final list.\n\nAnd now you can emit 'data' events with data in them at some point later,\n'''javascript\n    [0,1,2,3,4,5,6,7,8,9,10].forEach(function (x) {\n      lazy.emit('data', x);\n    });\n'''\n\nThe output will be produced by the 'join' function, which will output the\nexpected [0, 4, 8, 12, 16].\n\nAnd here is a real-world example. Some time ago I wrote a hash database for\nnode.js called node-supermarket (think of key-value store except greater). Now\nit had a similar interface as a list, you could .forEach on the stored\nelements, .filter them, etc. But being asynchronous in nature it lead to the\nfollowing code, littered with callbacks and temporary lists:\n'''javascript\n    var Store = require('supermarket');\n\n    var db = new Store({ filename : 'users.db', json : true });\n\n    var users_over_20 = [];\n    db.filter(\n      function (user, meta) {\n        // predicate function\n        return meta.age > 20;\n      },\n      function (err, user, meta) {\n        // function that gets executed when predicate is true\n        if (users_over_20.length < 5)\n          users_over_20.push(meta);\n      },\n      function () {\n        // done function, called when all records have been filtered\n\n        // now do something with users_over_20\n      }\n    )\n'''\nThis code selects first five users who are over 20 years old and stores them\nin users_over_20.\n\nBut now we changed the node-supermarket interface to return lazy lists, and\nthe code became:\n'''javascript\n    var Store = require('supermarket');\n\n    var db = new Store({ filename : 'users.db', json : true });\n\n    db.filter(function (user, meta) {\n        return meta.age > 20;\n      })\n      .take(5)\n      .join(function (xs) {\n        // xs contains the first 5 users who are over 20!\n      });\n'''\nThis is so much nicer!\n\nHere is the latest feature: .lines. Given a stream of data that has \\n's in it,\n.lines converts that into a list of lines.\n\nHere is an example from node-iptables that I wrote the other week,\n'''javascript\n    var Lazy = require('lazy');\n    var spawn = require('child_process').spawn;\n    var iptables = spawn('iptables', ['-L', '-n', '-v']);\n\n    Lazy(iptables.stdout)\n        .lines\n        .map(String)\n        .skip(2) // skips the two lines that are iptables header\n        .map(function (line) {\n            // packets, bytes, target, pro, opt, in, out, src, dst, opts\n            var fields = line.trim().split(/\\s+/, 9);\n            return {\n                parsed : {\n                    packets : fields[0],\n                    bytes : fields[1],\n                    target : fields[2],\n                    protocol : fields[3],\n                    opt : fields[4],\n                    in : fields[5],\n                    out : fields[6],\n                    src : fields[7],\n                    dst : fields[8]\n                },\n                raw : line.trim()\n            };\n        });\n'''\nThis example takes the 'iptables -L -n -v' command and uses .lines on its output.\nThen it .skip's two lines from input and maps a function on all other lines that\ncreates a data structure from the output.\n\n<a name=\"Documentation\" />\n# Documentation\n\nSupports the following operations:\n\n* lazy.filter(f)\n* lazy.forEach(f)\n* lazy.map(f)\n* lazy.take(n)\n* lazy.takeWhile(f)\n* lazy.bucket(init, f)\n* lazy.lines\n* lazy.sum(f)\n* lazy.product(f)\n* lazy.foldr(op, i, f)\n* lazy.skip(n)\n* lazy.head(f)\n* lazy.tail(f)\n* lazy.join(f)\n\nThe Lazy object itself has a .range property for generating all the possible ranges.\n\nHere are several examples:\n\n* Lazy.range('10..') - infinite range starting from 10\n* Lazy.range('(10..') - infinite range starting from 11\n* Lazy.range(10) - range from 0 to 9\n* Lazy.range(-10, 10) - range from -10 to 9 (-10, -9, ... 0, 1, ... 9)\n* Lazy.range(-10, 10, 2) - range from -10 to 8, skipping every 2nd element (-10, -8, ... 0, 2, 4, 6, 8)\n* Lazy.range(10, 0, 2) - reverse range from 10 to 1, skipping every 2nd element (10, 8, 6, 4, 2)\n* Lazy.range(10, 0) - reverse range from 10 to 1\n* Lazy.range('5..50') - range from 5 to 49\n* Lazy.range('50..44') - range from 50 to 45\n* Lazy.range('1,1.1..4') - range from 1 to 4 with increment of 0.1 (1, 1.1, 1.2, ... 3.9)\n* Lazy.range('4,3.9..1') - reverse range from 4 to 1 with decerement of 0.1\n* Lazy.range('[1..10]') - range from 1 to 10 (all inclusive)\n* Lazy.range('[10..1]') - range from 10 to 1 (all inclusive)\n* Lazy.range('[1..10)') - range grom 1 to 9\n* Lazy.range('[10..1)') - range from 10 to 2\n* Lazy.range('(1..10]') - range from 2 to 10\n* Lazy.range('(10..1]') - range from 9 to 1\n* Lazy.range('(1..10)') - range from 2 to 9\n* Lazy.range('[5,10..50]') - range from 5 to 50 with a step of 5 (all inclusive)\n\nThen you can use other lazy functions on these ranges.\n\n\n",
    "readmeFilename": "README.md",
    "repository": {
        "type": "git",
        "url": "git+ssh://git@github.com/pkrumins/node-lazy.git"
    },
    "scripts": {
        "test": "expresso"
    },
    "version": "1.0.11"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module lazy](#apidoc.module.lazy)
1.  [function <span class="apidocSignatureSpan">lazy.</span>range ()](#apidoc.element.lazy.range)
1.  [function <span class="apidocSignatureSpan">lazy.</span>super_ ()](#apidoc.element.lazy.super_)



# <a name="apidoc.module.lazy"></a>[module lazy](#apidoc.module.lazy)

#### <a name="apidoc.element.lazy.range"></a>[function <span class="apidocSignatureSpan">lazy.</span>range ()](#apidoc.element.lazy.range)
- description and source-code
```javascript
range = function () {
    var args = arguments;
    var step = 1;
    var infinite = false;

    if (args.length == 1 && typeof args[0] == 'number') {
        var i = 0, j = args[0];
    }
    else if (args.length == 1 && typeof args[0] == 'string') { // 'start[,next]..[end]'
        var arg = args[0];
        var startOpen = false, endClosed = false;
        if (arg[0] == '(' || arg[0] == '[') {
            if (arg[0] == '(') startOpen = true;
            arg = arg.slice(1);
        }
        if (arg.slice(-1) == ']') endClosed = true;

        var parts = arg.split('..');
        if (parts.length != 2)
            throw new Error("single argument range takes 'start..' or 'start..end' or 'start,next..end'");

        if (parts[1] == '') { // 'start..'
            var i = parts[0];
            infinite = true;
        }
        else { // 'start[,next]..end'
            var progression = parts[0].split(',');
            if (progression.length == 1) { // start..end
                var i = parts[0], j = parts[1];
            }
            else { // 'start,next..end'
                var i = progression[0], j = parts[1];
                step = Math.abs(progression[1]-i);
            }
        }

        i = parseInt(i, 10);
        j = parseInt(j, 10);

        if (startOpen) {
            if (infinite || i < j) i++;
            else i--;
        }

        if (endClosed) {
            if (i < j) j++;
            else j--;
        }
    }
    else if (args.length == 2 || args.length == 3) { // start, end[, step]
        var i = args[0], j = args[1];
        if (args.length == 3) {
            var step = args[2];
        }
    }
    else {
        throw new Error("range takes 1, 2 or 3 arguments");
    }
    var lazy = new Lazy;
    var stopInfinite = false;
    lazy.on('pipe', function () {
        stopInfinite = true;
    });
    if (infinite) {
        process.nextTick(function g () {
            if (stopInfinite) return;
            lazy.emit('data', i++);
            process.nextTick(g);
        });
    }
    else {
        process.nextTick(function () {
            if (i < j) {
                for (; i<j; i+=step) {
                    lazy.emit('data', i)
                }
            }
            else {
                for (; i>j; i-=step) {
                    lazy.emit('data', i)
                }
            }
            lazy.emit('end');
        });
    }
    return lazy;
}
```
- example usage
```shell
...
* lazy.tail(f)
* lazy.join(f)

The Lazy object itself has a .range property for generating all the possible ranges.

Here are several examples:

* Lazy.range('10..') - infinite range starting from 10
* Lazy.range('(10..') - infinite range starting from 11
* Lazy.range(10) - range from 0 to 9
* Lazy.range(-10, 10) - range from -10 to 9 (-10, -9, ... 0, 1, ... 9)
* Lazy.range(-10, 10, 2) - range from -10 to 8, skipping every 2nd element (-10, -8, ... 0, 2, 4, 6, 8)
* Lazy.range(10, 0, 2) - reverse range from 10 to 1, skipping every 2nd element (10, 8, 6, 4, 2)
* Lazy.range(10, 0) - reverse range from 10 to 1
* Lazy.range('5..50') - range from 5 to 49
...
```

#### <a name="apidoc.element.lazy.super_"></a>[function <span class="apidocSignatureSpan">lazy.</span>super_ ()](#apidoc.element.lazy.super_)
- description and source-code
```javascript
function EventEmitter() {
  EventEmitter.init.call(this);
}
```
- example usage
```shell
n/a
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
