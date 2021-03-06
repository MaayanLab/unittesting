# Functional testing
_Gregory Gundersen, Ma'ayan Lab meeting, 8 April 2015_

## Install Selenium

[Selenium](http://selenium-python.readthedocs.org/index.html) is open-source software for web browser automation. It is useful for testing because we can quickly and consistently mock a user of our web application. To install Selenium with `pip`, execute:

    pip install selenium

## Basics of functional testing

> **What is the difference between functional and unit testing?**

- A **unit test** verifies that an individual unit of code works as expected, isolating it and mocking its dependencies.
- A **functional test** verifies that a slice of functionality in a system works as expected. This may test many methods (or units), interact with dependencies, modify a database, use web services, and so on.

A unit test cannot detect that a required feature of the program is missing. A functional test can capture these requirements.

> **What is a "functional slice"?**

There isn't a precise definition, but I prefer to think of it as unit of functionality of client experience or expectation. For example, a functional test might verify that creating a new user or an API endpoint works as expected.

> **Why should I write functional test?**

For the [same reasons you write unit tests](https://github.com/MaayanLab/software-testing/blob/master/1-unit-testing/README.md): **stability**, **modularization**, **scalability**, etc.

> **When should I write functional tests?**

For scientists and researchers, unit tests are probably more important than functional tests because they allow us to directly verify that complex algorithms work—and continue to work—as expected. That said, functional tests can give you a measure of comfort in knowing that the basic aspects of your web application work across deployments.

Since many of our applications share a typical pipeline, `input gene list -> analyze list -> output results`, verifying that functional slice is probably a good start.

## Browser automation with Selenium

Since we primarily build web applications, we can write our functional tests by automating the browser with Selenium. Here's an example:

```python
from selenium import webdriver

# Create an instance of the Firefox web browser. There are drivers for other 
# browsers available, but this one ships with Selenium.
browser = webdriver.Firefox()

# Make a GET request to the URL passed in.
browser.get('http://www.google.com')

# Use Selenium's API to select and input text into the input field.
input_ = browser.find_element_by_id("lst-ib")
input_.send_keys('test')

# Select and click the submit button.
submit_btn = browser.find_element_by_id('sblsbb')
submit_btn.click()
```

## An automation script to print enrichment scores

Before we can test Enrichr's interface, we need to learn to navigate it. Let's complete this stub to perform crisp set enrichment, select a library, and hover over the results:

```python
import time

from selenium import webdriver
from selenium.webdriver.common.action_chains import ActionChains


browser = webdriver.Firefox()
browser.get('http://amp.pharm.mssm.edu/Enrichr/')

# 1. Select the button to use crisp gene set example
        
# 2. Select the submit button
        
# 3. Select a gene set library from the results

# Wait while Enrichr performs analysis
time.sleep(6)

# 4. Select enrichment terms and verify the first one
# HINT: Use `find_elements_by_css_selector()`
terms = ...

# Hover over enrichment term bar to see scores. Verify that all scores
# are correct
ActionChains(browser).move_to_element(terms[0]).perform()
tooltip = browser.find_element_by_css_selector('#aToolTip')
        
# 5. Print the scores

browser.quit()
```

## Wrapping Selenium with the `unittest` module

Last time, we discussed writing unit tests using Python's built-in `unittest` module. We can integrate our functional tests into the same framework, allowing us to execute unit and functional tests in the same suite:

```python
import unittest

from selenium import webdriver


class TestEnrichr(unittest.TestCase):

    def setUp(self):
        self.browser = webdriver.Firefox()
        self.browser.get('http://amp.pharm.mssm.edu/Enrichr/')

    def test_crisp_set_enrichment(self):
        # Add the code we just wrote here.
        pass
```


Below, we have functional tests for Enrichr's interface and API. We can use `nose` to run both tests as part of our test suite.

- [Testing Enrichr's interface](tests/test_enrichr_ui.py)
- [Testing Enrichr's API](tests/test_enrichr_api.py)

Remember, to run the tests with `nose`, execute:

    nosetests -v
    
## Degrees of coverage

Between unit and functional tests, we have test coverage for most of what we want:

- Unit tests to verify core algorithms, e.g. Fisher's exact test
- Functional tests to verify primary features, e.g. perform enrichment and verify results