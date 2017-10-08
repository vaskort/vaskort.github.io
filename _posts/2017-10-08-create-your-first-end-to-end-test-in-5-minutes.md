---
layout: post
title: Create your first automated end-to-end test in 5 minutes (with Chimp JS)
---

About a year ago, while I was working at [Hopster](www.hopster.tv), I was assigned a project to create the structure for automated testing so we can test the functionality of some basic pages like registration, login etc. There were a few things that I should have taken into consideration to complete this project:

- Create the end-to-end tests with a framework (obviously).
- Create a Jenkins job to a pre-existing Jenkins server
- Be able to trigger the Jenkins job whenever I pushed to develop branch.
- Send a success/fail message to a particular channel on Slack if the test passed/failed.

I will try to cover the first bullet in this post, the development of the end-to-end tests.

### The framework to create the tests

After some tries with different kind of frameworks and ways to create the tests I ended using [Chimp JS](https://chimp.readme.io/) because I found it the most straight forward to setup and its using [WebdriverIO](http://webdriver.io/) which has a quite good documentation of how to create the tests.  
Also I used [Cucumber.js](https://cucumber.io/) to write the acceptance/end-to-end tests thinking that at some point it would be helpful for the QA team to write its one feature files and then I would create the step definitions. 

### Installation

So lets get started, lets install ChimpJS (its better to install it globally)

{% highlight bash %}
  $ npm install -g chimp
{% endhighlight %}

The folder structure that we need to have is the following:

{% highlight bash %}
  chimp-test
  │
  └───features
      │   test.feature
      │
      └───steps
          │   test.js

{% endhighlight %}

### Feature file

Let's define the test in our feature file.

{% highlight gherkin %}
  # features/test.feature
  @watch

  Feature: Search term
    As an ASOS user
    I want the search term to appear on top of the list view
    So that I know which product I searched

    Scenario: User sees the search term she searched for
    When the ASOS page loads
    And I fill the search term
    Then I should see the search term on top of the next page
{% endhighlight %}

Check a very useful article on how to to create a feature file [here](http://www.bbc.co.uk/blogs/internet/entries/ff14236d-098a-3565-b678-ff4ba5776a5f).

### Test file (Step definition)

At this point we can run `chimp --watch` in our rooter folder (`chimp-test`) so we can copy paste the steps to use them in our `test.js` folder.

![Chimp terminal screenshot]({{ "/images/chimp-test.png" | absolute_url }})

{% highlight javascript %}
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
{% endhighlight %}

##### First Step

The next step is to check the documentation of [Webdriver](http://webdriver.io/api.html) and write our automation code inside every step.
In our **first step** we want to just load the ASOS homepage: 
{% highlight javascript %}
this.Given(/^the ASOS page loads$/, function () {
  browser.url('http://www.asos.com');
});
{% endhighlight %}

##### Second Step

In our **second step** we want to add a value to the search input and click the search buttom
{% highlight javascript %}
this.When(/^I fill the search term$/, function () {
  browser.setValue('.search-box', 'Jackets');
  browser.click('div.search a.go');    
});
{% endhighlight %}

##### Third Step

In our **third and last step** we want to make a simple assertion that the Jackets text exist as the search term on top of the list page

{% highlight javascript %}
this.Then(/^I should see the search term on top of the next page$/, function () {
  var searchTerm = '.search-term';
  browser.waitForVisible(searchTerm);
  expect(browser.getText(searchTerm)).toEqual('Jackets');
});
{% endhighlight %}

Hopefully we would see something like the following 

![Chimp terminal success]({{ "/images/chimp-success.png" | absolute_url }})

Congratulations you made your first simple end-to-end test! 

### Notes - Links
Full code on GitHub: [https://github.com/vaskort/chimp-test](https://github.com/vaskort/chimp-test)  
Chimp JS documentation: [https://chimp.readme.io/](https://chimp.readme.io/)  
Chimp JS Slack channel: [http://community.xolv.io/](http://community.xolv.io/)



