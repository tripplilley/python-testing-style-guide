python-testing-style-guide
==========================

This document is a draft style guide for writing Python tests (and Django tests). It is meant to be
expanded and refined over time.

Many of the recommendations stand in stark contrast to conventions for good code. This is
deliberate: tests are not regular code and have a much different set of risks
and rewards. 

If you're curious _why_ a particular practice is recommended, get
someone to defend or explain it (there's a change it's wrong and needs to be
updated). That said, a lot of these recommendations come from specific mistakes
and scars earned over many years of work, so don't be surprised if they're
passionately defended.

# Prerequisites 

## PEP8 first

Start with [PEP8](http://www.python.org/dev/peps/pep-0008/) and strive to understand why it makes
the recommendations it does. We like nearly all of it except for the Maximum Line Length recommendation. 

## Lint in your editor

Configure the Python linting plugin for your editor and use it every time without exception.

## Prefer imperfect tests to no tests

Tests should (almost) always accompany code we write professionally. We're
fortunate to work in an environment where testing is actively encouraged, so
don't waste that priveledge. We get away with all sorts of other efficiencies
and laziness because of our sincere dedication to pragmatic and practical
testing.

# The practice of testing

## Tox

Every package should have setup instructions for a developer in the `README`
and should be testable using `tox` after setup. Use 100% test success to
confirm your setup is complete.

## Master should be 100%

Strive to never push a failing test to master. If you want to push failing tests to another branch, that's fine (in fact often encouraged).

## Run tests often

We run our tests continuously with Jenkins (every morning and after every commit), but you still
need to run them locally before pushing code.

## Use TDD if it makes sense

Some people do TDD often, sometimes, or rarely. Figure out when it's an
effective habit for you.

## Pay attention to coverage

## Remember the QA team

We are fortunate in that we have a smart and effective QA team. Some functionality simply cannot be
effectively tested, which can feel frustrating to a developer. Before spending too much time on a
set of tests of limited benefit, consider whether the issue should be 

## Delete dead or misleading tests aggressively

Do not be shy about removing tests as soon as they start to become false or unhelpful.

## Focus on tests during code reviews


## Reproduce before testing

For regressions, you should always make sure you can reproduce the issue in a
browser as a typical user before writing a single line of code (test or
otherwise). After you can reproduce it, try to get a failing test to reliably
reproduce it (if possible). Only then is it a good time to start writing the
fix.

## Always "Click the thing"

No developer should ever rely on tests alone. Before resolving an issue, make
sure you have walked through it at least once in a browser as a typical user.

## Write regression tests whenever possible

## Fail tests first

How do you know if it is actually testing anything if the `assert` never failed?

## Write descriptive fail messages

When tests fail they should tell you exactly why. For example, this:

    response = self.client.post('/api/v1/data/', data=SAMPLE)
    assert response.status_code == status.HTTP_201_CREATED

...will fail with this message when the status code is correct:

    >           assert response.status_code == status.HTTP_201_CREATED
    E           AssertionError: assert 201 == 400
    E            +  where 400 = <rest_framework.response.Response object at 0x1070bbb10>.status_code
    E            +  and   201 = status.HTTP_201_CREATED

However, when the code includes a custom error message:

    response = self.client.post('/api/v1/data/', data=SAMPLE)
    assert response.status_code == status.HTTP_201_CREATED, "expected HTTP_201, got HTTP_{} data: {}".format(response.status_code, response.data)

...the reason for the failure is much clearer:

    >           assert response.status_code == status.HTTP_201_CREATED, "expected HTTP_201, got HTTP_{} data: {}".format(response.status_code, response.data)
    E           AssertionError: expected HTTP_201, got HTTP_400 data: {'name': [u'This field is required.']}
    E           assert 400 == 201
    E            +  where 400 = <rest_framework.response.Response object at 0x1070bbb10>.status_code
    E            +  and   201 = status.HTTP_201_CREATED

## Prefer fewer asserts per test

# Structure

## Prefer small tests

## Separate tests into different files

Tests are generally structured to mirror the file layout of the modules they
are testing. It is OK to group tests for small modules or to separate targeted
tests for a single module across many files.

## Follow import conventions

Imports should be grouped in the following order (extends PEP8 rules):

1. Python standard library imports
1. Public library imports
1. Django imports
1. Testing library imports
1. Internal libary imports
1. Local application imports

Each import line must be in alphabetical order inside its group.

    import datetime
    import inspect
    import logging
    import uuid
    import os


    from lxml import etree
    from lxml.html import document_fromstring, html5parser


    from django.contrib.auth import get_user_model
    from django.core.urlresolvers import reverse
    from django.test import TestCase
    from django.test.client import Client
    from django.views.defaults import server_error


    import mock

    from nose import SkipTest
    from nose.tools import assert_equals, assert_in


    import nest.models 

    from nest.tests.test_helper import create_document


    from entice.models import UserSubject
    from heron.decorators import anyone_allowed
    from heron.models import Trial
    from heron.urls import urlpatterns
    from tools.urls import traverse_urls


# Write to be read

Our tests are often the first thing other developers read to try to understand
our code. They'll also often be the first thing we read when something is on
fire and we need to fix a bug. Strive for readability in tests.

## Always write a docstring

Every single test should have a human-readable docstring. The docstring should
follow the pattern of the rest of the module.

Don't end docstrings with a period

## Use "should" in every docstring

Tests are often best when they focus on what behavior "should" happen in terms
of important actors in the system. A good mental trick to keep tests in this
style is to use "should" in every docstring. 

In addition, try to use a known actor or object as the subject of the docstring.

Keep the number of words before should (or "should not") to a minimum to improve clarity.

        '''The Manage Users page should show a button to revoke access to the site for each User'''

        '''The Manage Users page should show Users with obscured emails if they are associated with a demo Account'''

        '''A Tutorial should be able to exist in multiple Groups'''

        '''A European User should not be required to enter a state or province'''

## Prefer descriptive variables

A reader may have no idea what is happening inside your test. Help them by binding to descriptive variables.

Yes:

    reset_url = reverse('django.contrib.auth.views.password_reset_confirm', kwargs={'uidb36': self.user.id, 'token': token})
    absolute_reset_url = "http://{}.{}{}'.format(self.account.subdomain, settings.BASE_SITE, reset_url)
    expected = "Set your personal password: {}".format(absolute_reset_url)
    self.assertOutboxContainsBody(expected)

No:

    self.assertOutboxContainsBody(
        'Set your personal password: http://{subdomain}.{base_site}{url}'.format(
            subdomain=self.account.subdomain, base_site=settings.BASE_SITE,
            url=reverse('django.contrib.auth.views.password_reset_confirm',
                        kwargs={'uidb36': self.user.id, 'token': expected})))


# Expectations

## Always bind an `expected` value

We always bind an `expected` value because then the reader has no doubt of what
is happening inside our asserts. Conventions like this allow us to focus on
what's _different_ about the code instead of being distracted by eccentricities
of each implementation.

Yes:

    expected = [12, 19]
    self.assertEquals(expected, results)

No:

    self.assertEquals(40, num_lessons)


## Assert the `expected` value then the returned value

Yes:

    self.assertEquals(expected, topic_toc_links)

    self.assertEquals(expected, output)

No:

    self.assertEquals(results, expected)

## Prefer explicit expected values

Unlike regular code, tests are often stronger when they have explicit `expected` values despite the
cost. This can improve readability ("What does this JSON response actually look like in practice?")
and ensure that tests don't accidentally test nothing.

Yes:

    expected = ["2001-09-01",
                "2002-12-30",
                "2003-08-20",
                "2003-08-29",
                ]
    self.assertEquals(expected, dates)

No:

    expected = [lesson.started for lesson in self.lessons] # Oops, self.lessons was empty and I just tested nothing
    self.assertEquals(expected, dates)

# Things that people will yell about

## Never `print`

Use `logging` instead of `print` in your code.
Learn how to make the logger work for you rather than against you.
This should appear in every one of your files:

    import logging

    log = logging.getLogger(__name__)

# Unusual testing conventions

## Repeat yourself

# Django 

## Avoid fixtures

## Use CSS selectors

More people can read and write CSS selectors than XPath or alternatives.

## Prefer extremely targeted functional tests

## Use `t-` CSS classes

Every single test that relies on specific HTML markup *must* use/add a CSS class starting with `t-` to the required HTML elements.

This declarative style allows the HTML and CSS to be refactored without impacting tests (even extremely specific ones).

    num_revoke_buttons = len(nodes.cssselect(".t-user-table .t-revoke"))

## Consider templatetags

Templatetags are often much easier to test (especially in isolation) than the equivalent functional
test.

# Testing tools and techniques

## Use mocks with care

## Enjoy assertRaises

# How to avoid traps

## Really think about boundary values

## Test the obvious positive and negative cases separately

## Introduce some randomness

We introduce randomness to our tests to make sure we're handling a range of inputs sanely.

The two most common patterns for introducing randomness are using `uuid.uuid4()` for string values
and `random.randint()`.

    self.epub = nest.models.EpubArchive.objects.create(identifier=str(uuid.uuid4()))

    bit = self.epub.htmlfile_set.create(filename="first.html", virtual_pages=random.randint(1, 100))

    how_many = random.randint(10, 100)
    for i in range(how_many):
        ...

Warning: Randomness means that some test failures will become hard to reproduce. This is usually an
acceptable cost.

## Introduce some Unicode

When setting up expected values or string inputs, try to remember to include strange Unicode
characters.

    expected = [u"R", 
                u"å", 
                u"n",
                u"d", 
                u"☃", 
                u"ɯ"]

# Resources

* A gentle introduction to the _feel_ of testing (Django focus): [Test-Driven Web Development with Python](http://www.safariflow.com/library/view/Test-Driven+Web+Development+with+Python/9781449365141/)
* The classic description of TDD (Java examples, oh well): [Test Driven Development: By Example](http://www.safariflow.com/library/view/Test+Driven+Development%3A+By+Example/0321146530/)
* Another classic focused on explaining how testing works in the real world (more Java examples, oh well): [Test Driven Development: By Example](http://my.safaribooksonline.com/book/software-engineering-and-development/software-testing/0131016490)
* Recent take on testing lessons from a mature project: [Testing Django Projects at Scale](http://pyvideo.org/video/2333/testing-django-projects-at-scale), a PyCon CA 2013 video 
* Thoughts on mocking versus faking: [Stop mocking, start testing](http://nedbatchelder.com/blog/201206/tldw_stop_mocking_start_testing.html), a writeup with links to a PyCon 2012 video
* A presentation to convince you to start testing: [Getting Started Testing your Python](http://nedbatchelder.com/text/st.html#1)
