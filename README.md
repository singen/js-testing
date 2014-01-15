### What we use to test user-side javascript code

[The Mocha testing framework](http://visionmedia.github.io/mocha/)

[A small fixtures library](https://github.com/badunk/js-fixtures)  to load html inside tests

[A small jquery ajax mocking library](http://github.com/appendto/jquery-mockjax) to mock ajax requests

[The chai.js assertions library](http://chaijs.com/) to make tests more BDD-like

[The chai jquery plugin](https://github.com/chaijs/chai-jquery) to make assertions with jquery

[The Sinon js mock library](https://github.com/cjohansen/Sinon.JS) to mock objects and use fake timers

### Some glue html

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Mocha Tests</title>
  <link rel="stylesheet" href="lib/mocha.css" />
  <script type="text/javascript" src="../js/require-2.1.8.min.js"></script>
  <script type="text/javascript">
    var runner;
    require.config({
      baseUrl: "/js/lib",
      paths: {
        app: "/js/app",
        lib: "/test/lib",
        test: "/test",
        jquery: 'jquery-1.8.3.min',
        underscore: 'underscore-min.1.3.3'
      }
    })
    require(["lib/mocha"], function() {
      mocha.setup('bdd')
      mocha.reporter('html')
      require([
        "test/MySpec",
      ], function() {
        mocha.checkLeaks()
        mocha.globals(['jQuery'])
        if (window.mochaPhantomJS) { mochaPhantomJS.run() }
        else { runner = mocha.run() }
      })
    })
  </script>
</head>
<body>
<div id="mocha"></div>
</body>
</html>
```
