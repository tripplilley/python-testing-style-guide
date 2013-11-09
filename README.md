python-testing-style-guide
==========================

This document is a draft style guide for writing Python tests (and Django tests). It is meant to be
expanded and refined over time.

Many of the recommendations stand in stark contrast to conventions for good code. This is
deliberate: tests are not regular code and have a much different set of risks and rewards.


# PEP8 first

Start with [PEP8](http://www.python.org/dev/peps/pep-0008/) and strive to understand why it makes
the recommendations it does. We like nearly all of it except for the Maximum Line Length recommendation. 

# Lint in your editor

Configure the Python linting plugin for your editor and use it every time without exception.

# Run tests often

We run our tests continuously with Jenkins (every morning and after every commit), but you still
need to run them locally before pushing code.

# Follow import conventions

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

# Tox

# Write to be read

# Use TDD if it makes sense

# Always write a docstring

# Reproduce before testing

# Prefer small tests

# Fail tests first

# Never `print`

Use `logging` instead of `print` in your code.
Learn how to make the logger work for you rather than against you.
This should appear in every one of your files:

    import logging

    log = logging.getLogger(__name__)

# Repeat yourself

# Avoid fixtures

# Enjoy assertRaises

# Pay attention to coverage

# Use CSS selectors

# Prefer extremely targeted functional tests

# Use `t-` CSS classes

# Prefer descriptive variables

# Prefer fewer asserts per test

# Always bind an `expected` value

# Assert the `expected` value then the returned value

# Use mocks with care

# Resources

* A gentle introduction to the _feel_ of testing (Django focus): [Test-Driven Web Development with Python](http://www.safariflow.com/library/view/Test-Driven+Web+Development+with+Python/9781449365141/)
* The classic description of TDD (Java examples, oh well): [Test Driven Development: By Example](http://www.safariflow.com/library/view/Test+Driven+Development%3A+By+Example/0321146530/)
* Another classic focused on explaining how testing works in the real world (more Java examples, oh well): [Test Driven Development: By Example](http://my.safaribooksonline.com/book/software-engineering-and-development/software-testing/0131016490)
* Recent take on testing lessons from a mature project: [Testing Django Projects at Scale](http://pyvideo.org/video/2333/testing-django-projects-at-scale), a PyCon CA 2013 video 
* Thoughts on mocking versus faking: [Stop mocking, start testing](http://nedbatchelder.com/blog/201206/tldw_stop_mocking_start_testing.html), a writeup with links to a PyCon 2012 video
* A presentation to convince you to start testing: [Getting Started Testing your Python](http://nedbatchelder.com/text/st.html#1)
