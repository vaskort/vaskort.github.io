---
layout: post
title: An automated testing adventure with Chimp JS
---

Before a few months I was assigned a project to create the structure for automated
testing. There were a few things that I should have to complete were:

- Create the tests with a framework.
- Add my tests to the pre-existing Jenkins server for CI (Continuous integration - yep I
  didn't know what that meant in the beginning).
- Be able to trigger a Jenkins job when I push to develop.
- Send a success/fail message to a particular channel on Slack if the test passed/failed.

If all these don't make sense, don't worry, they didn't make to me as well. Read along.

### The framework to create the tests

After some tries with different kind of frameworks and ways to create the tests I ended using [Chimp JS](https://chimp.readme.io/) because I found it the most straight forward to setup and its using [WebdriverIO](http://webdriver.io/) which has a very good documentation of how to create the tests.

Basically, how I understand it was like using WebdriverIO without the hassle to setup everything (Selenium, Chromedriver etc.)

So lets get started...

So lets install ChimpJS (its better to install it globally)

{% highlight javascript %}
npm install -g chimp
{% endhighlight %}
