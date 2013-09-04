# PyFunct

PyFuncT stands for Python Functional Testing and it is a small framework that aims to help writing functional automated tests with python, to test web applications.

Principles:
* <b>Organization</b>: Provides a clean workflow to store element selectors, pages and config, helping you to keep your tests organized and reusable.

* <b>Flexibility</b>: PyFunct tests are fully built with python, giving you total flexibility. With frameworks that provide natural language-like programming, sometimes it can get really tricky to perform simple actions (like opening two browsers in the same test if you need to test live updates, for instance)

* <b>Freedom</b>:  Most of the framework is customizable and optional, so that you can choose not to use some parts of it or make a few tweaks, if necessary.

PyFunct includes:
* Pages (which holds element selectors and URLs)
* Actions
* Config (global configuration easily manageable)
* [Splinter](http://splinter.cobrateam.info/) driver compatibility, which includes selenium, phantomJS, zopetest and more

## Getting started

This should get you started with the basic functionality. You can look for more examples
in the `examples` folder.


### Step 1 - The test case
Here is a code snippet with two basic tests that do the same thing: A wikipedia search. The first uses pages and the second uses pages and actions. Both concepts will be explained ahead.

```python
from pyfunct import FunctTestCase

class MyTestCase(FunctTestCase):

    def test_searching_a_wiki(self):
        # This goes to the page and loads its elements.
        self.browser.open_page('wikipedia index')

        # Fill the search input with "Functional testing"
        self.browser.type(self.browser['search input'], 'Functional testing')

        # Submit the search by clicking the button
        self.browser.click_and_wait(self.browser['search button'])

        page_title = self.browser.page_title
        expected_title = 'Functional testing - Wikipedia, the free encyclopedia'
        self.assertIn(expected_title, page_title)

    def test_searching_a_wiki_using_actions(self):
        self.actions.perform_search(self.browser, 'Functional testing')

        expected_title = 'Functional testing - Wikipedia, the free encyclopedia'

        self.actions.assert_title_contains(self.browser, expected_title)

```
In the code above, we wrote a testcase that inherits from `FunctTestCase`. All PyFunct tests should inherit from it.
`FunctTestCase` is just a testcase that inherits from `unittest.TestCase` (python's native unittest testcase). Therefore, you can use its methods (such as `assertIn`).

Before each test execution, a browser instance is initialized. When the test is finished, it automatically exits.
If you want to create multiple browsers, all you need to do is call `self.create_browser()`.

### Step 2 - Creating pages
In Step 1 we've made references to `wikipedia index`, `search input` and `search button`. Those aliases were defined in the following Page class:
```python
from pyfunct import Page

class IndexPage(Page):

    page_name = 'wikipedia index'

    def get_url(self):
        return '/'

    @property
    def elements_selectors(self):
        return (
            ('search input', "//input[@name='search']", 'xpath'),
            ('search button', "#searchButton", 'css')
        )

```
All classes that inherit from Page and provide a `page_name` will be accessible by the self.browser instance.

### Step 3 - Creating Actions
In the second test (`test_searching_a_wiki_using_actions`), we used two actions: `perform_search` and `assert_title_contains`. By doing that, we accomplished the same thing we did in the fist test, but in a simpler and more reusable way. In order to write such actions and have them accessible through the `actions` attribute from a `FunctTestCase`, all you need to do is use the `@action` decorator, as follows:

```python

from pyfunct import action

@action
def perform_search(browser, query):
    # This goes to the page and loads it's elements selectors.
    browser.open_page('wikipedia index')

    # Fill the search input with "Functional testing"
    browser.type(browser['search input'], 'Functional testing')

    # Submit the search by clicking the button
    browser.click_and_wait(browser['search button'])

@action
def assert_title_contains(browser, expected_title):
    page_title = browser.page_title
    assert expected_title in page_title, "The expected title was not found in the page title"
```

### Step 4 - Manage your config
Until now, we haven't defined the browser driver or the base url. Pyfunct comes with a simple class-based configuration, which sets the global configuration attributes of your choice. Check it out:
```python
from pyfunct import BaseConfig

class WikipediaConfig(BaseConfig):
    base_url = 'http://en.wikipedia.org'
```

That's it, we've just set the global `base_url` config to wikipedia, since we are testing the wikipedia page.
There's no need to change the `default_driver_name`, since it's using splinter by default, unless you want to use another browser. In that case, check out the documentation.

### Step 5 - Run your tests
Currently, PyFunct does not provide a test runner and you can run it as you wish. A good choice for it is [nose](https://github.com/nose-devs/nose).
