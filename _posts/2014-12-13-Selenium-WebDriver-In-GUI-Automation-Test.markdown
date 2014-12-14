---
layout: post
title: "Selenium WebDriver In GUI Automation Test"
date: 2014-12-14 11:04:24
categories: IT ELK
---

##Selenium WebDriver Introduction

Selenium contains many tools that help us to test our application. The most famous is Selenium IDE, which is a firefox plugin-in. Using that is easy in the first glance, it record your actions as scripts and play it later, however, the maintenance is disaster. Selenium RC is another tools, which  boot a proxy to intercept the request that we sent and inject JavaScript to page to action certain movement. it was written purely in JavaScript for all the browser automation. The JavaScript, in Selenium RC, would then emulate user actions. This JavaScript would automate the browser from within the browser.

Compared with its sibling( Selenium RC and Selenium IDE) , WebDriver is control the browser from the outside the browser. The technical mechanism under the hood is to  uses accessibility API to driver the browser. If we look at Firefox, it uses JavaScript to access the API. If we look at Internet Explorer, it uses C++. Because of that, It won't restricted by sandbox mechanism that introduced by browsers for the sake of safety. However, the downside  is obvious. It will not support new browser but Selenium RC which use web lingua france——JavaScript will . More information about Selenium WebDriver could be found in [here](http://docs.seleniumhq.org/projects/webdriver/).

```flow

WebDriverAPI=>operation: WebDriver API
WebDriverSPI=>operation: WebDriver SPI
JSONWireProtocol=>operation: JSON Wire Protocol
SeleniumServer=>operation: Selenium Server

WebDriverAPI->WebDriverSPI->JSONWireProtocol->SeleniumServer

```

##Development Environment

###maven pom dependency

{% highlight xml %}
<dependency>
	<groupId>org.seleniumhq.selenium</groupId>
	<artifactId>selenium-java</artifactId>
	<version>2.44.0</version>
</dependency>
<dependency>
	<groupId>org.seleniumhq.selenium</groupId>
	<artifactId>selenium-server</artifactId>
	<version>2.44.0</version>
</dependency>
{% endhighlight %}

###Fetch WebDriver

Firefox Driver does not require a download as it is bundled with the Java client bindings. However, Chrome WebDriver need to be download explicitly. The download URL is in here [this](https://sites.google.com/a/chromium.org/chromedriver/downloads)(if it is invalid, please google `chrome webdriver`).

##Page Object Pattern

> A good thing about higher level testing like this, is that the tests will be pretty resilient to changes in your code. As long as the elements rendered on a page, and the routing stays the same, if you change the underlying implementation one day, your E2E tests should still pass.        
> ---- [Andy Shora](http://andyshora.com/unit-testing-best-practices-angularjs.html)

Maintain the element position code is pretty tricky. Especially, if you use process pattern to write these tests, you might spread these statement into different test cases, it will be a nightmare to maintain, once the element position varied.Intuitively, What about the elements position codes are encapsulated in one class which represent the page it attached? Here, we introduced the **Page Object** design pattern.  Applying this pattern, the testing code will be maintainable and cooperate with  [fluent interface](http://en.wikipedia.org/wiki/Fluent_interface) creating your own DSL so that people can see intent. For example, the following code will carry out *Log in* action.

{% highlight java %}

logInPage.typeUserName("username")
	 .typePassword("123")
	 .clickLogInBtn();

{% endhighlight %}

In **src/main/java** directory,  Page Objects are suggested to be put. Test cases are put in **src/test/java** as usual.

##Workflow

 1. prerequisite configuration (*src/main/resources/*)
 2. create a page object (if not exist)
 3. use Selenium IDE to help find certain element  
 4. wrap this element up in certain page object and provide [fluent interface](http://en.wikipedia.org/wiki/Fluent_interface) to access 
 5. write test cases in *src/test/main* with junit.
 6. run the test, debug and tweak code.

##Waiting Elements to appear

if target project is developed by SPA, which heavily rely on ajax to interact with server. So even if the whole page have load completely, elements can happen asynchronously so we never know when they will appear.

 In *SeleniumHelper.class* , two helper methods could be used to wait.
  
  1. *waitAndSleep* use thread sleep mechanism to wait. 
  2. *waitAMoment* use *Implicit Waits* provided by selenium to wait. 

>An implicit wait is to tell WebDriver to poll the DOM for a certain amount of time when trying to find an element or elements if they are not immediately available. The default setting is 0. Once set, the implicit wait is set for the life of the WebDriver object instance.

##Exception Handler
Exceptions are unavoidable, but some handy tools provided by selenium will help us debug and tweak our test code. There is a screen snapshot helper function in `SeleniumHelp` class. You could put this method in catch block to capture picture when the code crashed. 

##Reference
----------------------------------------
1. [Selenium 2 Testing Tools](https://www.packtpub.com/web-development/selenium-2-testing-tools-beginner%E2%80%99s-guide)
2. [Selenium WebDriver Documentation](http://docs.seleniumhq.org/projects/webdriver/)
3. [The Architecture of Open Source Application -- Selenium](http://www.aosabook.org/en/selenium.html)