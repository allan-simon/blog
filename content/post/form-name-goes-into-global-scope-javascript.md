+++
date = "2015-02-23T17:11:21+08:00"
draft = true
title = "Form name goes into global scope in javascript"

+++

An interesting problem a friend of mine discovered today

Can you spot the problem?

<!--more-->

### Le javascript 

```
var CreateVersion = (function(){
    'use strict';

    var foo = function(text) {
        console.log(text);
    };

    return {
        foo : foo 
    };
}());
```

### Le HTML

```
<form name="CreateVersion" method="post" action="">
    <div class="form-group">
        <label>
            <span>something</span>
            <!--
                onClick on a span is a bad pratice and will be
                the object of an other post
            -->
            <span onclick="CreateVersion.foo('plop')">plop/span>
            <span onclick="CreateVersion.foo('prout')">prout</span>
        </label>
        <input
            id="CreateVersion_name"
            name="CreateVersion[name]"
            class="form-control version_name"
            type="text"
        />
    </div>
    <div class="form-group">
        <label>
            <span>something something</span>:
            <span class="text_length">0</span>
        </label>
        <input
            id="CreateVersion_articleNotes"
            name="CreateVersion[articleNotes]"
            class="form-control"
            type="text"
        />
    </div>
    <hr/>
    <div>
        <button
            type="submit"
            id="CreateVersion_submit"
            name="CreateVersion[submit]"
        >
            submit
        </button>
    </div>
</form>
```

### Le erreur

```
TypeError: CreateVersion.foo is not a function
```

No, not in IE6, in last version of Firefox/Chrome 


### Le solution

This line 

```
    <form name="CreateVersion" method="post" action="">
```

declare a Form with a name `CreateVersion` which is exactly the same name as our
Javascript module, and guess what ?

The name is exported in the DOM, which that when you do that

```
<form name="Foo">
```

Then you can do 

```
document.Foo
```

to access to the DOM node, and in the HTML itself, the "document" is implicit which
mean that "document.Foo" collide with any javascript defined variable "Foo"

For more explanation see [this stackoverflow question](http://stackoverflow.com/questions/1415747/javascript-function-and-form-name-conflict)

### How to avoid that:

Some possibilities:

  * Make it a coding style policy that module name start contains something "javascript only" like `FooModule` or `FooJS`
  * put your modules in one big wrapping modules (like `MyCorp.Foo`)
