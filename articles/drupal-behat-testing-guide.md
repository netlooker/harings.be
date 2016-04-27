# Behat introduction for Drupal

* [What is Behat?](#whatisbehat)
* [Running tests](#runningtests)
* [Feature files](#featurefiles)
	* [The feature definition](#featuredefinition)
    * [Scenarios](#scenarios)
    * [Scenario outlines](#scenariooutline)
* [Contexts](#contexts)
    * [What are contexts](#whatarecontexts)
    * [Creating new contexts](#creatingnewcontexts)
* [Usefull tips](#usefulltips)
    * [Tags](#tags)
    * [Debugging](#debugging)
    	* [Xdebug / Phpstorm](#xdebug)
        * [Viewing pages](#viewingpages)
* [What to test](#whattotest)

## <a name="whatisbehat"></a> What is Behat?

Behat is an open source Behavior Driven Development framework for PHP 5.3+.

It allows us to write human understandable test scenarios. Because
the scenarios are easy to write, they could be written by someone
that has no knowledge of coding.

We tell Behat what it has to doe by writing it down.

## <a name="runningtests"></a> Running tests

It is for us, important to have the correct configuration before we build
our environments `build-dev`.

The build.properties.local needs to contain:
```
behat.base_url = http://information.local/
behat.test_tags = @information,@shared
```

The actual tags are depending on the project you are working on, the 
tags to be used can be found in the **build.properties.dist.PROJECT.local**

Before we start creating tests, we quickly show how to run Behat 
tests. There is much more possible then covered here, but this 
should give you a kickstart into behat.

When we go into the */tests* folder in our project root we see a behat and behat.yml file.

If this file is not present, you can run `./bin/phing setup-behat` from the project root
to regenerate it.

The behat.yml file contains our configuration.

The behat file, is our executable.

Awesome, using these 2 files, we have basically everything we need.

Let's open the terminal and navigate to the tests folder. We can run
the following commands:

```bash
./behat
```

This simply runs behat using our default config file. 
It wil also run every test it can find in the current folder.

More options:

```bash
; Running a specific test:
./behat features/inpage_nav.feature

; Show all possible commands:
./behat -h
```

I will not go in depth, but [the Behat documentation is pretty solid](http://docs.behat.org/en/v2.5/guides/6.cli.html).

## <a name="featurefiles"></a> Feature files

Feature files are the files that contain the actual test.
The filename should be simple, but should be clear enough to know 
what it is about.

### <a name="featuredefinition"></a> The feature definition

Every feature file starts with a **feature** block:

```gherkin
Feature: User authentication
  In order to protect the integrity of the website
  As a product owner
  I want to make sure only authenticated users can access the site administration
```

Besides telling you what the tests are about, they have no function. 
They should however, be short and descriptive enough to tell users 
what is going to be tested.

### <a name="scenarios"></a> Scenarios

Each scenario is a test. A scenario starts with telling that it is a 
scenario:

```gherkin
Scenario: Anonymous user can see the user login page
```

So, here we tell that what we are going to test is, that anynymous 
users can see the user login page. Once again, this is a purely 
informative line, but it is required.

Next we can start defining our steps. During this presentation it 
wil be impossible to cover all of the possibility's, but you can run 
the following command inside the folder where your tests live, to 
get a good idea.

```bash
$ ./behat -dl

Output:
...
default | Then I should not see the message( containing) :message
default | Given I am an anonymous user
default | When /^(?:|I )press "(?P<button>(?:[^"]|\\")*)"$/
...
```

As you can see, there are in the example output above, 3 differten 
starting words. There are a total of 5:

> *Given*, *When*, *Then*, *But* and *And*

Or if you are looking for something specific you can run:

```bash
$ ./behat -dl | grep drush

Output:
...
default | Given I run drush :command
default | Given I run drush :command :arguments
default | Then drush output should contain :output
default | Then drush output should match :regex
default | Then drush output should not contain :output
default | Then print last drush output
...
```

Now that we know, what we can run when testing, we can start 
building our test. We'll continue by adding our first step.

```gherkin
Given I am not logged in
```

That's clear. We simple tell behat that in this scenario, we are not 
logged in. 

Next step:

```gherkin
When I go to "user"
```

This is also self explaining, we said we were testing that anonymous 
users can access the login page.

So what we have done so far is:
1. We went to the website as an anonymous users.
2. We went to the page "user"

Now we have to finalize our check. We must make sure that we are 
seeing a login page and not the actual user page.

We will no longer define *when*, but instead we'll use *then*, *and* 
or *but*.

I will group the rest of the feature as it will be easier to 
understand.

```gherkin
Then I should see the text "Log in"
And I should see the text "Request new password"
And I should see the text "Username"
And I should see the text "Password"
But I should not see the text "Log out"
And I should not see the text "My account"
```

There we go, by adding the actual *checks* our scenario is now 
complete. As Behat uses the [*cucumber*][cucumber] all steps should 
be human readable, also by someone that is not involved in coding.

As seen with the example, the possibility's out of the box are 
endless. In the following example we'll do a more difficult test.

### <a name="scenariooutline"></a> Scenario outline

A scenario outline, is a more advanced version of the basic 
*scenario*.

The idea of a scenario outline, is to run a same set of steps, for 
multiple scenarios.

To define a scenario outline, we simply use it as our first line.

Let's go thorough an example: (announcements_block.feature)

```gherkin
Scenario Outline: Visitors should see the latests announcements block on content types
```

What we are going to test, is that users, can see the announcement 
block on various content types.

```gherkin
Given I am logged in as a user with the "administrator" role
```

This one is a little bit different, as we will have to access 
administration pages later in the test, we can already login as an 
administrator. This login session will remain as long as the current 
scenario runs.

Before we start to add our next step, we will put the following 
contents, at the bottom of the scenario. It should also remain at 
this position, one break lower then the last step.

```gherkin
Examples:
  | content_name         | reference_fieldname         | label_visible |
  | Department           | field_core_department       | should        |
  | Policy               | field_core_policies         | should not    |
```

So what we just dit, was defining a set of examples. This means that 
the test scenario, will run twice. Once for the department, and once 
for the policy.

The table heads we have defined here, are not bound to anything, the 
text is up to you to choose.

Let's go to the next step and see how we are going to use this data.

```gherkin
Given "<content_name>" content:
  | title                             | status | field_core_description |
  | <content_name> with announcements | 1      | Sample description     |
```

Loads of new stuff here, what we did here is the following:

We create a Department/Policy content, with a title, status and 
description.

<content_name> gets replaced by it's corresponding table in the 
*Examples*. This way we can make more complex test, and avoid to 
much copy pasting.

As we now have our regular content, we need announcements, as you 
remember, we added reference_fieldname to our examples table, this 
will be used now to make the acual entity reference.

```gherkin
Given "Announcement" content:
  | title                          | status | <reference_fieldname>             |
  | Announcement on <content_name> | 1      | <content_name> with announcements |
```

An entity reference in Behat, uses just the title of the target 
entity. In case you are having a field like a link you can do the 
following to fill in the title and url:

```gherkin
Title - http://google.be/
```

We now have a content, linked to the announcement. Next we should 
make our checks again. This we can do like this:

```gherkin
When I go to "admin/content"
And I follow "<content_name> with announcements"
Then I <label_visible> see an ".field--announcement-block h2" element
Then I should see the link "Announcement on <content_name>"
```

What we do here is, we go to the content overview page, then we 
click on the link "Policy/Department with announcements". 
After we are then on the page, we check if we can see the 
announcement label. 

And we check if we can see the link towards the annoucement.

Once again our test is now complete. We can play with the examples 
the way we want. A good example for this is the <label_visible>.

We can, by putting example variables inside our step, make the step 
dependent on the example.

## <a name="contexts"></a> Contexts

### <a name="whatarecontexts"></a> What are contexts

Contexts are methodes containing the code behind a step.

Now, lets dive into the more technical side, Context.

Contexts is the code side implementation of steps (simply put). It 
allows us to define custom events and allow us to do things with 
Behat, that are specific to a project.

Instead of trying to explain contexts, we'll just jump ahaid to 
**Creating context** which will go more in depth on them.

### <a name="creatingcontexts"></a> Creating contexts

Behat helps you when it comes to context creation. Lets say, for 
example you want to:

```gherkin
When I go to "John" his home
```

It's a verry simple step, we want to go to someone's place.

Now, instead of trying to figure out how we should implement this, 
lets run our test.

```bash
./behat features/home.feature

Output:
...
Undefined step "When I go to "John" his home"
...
FeatureContext has missing steps. Define them with these snippets:

/**
 * @When I go to :arg1 his home
 */
public function iGoToHisHome($arg1)
{
	throw new PendingException();
}
...

```

What just happend? Well, Behat tried to find the step definition 
(method) for "When I go to :arg1 his home".

Because it could not find it, it is kind enough to tell us how we 
can define it.

Within the method, you can do whatever you want with the values you 
receive from the test. $arg1 will contain, using our example, the 
string John.

Let's now give some more functionallity to the method:

```php
/**
 * @When I go to :name his home
 */
public function iGoToHisHome($name)
{
	// $this->getSession() contains the current scenario session.
	$this->getSession()->visit('/home/' . $name);
}
```

So, we simply redirect the user to "/home/John". We can do alot more 
with this.

For more examples you can take a look at 
**DigitalTransformationContext.php** inside the /tests/src/context 
folder. This is also our main context file.

**And should ideally be the only to contain custom step 
definition.**

> All other methodes, should be placed into helper classes. Good 
example is:
**FileContextHelper.php**

## <a name="usefulltips"></a> Usefull tips

### <a name="tags"></a> Tags

Within our project, we use tags to differentiate when what test 
should run.

Currently we have the following tags:

**@api** tag is used to define scenarios that use the Drupal api.
So that is basically every step of ours.

**@information, @political, @brp, @cwt**, are tags for each 
project.
A test that should only run for political should be tagged with 
@political.

**@shared**, these are tests that should run for information, 
and political.

**@wip**, this can be used to define a test that is not finished. 
But why would we ever push this?

A tag can be defined on feature and on scenario level.

Tags are defined in the behat.yml file:

```json
filters:
  tags: "~@wip&&@information,@shared"
```

Meaning: Not **@wip** AND (**@information** OR **@shared**)

### <a name="debugging"></a> Debugging

#### <a name="xdebug"></a> Xdebug / Phpstorm

When it comes to debugging Behat, I suggest you first of all to read
[Using Behat with phpstorm](http://blog.jetbrains.com/phpstorm/2014/07/using-behat-in-phpstorm/)

The initial setup might be a little pain (feel free to ask for help)
but once it it setup, you can use Behat and Xdebug to run and debug
tests.

This allows you to develop and test without manually doing the same
thing over and over again. Whenever you changed code, you can rerun
the test and see if the result is ok. Huge timesaver.

#### <a name="viewingpages"></a> Viewing pages

It might also be usefull at some point, to view the page before a 
the step fails. It can help you locate problems in your code or in
your test.

This setup is depending on your os. 

First of all you need to locate the command that can open a browser 
from the terminal.

osx default: `open http://google.be`
linux default: `google-chrome http://google.be` or chromium, 
firefox

Once we know the command to open the browser, we open up the 
**behat.yml**.

Add *show_cmd: open %s* as in the example:

```json
Behat\MinkExtension:
    goutte: ~
    selenium2: ~
    javascript_session: 'selenium2
    base-url: 'http://information.ecloc'
    show_cmd:  open %s
```

Now that we have configured Behat with the correct command to open
the correct browser, we can use the following in our behat to show
the page in the browser.

```gherkin
Then show last response
```

The browser will open, and show the full page including it's markup
making it easy to find potiential issues.

## <a name="whattotest"></a> What to test?

By importance:

1. Custom implementation
2. Bugfixes / regressions (feature exports gone wrong)
3. Critical custom configurations (editorial workflow, permissions, multilingual, views)
4. Business critical functionality (Tests by requests of the client)
5. ...

What not to test:
Basic Drupal delivered functions (content forms, admin menu, ...)

[cucumber]: https://en.wikipedia.org/wiki/Cucumber_(software)
