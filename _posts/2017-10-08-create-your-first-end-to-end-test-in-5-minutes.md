---
layout: post
title: Create your first automated end-to-end test in 5 minutes (with Chimp JS)
---

About a year ago I was assigned a project to create the structure for automated end-to-end testing. This could give us the benefit to test the functionality of some basic pages like registration, login etc and generally give us a piece of mind that we didn't break anything critical. There were a few things that I should have taken into consideration to complete this project:

- Create the end-to-end tests with a framework (Chimp JS).
- Create a Jenkins job to a pre-existing Jenkins server.
- Trigger the Jenkins job whenever we pushed to the develop branch.
- Send a success/fail message to a Slack channel whenever a test passed or failed.

I will try to cover the _first bullet_ in this post, the development of the end-to-end tests.

### The framework to create the tests

After some tries with various frameworks I ended up using [Chimp JS](https://chimp.readme.io/) as I reckon it's pretty straight forward to setup and it's using [WebdriverIO](http://webdriver.io/) which has a quite good documentation.


Also I used [Cucumber.js](https://cucumber.io/) to write the acceptance/end-to-end tests thinking that at some point the QA team would jump in and create the feature files for me, then I would only have to create the step definitions. Keep reading if these don't make much sense yet, hopefully they will :) 

### Installation

So lets get started, lets install ChimpJS with npm (its better to install it globally).

```bash
  $ npm install -g chimp
```

The folder structure that we need is the following:

```bash
  chimp-test
  │
  └───features
      │   test.feature
      │
      └───steps
          │   test.js

```

### Feature file

Now let's define the user story in our feature file. Let's try to test something very simple in the [ASOS](http://www.asos.com) homepage, that whatever value we add in the search input, after clicking the search button, it would exist as a title in the following product list page.

```gherkin
  # features/test.feature
  @watch

  Feature: Search term
    As an ASOS user
    I want the search term to appear on top of the product list view
    So that I know which product I searched for

    Scenario: User sees the search term she searched for
    When the ASOS page loads
    And I fill the search term
    Then I should see the search term on top of the next page
```

Check a very useful and detailed article on how to to create a feature file [here](http://www.bbc.co.uk/blogs/internet/entries/ff14236d-098a-3565-b678-ff4ba5776a5f).

### Test file (Step definition)

At this point we can run `chimp --watch` in our root folder (`chimp-test`) so we can copy paste the steps from the terminal and use them in our `test.js` file.

![Chimp terminal screenshot]({{ "/images/chimp-test.png" | absolute_url }})

Create the `test.js` file like the following:  

```javascript
  // steps/test.js
  module.exports = function(){
    'use strict';

    this.Given(/^the ASOS page loads$/, function () {
      return 'pending';
    });
    
    this.When(/^I fill the search term$/, function () {
      return 'pending';
    });
    
    this.Then(/^I should see the search term on top of the next page$/, function () {
      return 'pending';
    });
  };
```

#### First Step

The next step is to check the documentation of [Webdriver](http://webdriver.io/api.html) and write our code inside every step.
In the **first step** we want to just navigate to the ASOS homepage:

```javascript
this.Given(/^the ASOS page loads$/, function () {
  browser.url('http://www.asos.com');
});
```

#### Second Step

In the **second step** we want to add a value to the search input and click the search button.

```javascript
this.When(/^I fill the search term$/, function () {
  browser.setValue('.search-box', 'Jackets');
  browser.click('div.search a.go');    
});
```

#### Third Step

In the **third and last step** we want to make a simple assertion to check that our search term exist as a title on top of the list page.

```javascript
this.Then(/^I should see the search term on top of the next page$/, function () {
  var searchTerm = '.search-term';
  browser.waitForVisible(searchTerm);
  expect(browser.getText(searchTerm)).toEqual('Jackets');
});
```

Hopefully if we rerun `chimp --watch` our tests should pass and we should see something like the following:

![Chimp terminal success]({{ "/images/chimp-success.png" | absolute_url }})

Congratulations! You made your first simple end-to-end test!  
Leave a comment if you want me to cover the rest of the bullet points listed in the beginning of the article.

### Notes - Links

Full code on GitHub: [https://github.com/vaskort/chimp-test](https://github.com/vaskort/chimp-test)  
Chimp JS documentation: [https://chimp.readme.io/](https://chimp.readme.io/)  
Chimp JS Slack channel: [http://community.xolv.io/](http://community.xolv.io/)