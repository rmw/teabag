Teabag
======
[![Build Status](https://travis-ci.org/modeset/teabag.png)](https://travis-ci.org/modeset/teabag)
[![Code Climate](https://codeclimate.com/github/modeset/teabag.png)](https://codeclimate.com/github/modeset/teabag)

Teabag is a Javascript test runner built on top of Rails. It can run tests in the browser, or headlessly using PhantomJS or with Selenium WebDriver.

It's objective is to be the simplest, while also being the most complete Javascript testing solution for Rails. It takes full advantage of the asset pipeline and ships with support for Jasmine, Mocha, QUnit, and (experiementally) Angular.

Ok, another Javascript test runner, right? Really? Yeah, that's tough, but we're pretty confident Teabag is one of the nicest and most full featured you'll find at the moment. And if you disagree, let us know and we'll probably fix whatever it is that you didn't like.

Feedback, ideas and pull requests are always welcome, or you can hit us up on Twitter [@modeset_](https://twitter.com/modeset_).

If you'd like to use Teabag with [Guard](https://github.com/guard/guard), check out the [guard-teabag](https://github.com/modeset/guard-teabag) project.

## Screenshots

### Running in the console
![Console Reporter](https://raw.github.com/modeset/teabag/master/screenshots/console-reporter.png)

### Running in the console using [Tapout](https://github.com/rubyworks/tapout)
![Console Reporter Tapout](https://raw.github.com/modeset/teabag/master/screenshots/console-reporter-tapout.png)

### Running in the browser
![Browser Reporter](https://raw.github.com/modeset/teabag/master/screenshots/html-reporter.jpg)


## Table of Contents

1. [Installation](#installation)
2. [Usage](#usage)
3. [Writing Specs](#writing-specs)
4. [Fixtures](#fixtures)
5. [Coverage](#coverage)
6. [Suites](#suites)
7. [Configuration](#configuration)
8. [Test Frameworks](#test-frameworks)
9. [Support Libraries](#support-libraries)
10. [CI Support](#ci-support)


## Installation

Add it to your Gemfile. In most cases you'll want to restrict it to the `:asset`, or `:development, :test` groups.

```ruby
group :assets do
  gem "teabag"
end
```

Optionally run the install generator to get the initializer and a basic spec helper.

```
rails generate teabag:install
```

You can tell the generator which framework you want to use, if you want coffeescript spec helper files, and if you want the env file created (used by the command line interface).

```
rails generate teabag:install --framework=mocha --coffee
```

You can optionally [install phantomjs](http://phantomjs.org/download.html).  If phantomjs is found it will be used, otherwise the gem will be used as a fallback -- which might not be optimal.


## Usage

Teabag uses the Rails asset pipeline to serve files. This allows you to use `= require` in your test files, and allows you use things like HAML or RABL/JBuilder within your fixtures.  You can run Teabag in three ways -- in the browser, via the rake task, and using the command line interface.

Here's a great [Quick Start Walkthrough](https://github.com/modeset/teabag/wiki/Quick-Start-Walkthrough) for writing and running your first tests.

### Browser

```
http://localhost:3000/teabag
```

You can focus tests in various ways, and Teabag provides an interface to run focus tests by suite, file, and filter.

### Rake

```
rake teabag suite=my_fantastic_suite
```

### Command Line Interface

The command line interface requires a `teabag_env.rb` file that you can get by running the generator. This file is used to load the Rails environment so Teabag can run within the context of Rails. This file can be in the spec, test, or root path -- but can be specified using the `--require` option.

```
bundle exec teabag
```

The CLI provides several ways of focusing tests. You can specify the suite to run, the files to run, and a filter.

```
bundle exec teabag --suite=my_fantastic_suite
bundle exec teabag spec/javascripts/calculator_spec.js
bundle exec teabag --filter="Calculator should add two digits"
```

Teabag also has support for [tapout](https://github.com/rubyworks/tapout). Use the tap_y formatter and pipe the results to tapout to use any of the reporters that tapout provides.

```
bundle exec teabag -q --format=tap_y | tapout progress
```

Get full command line help:

```
bundle exec teabag --help
```

**Note:** By default the rake task and command line interface run within the development environment, but you can specify the environment using `RAILS_ENV=test rake teabag`. This is to stay consistent with what you might see in the browser (since that's likely running in development).


## Writing Specs

Depending on what framework you use this can be slightly different. There's an expectation that you have a certain level of familiarity with the test framework that you're using. Right now Teabag supports [Jasmine](http://pivotal.github.com/jasmine), [Mocha](http://visionmedia.github.com/mocha) and [QUnit](http://qunitjs.com).

Since we have the asset pipeline at our fingertips you can feel free to use the `= require` directive throughout your specs and spec helpers.

Here's a basic spec written in Javascript using Jasmine:

```javascript
//= require jquery
describe("My great feature", function() {

  it("will change the world", function() {
    expect(true).toBe(true);
    expect(jQuery).toBeDefined();
  });

});
```

Check out examples of a [Mocha Spec](https://github.com/modeset/teabag/wiki/Using-Mocha), a [QUnit Test](https://github.com/modeset/teabag/wiki/Using-QUnit), and an [Angular Test](https://github.com/modeset/teabag/wiki/Using-Angular).

### Pending Specs

We've normalized declaring that a spec is pending between Mocha and Jasmine. Since Jasmine lacks the concept we've added it in, and since Mocha has several ways to accomplish it we thought it would be worth mentioning what we consider the standard between the two to be. QUnit doesn't support specifying a test as pending.

To mark a spec as pending you can either not provide a function as the second argument to `it`, or you can use `xit` and `xdescribe`. Mocha provides some additional ways to accomplish this, but to keep it consistent we've normalized on what they both support.

```coffeescript
describe "My great feature", ->

  it "hasn't been tested yet"

  xit "has a test I can't figure out" ->
    expect("complexity").to.be("easily testable")

  xdescribe "A whole section that I've not gotten to", ->

    it "hasn't been tested yet", ->
      expect(true).to.be(false)
```

If you're using a specific framework and you want to take advantage of the things that framework provides you're free to do so. This is provided as the standard as the Teabag reporters understand the techniques above and have specs for them.

### Using Require.js
If you are using require.js to get your files you can set a configuration option for your suite of "use_require".
```ruby
Teabag.setup do |config|
  config.suite do |suite|
    suite.use_require = true
  end
end
```
Then in your suite spec helper, add require.js to be included, if you have not already. (Note: Teabag doesn't include require.js with it, so you will need to provide your own require.js and require the correct path.)
```javascript
//= require require
```

Once you've done that, when that suite is executed, Teabag will use require.js to get all the specs in the suite (or specific files). In your specs you will need to use require to pull down the dependencies as you would normally. Here is an example with mocha.
```javascript
define(['Model'] , function (Model) {
  describe('Model' , function () {
    // put your tests here
  });
});
```

### Deferring Execution

Teabag allows deferring execution in the cases when you're using AMD or other asynchronous libraries. This is especially useful if you're using [CommonJS](http://www.commonjs.org/), etc.  You can tell Teabag to defer and then execute the runner yourself later -- after loading asychronously. There's a wiki article about how you can setup your specs and spec helper when using [RequireJS with Teabag](https://github.com/modeset/teabag/wiki/RequireJS-with-Teabag).

```javascript
Teabag.defer = true;
setTimeout(Teabag.execute, 1000); // defers execution for 1 second
```


## Fixtures

You're free to use your own fixture library (like jasmine-jquery, which we've included as a support library), but Teabag ships with a fixture library that works with Jasmine, Mocha, and QUnit with a minimum of effort, has a nice consistent API, and isn't dependent on jQuery.

The fixture path is configurable within Teabag, and the views will be rendered by a standard controller.  This allows you to use things like RABL/JBuilder if you're building JSON, or HAML if you're building markup.  The element that Teabag creates is "#teabag-fixtures", in case you need to access it directly -- or you can access it via `fixture.el` after loading fixtures.

### Loading Files

Loading fixtures allows you to specify any number of files to load, and if they should be appended to the fixture element, or replace what's currently there.

`fixture.load(url[, url, ...], append = false)` or `fixture(url[, url, ...], append = false)`

### Setting Manually

If you don't want to load files directly from the server you can provide strings instead of files, otherwise behaves like load.

`fixture.set(html[, html, ...], append = false)`

### Cleaning Up

You shouldn't have to cleanup (we do that for you based on your test framework), but if you need it.

`fixture.cleanup()`

### Preloading Files

Some test cases require stubbing Ajax requests, and in those cases you may want to preload the fixture files -- which caches them for later.  You can preload fixtures in your spec helper, or before you start mocking Ajax methods.

`fixture.preload(url[, url, ...])`

### Example Usage

```coffeescript
fixture.preload("fixture.html", "fixture.json") # make the actual requests for the files
describe "Using fixtures", ->

  fixture.set("<h2>Another Title</h2>") # create some markup manually (will be in a beforeEach)

  beforeEach ->
    @fixtures = fixture.load("fixture.html", "fixture.json", true) # append these fixtures which were already cached

  it "loads fixtures", ->
    expect($("h1", fixture.el).text()).toBe("Title") # using fixture.el as a jquery scope
    expect($("h2", fixture.el).text()).toBe("Another Title")
    expect(@fixtures[0]).toBe(fixture.el) # the element is available as a return value and through fixture.el
    expect(@fixtures[1]).toEqual(fixture.json[0]) # the json for json fixtures is returned, and available in fixture.json
```

Check out some example of using fixtures with [Mocha](https://github.com/modeset/teabag/wiki/Using-Mocha), [QUnit](https://github.com/modeset/teabag/wiki/Using-QUnit), and [Angular](https://github.com/modeset/teabag/wiki/Using-Angular).


## Coverage

Teabag can use [Istanbul](https://github.com/gotwarlost/istanbul) to generate code coverage statistics and reports. Install Istanbul and adjust the configuration to always generate coverage reports, or specify by passing `--coverage` to the command line interface. Check the [configuration](#configuration) for more information.

Each suite allows you to specify which files should be ignored when generating coverage reports, which allows you to ignore support libraries and dependencies that you're not testing.

The following example will generate a simple text report and an HTML report with annotated source that you can inspect further.

```shell
bundle exec teabag --coverage-reports=text,html
```

An example text report that's output to the console after the tests have completed.
```
--------------------+-----------+-----------+-----------+-----------+
File                |   % Stmts |% Branches |   % Funcs |   % Lines |
--------------------+-----------+-----------+-----------+-----------+
  phantomjs/        |     93.75 |        75 |     94.12 |     93.65 |
    runner.coffee   |     93.75 |        75 |     94.12 |     93.65 |
--------------------+-----------+-----------+-----------+-----------+
All files           |     93.75 |        75 |     94.12 |     93.65 |
--------------------+-----------+-----------+-----------+-----------+
```


## Suites

Teabag uses the concept of suites to group your tests at a high level. These suites are run in isolation from one another, and can have different configurations. You can define suites in the configuration, and for brevity `config` is the argument passed to the `Teabag.setup` block.

When creating a suite, provide a name (optional) and a block. The following example defines a suite named "my_suite". You can focus run this suite by browsing to `/teabag/my_suite` or running the rake task with `suite=my_suite`.

```ruby
config.suite :my_suite do |suite|
  suite.helper = "my_spec_helper.js"
end
```

There's always a "default" suite defined, and to modify this suite just don't specify a name, or use `:default`. In this example we're setting the default suite configuration.

```ruby
config.suite do |suite|
  suite.helper = "other_spec_helper.js"
end
```

**Note:** Suites inherit from the default suite, so default configuration will propagate to all other suites.

### Manifest Style

Teabag is happy to look for files for you, but you can disable this feature and maintain a manifest yourself.  Each suite can utilize a different spec helper and you can use these to create your own manifest using the `= require` directive. This isn't recommended because it limits your abilities to run specific files from the command line interface, but it's available if you want to use it.

Tell the suite that you don't want it to match any files, and then require files in your spec helper.

```ruby
config.suite do |suite|
  suite.matcher = nil
  suite.helper = "spec_manifest"
end
```

### Suite Configuration Directives

<dl>

<dt> matcher </dt><dd>
  You can specify a file matcher and all matching files will be loaded when the suite is run. It's important that these files can be served via sprockets / are within an asset path.<br/><br/>

  <b>Note:</b> Can also be set to <code>nil</code>.<br/><br/>

  <b>default:</b> <code>"{spec/javascripts,app/assets}/**/*_spec.{js,js.coffee,coffee}"</code>
</dd>

<dt> helper </dt><dd>
  Each suite can load a different spec helper, which can in turn require additional files. This file is loaded before your specs are loaded, and can be used as a manifest.<br/><br/>

  <b>default:</b> <code>"spec_helper"</code>
</dd>

<dt> javascripts </dt><dd>
  These are the core Teabag javascripts. It's strongly encouraged to include only the base files here. You can require other support libraries in your spec helper, which allows you to change them without having to restart the server.<br/><br/>

  <b>Note:</b> To use the CoffeeScript source files use <code>"teabag/jasmine"</code> etc.<br/><br/>

  <b>available:</b> teabag-jasmine, teabag-mocha, teabag-qunit<br/>
  <b>default:</b> <code>["teabag-jasmine"]</code>
</dd>

<dt> stylesheets </dt><dd>
  If you want to change how Teabag looks, or include your own stylesheets you can do that here. The default is the stylesheet for the HTML reporter.<br/><br/>

  <b>Note:</b> Spec related CSS can and should be loaded using fixtures.

  <b>default:</b> <code>["teabag"]</code>
</dd>

<dt> no_coverage </dt><dd>
  When running coverage reports, you probably want to exclude libraries that you're not testing.
  Accepts an array of filenames or regular expressions. The default is to exclude assets from vendors or gems.<br/><br/>

  <b>default:</b> <code>`[%r{/lib/ruby/gems/}, %r{/vendor/assets/}, %r{/support/}, %r{/(.+)_helper.}]`</code>
</dd>

</dl>


## Configuration

The best way to read about the configuration options is to generate the initializer and env, but we've included the info here as well.

**Note:** `Teabag.setup` may not be available in all environments, so the generator wraps it within a check.

<dl>

<dt> mount_at </dt><dd>
  This determines where the Teabag routes will be mounted. Changing this to "/jasmine" would allow you to browse to http://localhost:3000/jasmine to run your specs.<br/><br/>

  <b>default:</b> <code>"/teabag"</code>
</dd>

<dt> root </dt><dd>
  This defaults to Rails.root if left nil. If you're testing an engine using a dummy application it can be useful to set this to your engines root.. E.g. <code>Teabag::Engine.root</code><br/><br/>

  <b>default:</b> <code>nil</code>
</dd>

<dt> asset_paths </dt><dd>
  These paths are appended to the Rails assets paths (relative to config.root), and by default is an array that you can replace or add to.<br/><br/>

  <b>default:</b> <code>["spec/javascripts", "spec/javascripts/stylesheets", "test/javascripts", "test/javascripts/stylesheets"]</code>
</dd>

<dt> fixture_path </dt><dd>
  Fixtures are rendered through a standard controller. This means you can use things like HAML or RABL/JBuilder, etc. to generate fixtures within this path.<br/><br/>

  <b>default:</b> <code>"spec/javascripts/fixtures"</code>
</dd>

</dl>

### Console Runner Specific (Teabag Env)

These configuration directives are applicable only when running via the rake task or command line interface and should be set within the teabag_env.rb file. You can get this file by running the generator.

<dl>

<dt> driver </dt><dd>
  Allows you to specify which driver to use when running headlessly. Supports <a href="http://phantomjs.org">PhantomJS</a> and <a href="http://seleniumhq.org/docs/03_webdriver.jsp">Selenium Webdriver</a>.<br/><br/>

  Check this wiki article for information about <a href="https://github.com/modeset/teabag/wiki/Using-Selenium-WebDriver">Using Selenium WebDriver</a>, and this one about <a href="https://github.com/modeset/teabag/wiki/Using-PhantomJS">Using PhantomJS</a>.<br/><br/>

  <b>available:</b> phantomjs, selenium<br/>
  <b>default:</b> <code>"phantomjs"</code>

  <ul>
    <li>CLI: -d, --driver DRIVER</li>
    <li>ENV: DRIVER=selenium</li>
  </ul>
</dd>

<dt> server </dt><dd>
  Specify a server to use with Rack (eg. thin, mongrel). If nil is provided Rack::Server is used.<br/><br/>

  <b>default:</b> <code>nil</code>

  <ul>
    <li>CLI: --server SERVER</li>
    <li>ENV: SERVER=thin</li>
  </ul>
</dd>

<dt> server_timeout </dt><dd>
  Timeout for starting the server in seconds. If your server is slow to start you may have to bump this, or you may want to lower this if you know it shouldn't take long to start.<br/><br/>

  <b>default:</b> <code>20</code>

  <ul>
    <li>CLI: --server-timeout SECONDS</li>
    <li>ENV: SERVER_TIMEOUT=10</li>
  </ul>
</dd>

<dt> server_port </dt><dd>
  By default Teabag will locate an open port when starting the server, but if you want to run on a specific port you can do so by providing one.<br/><br/>

  <b>default:</b> <code>nil</code>

  <ul>
    <li>CLI: --server-port PORT</li>
    <li>ENV: SERVER_PORT=31337</li>
  </ul>
</dd>


<dt> fail_fast </dt><dd>
  If you have several suites it can be useful to make Teabag fail directly after any suite contains failures, but in environments like CI this may not be desirable.<br/><br/>

  <b>default:</b> <code>true</code>

  <ul>
    <li>CLI: --[no-]fail-fast</li>
    <li>ENV: FAIL_FAST=false</li>
  </ul>
</dd>

<dt> formatters </dt><dd>
  You can specify the formatters that Teabag will use when outputting the results.<br/><br/>

  <b>available:</b> dot, tap, tap_y, swayze_or_oprah<br/>
  <b>default:</b> <code>"dot"</code>

  <ul>
    <li>CLI: -f, --format FORMATTERS</li>
    <li>ENV: FORMATTERS=dot,swayze_or_oprah</li>
  </ul>
</dd>

<dt> suppress_log </dt><dd>
  Teabag pipes all console[log/debug/error] calls to STDOUT. This is useful to catch places where you've forgotten to remove them, but in verbose applications this may not be desirable.<br/><br/>

  <b>default:</b> <code>false</code>

  <ul>
    <li>CLI: -q, --[no-]suppress-log</li>
    <li>ENV: SUPPRESS_LOG=true</li>
  </ul>
</dd>

<dt> color </dt><dd>
  Specify if you want color output by default.<br/><br/>

  <b>default:</b> <code>true</code>

  <ul>
    <li>CLI: -c, --[no-]color</li>
    <li>ENV: COLOR=false</li>
  </ul>
</dd>

<dt> coverage </dt><dd>
  Add instrumentation to your code and display coverage information. Requires <a href="https://github.com/gotwarlost/istanbul">istanbul</a>.<br/><br/>

  <b>default:</b> <code>false</code>

  <ul>
    <li>CLI: -C, --coverage</li>
    <li>ENV: COVERAGE=true</li>
  </ul>
</dd>

<dt> coverage_reports </dt><dd>
  Specify which code coverage reports instanbul should generate.<br/><br/>

  <b>available:</b> text-summary, text, html, lcov, lcovonly, cobertura<br/>
  <b>default:</b> <code>nil</code>

  <ul>
    <li>CLI: -R, --coverage-reports REPORTS</li>
    <li>ENV: COVERAGE_REPORTS=text,html</li>
  </ul>
</dd>

</dl>


## Test Frameworks

[Jasmine](http://pivotal.github.com/jasmine) is used by default unless you specify otherwise. We've been using Jasmine for a long time, and have been pretty happy with it. It lacks a few important things that could be in a test framework, so we've done a little bit of work to make that nicer. Like adding pending spec support.

[Mocha](http://visionmedia.github.com/mocha) came up while we were working on Teabag -- we read up about it and thought it was a pretty awesome library with some really great approaches to some of the things that some of us browser types should consider more often, so we included it and added support for it. We encourage you to give it a try. Read more about [Using Mocha](https://github.com/modeset/teabag/wiki/Using-Mocha) with Teabag.

[QUnit](http://qunitjs.com) We're not sure about how many people use QUnit, but we like jQuery, so we added it. Read more about [Using QUnit](https://github.com/modeset/teabag/wiki/Using-QUnit) with Teabag.

[Angular](http://angularjs.org/) This is an experimental addition, and feedback is needed. Read more about [Using Angular](https://github.com/modeset/teabag/wiki/Using-Angular) with Teabag.


## Support Libraries

We know that testing usually requires more than just the test framework, so we've included some of the libraries that we use on a regular basis.

- [Sinon.JS](http://sinonjs.org) Standalone test spies, stubs and mocks for JavaScript. No dependencies, works with any unit testing framework.
- [ChaiJS](http://chaijs.com/) BDD / TDD assertion library for node and the browser that can be delightfully paired with any javascript testing framework.
- [expect.js](https://github.com/LearnBoost/expect.js) Minimalistic BDD assertion toolkit based on should.js.
- [jasmine-jquery.js](https://github.com/velesin/jasmine-jquery) A set of custom matchers for jQuery, and an API for handling HTML fixtures in your specs.
- [angular-scenario.js](https://github.com/angular/angular.js) Angular test setup.

You can require these files in your spec helper by using:

```javascript
//=require support/sinon
//=require support/chai
//=require support/expect
//=require support/jasmine-jquery
//=require support/angular-scenario
```


## CI Support

Teabag works great on CI setups. If you're using TravisCI it just works, but if you're using something else all you should need is to ensure phantomjs is installed.

If you want to generate reports that CI can use you can install istanbul for coverage reports -- and output the report using the cobertura format, which Hudson can read.

Again on hudson compatibile CI setups, you can track spec failure information/rate tracking by using the tap formatter, which can be parsed by hudson.

A good setup:

```
teabag -q --coverage-reports=cobertura --format=tap
```

Or using Rake/ENV:

```
SUPPRESS_LOG=true COVERAGE_REPORTS=cobertura FORMATTERS=tap rake
```

There is also a TeamCity formatter:

```
teabag -q --format=teamcity
```

Or using Rake/ENV:

```
SUPPRESS_LOG=true FORMATTERS=teamcity rake
```

## License

Licensed under the [MIT License](http://creativecommons.org/licenses/MIT/)

Copyright 2012 [Mode Set](https://github.com/modeset)

All licenses for the [bundled Javascript libraries](https://github.com/modeset/teabag/tree/master/vendor/assets/javascripts) are included (MIT/BSD).


## Make Code Not War
![crest](https://secure.gravatar.com/avatar/aa8ea677b07f626479fd280049b0e19f?s=75)

