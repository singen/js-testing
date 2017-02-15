### What we use to test user-side javascript code

[The Mocha testing framework](http://mochajs.org/)

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

### Some glue code

```javascript
define(["lib/fixtures", "underscore"], function(fixtures) {
  function loadPage(options) {
    var mainDataRegex = / data-main="([^"]*)"/
    fixtures.cleanUp()
    var html = fixtures.read(options.url)
    var mainData = html.match(mainDataRegex)[1] + ".js"
    fixtures.set(html.replace(mainDataRegex, ""))
    $('#js-fixtures').show()
    fixtures.window().onload = function() {
      var r = fixtures.window().require
      r.config({
        shim: { "lib/jquery.mockjax": { deps: ["jquery"] } },
        baseUrl: "/js/lib",
        paths: {
          lib: "/test/lib",
          jquery: 'jquery-1.8.3.min'
        }
      })
      r(["lib/sinon-1.7.3", "lib/jquery.mockjax"], function() {
        r.config({ baseUrl: '/js', map: options.dependenciesMap })
        options.beforeMainData && options.beforeMainData()
        r([mainData], function(main) {_.each(main, function(value, key) {
          if(_.isFunction(value.done) && options.callbacks[key]){
            value.done(function(){
              try {
                options.callbacks[key](arguments)
              } catch(e) {
                options.onError(e)
              }
            })
          }
        })})
      })
    }
  }

  function waitsFor(latchFunction, timeoutMessage, timeout, done) {
    var INTERVAL = 10
    var timeleft = timeout
    function test() {
      if(latchFunction()) {
        done()
      } else if(timeleft > 0) {
        timeleft -= INTERVAL
        setTimeout(test, INTERVAL)
      } else {
        done(timeoutMessage)
      }
    }
    test()
  }

  function waitForPageChange(regex, done) {
    waitsFor(function() {
      return fixtures.window().location.href.match(regex)
    }, "timeout while waiting page to change to match " + regex, 1000, done)
  }

  function clockDependent(clock) {
    function type(input, text) {
      for(var i = 0; i <= text.length; i++) {
        input.val(text.substring(0, i)).trigger('keyup')
        clock.tick(30)
      }
      clock.tick(200)
      return input
    }

    function backspace(input) {
      var text = input.val()
      input.val(text.substring(0, text.length - 1)).trigger('keyup')
      clock.tick(200)
      return input
    }

    return {
      type: type,
      backspace: backspace
    }
  }

  function mockjax(url, response, status) {
    return fixtures.window().$.mockjax({
      url: url,
      status: status || 200,
      responseTime: 0,
      dataType: 'json',
      contentType: 'application/json',
      responseText: JSON.stringify(response)
    })
  }

  function mockjaxForm(url, response, status) {
    return fixtures.window().$.mockjax({
      url: url,
      status: status || 200,
      responseTime: 0,
      responseText: response
    })
  }

  return {
    loadPage: loadPage,
    waitForPageChange: waitForPageChange,
    clockDependent: clockDependent,
    mockjax: mockjax,
    mockjaxForm: mockjaxForm
  }
})
```
