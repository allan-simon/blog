+++
date = "2015-03-11T22:40:50+08:00"
draft = true
title = "symfony 2.6 tips #1"
+++

Here a list of tips for symfony 2.6 I've gathered these days

###Skip test in phpunit

Imagine you're rewriting your application to split 'big' pages into
set of smaller actions. You got your test already written but they
are going to fail the time you finish the rewrite, and because you're
in company with tight deadline, you don't have time to adapt the tests
"right here right now" several possibilities 

  * Leave the tests failing until you fix them `master`.
  * Delete the test.
  * Put between comments the failing tests.

All these solutions are suboptimals, instead you can add this line at the beginning of your test method

```
public function testMyTestWhichYouWantToSkip()
{
    $this->markTestSkipped("message explaining why it's skipped");
    // your test code, unmodified
}
```

so that it will appear with a `S` in the test report, so that you have a reminder that there's something to fix.


### Disable form validation in test

You're writing functionnal test and you need to check the case of an form submitted with bogus data (to be sure your server side code is protected against hand-crafted request). By default the form you get in functionnal test will have the same checking rules than the one on server side, so in order to disable checking you can do 

```
$form['country']->disableValidation()->select('Invalid value');
```

### (Doctrine) don't use findById() when you just mean find() 

well, the title is self-explaining
