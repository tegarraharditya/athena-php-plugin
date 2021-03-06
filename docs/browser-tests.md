# Introduction

For our basic usage introduction, we will be testing `google.com` interface. If you have not yet installed Athena, refer to Athena project page.

# Project Setup

To start using Athena, we'll need to setup a directory, where we will save our tests into.

```
myteam-tests/
├── GoogleBrowserTest.php
├── Report
└── athena.json
```

`athena.json` is where Athena reads the configuration from. In this case we will contain only the necessary information for a simple Browser test run.

`Report/` directory is where the HTML report, generated by Athena, will be output.

## The `selenium.hub_url`

The first thing you specify in `athena.json` file (when doing automated browser testing), is the `selenium.hub_url` key. You're telling Athena where the interface for manipulating the browser is located.

```json
{
  "selenium" : {
    "hub_url" : "http://athena-selenium-hub:4444/wd/hub"
  },
```

As you can see `selenium.hub_url` key, is pointing to our—local Selenium set-up.

`selenium-hub` docker container is linked with Athena's PHP container, and is where `athena-selenium-hub` is mapped.

Please refer to [Athena Selenium Plugin](https://athena-oss.github.io/plugin-selenium/) documentation.

## The `report`

One of our goals is to debug what actions were performed by the Browser. We don't need too much detail, but enough to understand what happened, specially in case something goes wrong.

```json
  "report" : {
    "format" : "html",
    "outputDirectory" : "./Report"
  }
}
```

Setting up a report is fairly easy, you just have to define the `report.format`, and `report.outputDirectory`. We will make use of our `./Report` directory to keep things nice and tidy.

# Writing a Test

```php
namespace Tests;

use Athena\Test\AthenaBrowserTestCase;

class GoogleBrowserTest extends AthenaBrowserTestCase
{
    public function testSearch_RandomSearchString_ShouldShowResultsPage()
    {

    }
}
```

At the first glance, it looks fairly standard: A namespace, a parent class and a big self-explanatory method name.

## The Namespace

One of the caveats of writing a test in Athena, is the namespace. It should—always—start with—`Tests\`.
Internally Athena will map `Tests\` to your testing directory.

This behaviour gives you freedom to choose how you organise the directory, where you store your tests.

## The Parent Class

Another close look to the code will make our parent class, `AthenaBrowserTestCase` stand-out, and you'll ask yourself what it does, if you didn't, you have now. When building our tests we should—always—include Athena's test cases.

Each type of test is wrapped with a Athena class of it's own, as you can imagine, this introduces custom behaviour when needed. I won't cover here all the types, as the focus is our Browser test case. After completing the guide, I recommend you giving them a—quick—look.

## The Method Name Convention

Adopting conventions, in most situations, is a good idea. This keeps things consistent, making it easier to read.

We follow the `MethodName_StateUnderTest_ExpectedBehavior` convention. If you are interested in knowing more about it, refer to the [Coding Standards](../best-practices/conventions.md) page.

Food for though: If you have however spent a lifetime using a different convention, you can always use your own. At the end of the day, it doesn't matter, consistency for the reader is the key. Is what we strive for. Easy and understandable tests.

## Manipulating The Browser

Accessing the Browser interface, is done through `Athena::browser()`. From this method on, you'll have access to a fluent interface, to interact with the DOM and/or browser navigation and configuration.

```php
namespace Tests;

use Athena\Athena;
use Athena\Test\AthenaBrowserTestCase;

class GoogleBrowserTest extends AthenaBrowserTestCase
{
    public function testSearch_RandomSearchString_ShouldShowResultsPage()
    {
        // 1.
        $currentPage = Athena::browser()->get('http://google.com');

        // 2.
        $currentPage->find()
            ->elementWithName('q')
            ->sendKeys("athena")
            ->click();

        // 3.
        $currentPage->findAndAssertThat()
            ->existsAtLeastOnce()
            ->elementWithCss('.sbsb_c');
    }
}
```

The first step we take, is to tell the browser to navigate to our `google.com` page. We did write the full URL, although this is not necessary.

Next step: We find the element, write the search string and click it. We know for a fact, that this will activate the autocomplete results box.

Even sure, we need to assert: at least one autocomplete result element exists. We do this, through the `findAndAssertThat()` method which, as the name says, finds our element and asserts the conditions are met.
If one of the conditions fails, an exception will be thrown.

# Execute The Test

Athena runs it's tests through the command line interface, so we'll need to navigate inside Athena's project directory, to access it's executable.

```bash
$ athena php browser
...

usage: athena php browser <browser-name> <tests-directory> <config-file> [<options>...] [(<phpunit-options>|<paratest-options>)...]

    <browser-name>                      Browser to be used. Such as firefox, phantomjs, or chrome
    <tests-directory>                   This directory will be mounted inside the docker container, and used to search for the tests
    <config-file>                       Athena config file, with proxy configurations, grid options, etc
    [--parallel=<number>]               Specify the number of jobs to be ran in parallel. In case this options is specified, Paratest will be ran, instead of PHPUnit
    [--php-version=<version>]           Switch between available PHP versions. E.g. --php-version=7.0
    [--override-athena-dependencies]    Override PHP plugin dependencies with the ones found inside the tests directory
    [--restore-athena-dependencies]     Restore PHP plugin original dependencies
```

Writing `./athena php browser` and hitting enter, will show you the basic usage, on the requirements to run a browser test case. Most likely by now you already know the next steps.

```bash
$ athena php browser firefox ../myteam-tests ../myteam-tests/athena.json
```

Once you run that command, if it is your first time running athena, you'll most likely see a lot of input. This is athena setting up it's docker images.

When it's all completed, or you are running the command a second time, after having everything installed, you should see the following:


```bash
...

PHPUnit 5.1.4 by Sebastian Bergmann and contributors.

.                                                                   1 / 1 (100%)

Time: 3.2 seconds, Memory: 5.25Mb

OK (1 test, 0 assertions)
```

# Reading The Report

In our configuration file, we've specified `Report/` as our output directory for the report file, so that where we will be looking for it.

Open `report.html` in your browser, and you should see nice HTML report containing all the steps we took, together with screenshots for each one.

# Configure Proxy and/or Grid Hub

When `athena php browser` is run, it will try to automatically link with a running Grid Hub or/and Proxy Server.

In case `--skip-proxy` or/and `--skip-hub` exists, the link will not be performed.

For performing a link with another running container, you can optionally specify `--link-proxy=<container_name>` and/or `--link-hub=<container_name>`.

# Parallel Tests

```bash
$ athena php browser firefox my-tests/ my-tests/athena.json --parallel=<number> [<paratest-options>...]
```

```bash
$ athena php browser firefox my-tests/ my-tests/athena.json --parallel=2
```

## All Available Options

```bash
$ athena php browser firefox my-tests/ my-tests/athena.json --parallel --help
...

Usage:
  paratest [options] [--] [<path>]

Arguments:
  path                                   The path to a directory or file containing tests. (default: current directory)

Options:
  -p, --processes=PROCESSES              The number of test processes to run. [default: 5]
  -f, --functional                       Run methods instead of suites in separate processes.
      --no-test-tokens                   Disable TEST_TOKEN environment variables. (default: variable is set)
  -h, --help                             Display this help message.
      --coverage-clover=COVERAGE-CLOVER  Generate code coverage report in Clover XML format.
      --coverage-html=COVERAGE-HTML      Generate code coverage report in HTML format.
      --coverage-php=COVERAGE-PHP        Serialize PHP_CodeCoverage object to file.
  -m, --max-batch-size=MAX-BATCH-SIZE    Max batch size (only for functional mode). [default: 0]
      --filter=FILTER                    Filter (only for functional mode).
      --whitelist=WHITELIST              Directory to add to the coverage whitelist.
      --phpunit=PHPUNIT                  The PHPUnit binary to execute. (default: vendor/bin/phpunit)
      --runner=RUNNER                    Runner or WrapperRunner. (default: Runner)
      --bootstrap=BOOTSTRAP              The bootstrap file to be used by PHPUnit.
  -c, --configuration=CONFIGURATION      The PHPUnit configuration file to use.
  -g, --group=GROUP                      Only runs tests from the specified group(s).
      --exclude-group=EXCLUDE-GROUP      Don't run tests from the specified group(s).
      --stop-on-failure                  Don't start any more processes after a failure.
      --log-junit=LOG-JUNIT              Log test execution in JUnit XML format to file.
      --colors                           Displays a colored bar as a test result.
      --testsuite[=TESTSUITE]            Filter which testsuite to run
      --path=PATH                        An alias for the path argument.
```

Once the option --parallel is set, under hood paratest will replace phpunit.
