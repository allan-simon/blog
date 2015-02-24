+++
date = "2015-02-25T00:08:11+08:00"
draft = true
title = "Redmine-cli and redmine API"

+++


I found a tool to manage redmine from the cli, in python: 

`pip install redmine-cli`

then a config file `~/.redmine-cli`

```
[default]
key=API_KEY_YOU_CAN_SEE_IN_YOUR_ACCOUNT_ON_REDMINE
my_id=ID_YOU_CAN_ON_YOUR_PROFILE_PAGE
root_url=HOME_PAGE_OF_REDMINE
```

then you can do 

  1. `redmine open 123` to open the issue 123 in your browser
  2. `redmine issues` to see your issues
  3. `redmine issue 123` to see information about issue 123

(it has more options I let you check the doc)

  * [original repo](https://github.com/yanjost/redmine-cli)
  * [my fork where I have some features (like updating a ticket](https://github.com/allan-simon/redmine-cli)

(hopefully my modification are in PR on the original repo and will be soon integrated)

### The long story behind:

As any good Linux user, I love the CLI (no, there's no such thing 
as a **good** Linux user who does not love the CLI). Starting from
that, though I appreciate using Redmine to manage features to implement,
bugs etc. I'm not really fan of needing to click here and there,
especially when the only things I want is

  * know what a Merge request on gitlab referencing the ticket XXX is about
  * see what tickets are assigned to me
  * update a ticket status

It becomes even less practical when you want to update a list of ticket
you can either:

  1. select in the list, while keeping shift pressed, the list of ticket you want to update
  2. right click,
  3. update the status

The problem of that technique is that, if you click a little too much on the right,
on the left etc. you loose your selection, which can be very frustating
Then you have the second technique

  1. click on ticket N
  2. click on update
  3. update, click on submit

each of these steps impliying a page load, no need to tell you how much times
it takes even for 5 tickets

### The CLI will always be there for you: redmine-cli

For now some month, whenever I'm looking for that little tool, the reflex is to
search for it on github. As I knew there was a Redmine API and that we don't have
time for a NIH (Not Invented Here) syndrom, it was very likely such a CLI tool
would already exist.

I found a several tool, [one in ruby](https://github.com/diasjorge/redmine-cli)
which seems rather complete, but as it was not updated for 2 years, and I didn't felt
like installing ruby on my computer (I guess it's already install but...) and it was
late, so losing 2 minutes to know how to install the project was too much

  * ALWAYS PUT CLEAR INSTRUCTION , DUMB PROOF, ON HOW TO INSTALL YOUR TOOL
  * if a project is "complete" and its the reason why you're not updating it, state it
so that people know if the project is dead or just 'in good shape enough but the author is still active'

So I found the python project, which was updated in 2014 and
had clear installation instruction `pip install redmine-cli`

### Missing feature: redmine API my love

while implementing the feature I found that 

  * the API call was returning a 200 (without nothing in the body)
  * BUT the status was not updated (though the ticket was showing activity 'less than a minute ago')

after some research I found [that redmine discussion](http://www.redmine.org/boards/2/topics/25920)

  * I need to declare the header `Application/Json` (normal)
  * It needs to have the `status_id` value as a string (and I was giving an integer)

The second point is raging because GUESS WHAT, that integer was given to me BY REDMINE ITSELF !
So please please when you do an API
 
  * Be consistent, if `status_id` is an integer once, it must always be a integer
  * Have a clear documentation, with example


