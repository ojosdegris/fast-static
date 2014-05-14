fast-static
===========
Russian: https://github.com/Hkey1/fast-static/blob/master/README_ru-ru.md

fast-static is a node.js module, a simple build automation tool for front-end files.
You do not need to write special build files in order to use it.

In addition to that fast-static will enable in memory cache when ``` env=production ```.
This makes fast-static faster than default Express/Connect static middleware.


##Features
* Converts: coffee, haml, less, jade, sass, md
* Simple to include files (USE tag)
* Autodetect mime-type
* gzip
* browser cashing (ETag)

When ``` env=production ```
* minify css, js and html
* join files
* inserts small images into css
* in-memory-cash (compiled files, gzip results and ETag).


##Install
```
    npm install fast-static
```

##Middleware
Like static middleware in Express/Connect
```javascript
    var fastStatic= require('fast-static');
    app.use(onRoot,fastStatic.use(root,options));// app.use(onRoot,app.static(root,options));
```
Options is an optional parameter which is described in details below.
fast-static has options presets based on process.env.NODE_ENV value.

Example
```javascript
    var app = require('express')();
    var fastStatic= require('fast-static');
    app.use('/static',fastStatic.use('./static'));
    app.listen(process.env.PORT);
```
##Anser
returns parsed file
```javascript
    fastStatic.answer(req,res,pathToFile);
```
For example output processed ./static/intro/index.jade
```javascript
    var app = require('express')();
    var fastStatic= require('fast-static');
    app.use('/static',fastStatic.use('./static',options));
    app.get('/',function(req,res){
        fastStatic.answer(req,res,'./static/intro/index.jade');
    });
    app.listen(process.env.PORT);
```
##USE tag
Preprocess tag for including fast-static content.

```html
<USE>
    1.js
    1.css
    dir/
        2.sass
        2.coffee
        subdir1/
            3.less
            3.js
        subdir2/
            4.js
            4.css
        subdir3/5.css
</USE>

<html>
<head>
    <title>Hello world</title>
    <USE>.css</USE><!-- In this place will be inserted link (css) tags -->
</head>
<body>
    <USE>header.html</USE><!-- Include file header.html -->
    <h1>Hello world</h1>
    <USE>
        #this is comment

        # Use ! starting string to output text
        ! Hello world from Use tag <br />

        # Syntax for constants %host%
        ! host=%host% <br />

        # you can use env switcher [value if env=development|value if env=production]
        ! env=[dev|prod] <br />

        # Next line will be include jquery.js on development, and jquery.min.js on production
        http://site.com/jquery[|.min].js
    </USE>

    <USE>footer.jade</USE><!-- Include file footer.jade -->

    <USE>.js</USE><!-- In this place will be inserted script tags -->
</body>
</html>
```

##Warnings
###Pathes are relative to a file path in the file system
If you use fastStatic.answer you nead to add a ``` base ``` tag.

Example, if you send on homepage /static/intro/index.html, you nead to add in this file
```html
    <base href="http://<use>!%host%</use>/static/intro/" />
```

###On env=production cache is on
In this mode file changes will not lead to changes in the responses.
You nead to restart service by calling ``` fastStatic.dropCash() ```.

###Dont use for big files
Memory is limited.
If you have directory with big files (>1-2MB) use other middleware (for example app.static()) for that directory;
Move large files to another directory or use other static middlewares.

Example:
```javascript
    var app = require('express')();
    var fastStatic= require('fast-static');
    app.use('/static',fastStatic.use('./static',options));
    app.use('/bigFiles',app.static('./bigFiles',options));
    app.listen(process.env.PORT);
```

###HAML & JADE is static
There are no req/request in locals in this files.

##Options
Options         |                                                       | default (dev/prod)
-------------   | ----------------                                      |-------------
maxAge          | Browser cache maxAge in milliseconds.                 | 0
hidden          | Allow transfer of hidden files                        | false
redirect        | Redirect to trailing "/" when the pathname is a dir   | true
index           | Default file name                                     | 'index.html'
env             | 'production' or 'development'                         | process.env.NODE_ENV
gzip            | use gzip                                              | true
dateHeader      | send date header                                      | false
cash            | cash                                                  | false/true

###Options.filters
To disable filter you nead to set options.filters[filterName]=false
To enable filter you nead to set options.filters[filterName]=filterOptions


Filters below are ON by default

Filter          |                                                       | url
-------------   | ----------------                                      |-------------
coffee          | compiles coffescript                                  | https://github.com/jashkenas/coffee-script
haml            | compiles haml                                         | https://github.com/creationix/haml-js
jade            | compiles jade                                         | https://github.com/visionmedia/jade
less            | compiles less                                         | https://github.com/less/less.js
saas            | compiles saas                                         | https://github.com/andrew/node-sass
md              | compiles md                                           | https://github.com/chjj/marked
use             | compiles USE tag                                      |


This filters are ON only for Production environment

Filter          |                                                       | url
-------------   | ----------------                                      |-------------
min.css         | minimify css                                          | https://github.com/GoalSmashers/clean-css
min.js          | minimify js                                           | http://lisperator.net/uglifyjs
min.html        | minimify html                                         | http://kangax.github.io/html-minifier/
inline.img.css  | insert small images into css                          |



###Filters options
Each filter has exts and order options.
* exts= list of file extentions. For example "html,htm"
* order= integer order


###Options.filters['inline.img.css']

Options         |                                                                | default
-------------   | ----------------                                               |-------------
maxLen          | If len of image file >  maxLen image will not be insert in css | 4096  (4KB)

###Options.filters['use']

Options         |                                                                | default (dev/prod)
-------------   | ----------------                                               |-------------
tabLen          | Len of tab symbol in spaces                                    | 4
joinFiles       | join files into one                                            | false/true


##Writing you own filter
```javascript
    fastStatic.addFilter({
        exts:'tea,littea',// list of in extension
        ext:'js',// out extension or false if not changes
        order:100,// order 100 for compilers, 300 for minimizators
        fun:function(data,filename, ext, cb){
            //..
            cb(err,data);
        }
    });
```


