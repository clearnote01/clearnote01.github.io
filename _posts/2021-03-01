---
layout: post
title: "JSON as a programming language!"
subtitle: 'A thought experiment to explore JSON for nefarious means'
author: "clearnote01"
header-style: text
tags:
  - Json  
  - Programming
---

The basic idea is of adding few selected programming grammar in JSON objects, that would allow us to use JSON for more powerful configuration and describe complex situations without overly complicating the JSON used for configuration. 

I don't know if it's a good idea, and I thought writing about it would help me in evaluating it. After I had defined a simple architecture in my mind, I went to look, if something like this exists, and I found [jsonnet](https://jsonnet.org) which is not JSON but an altogether new language extending JSON. That does mean that it can be much more powerful, like adding error propogation, imports, arithmetics, etc. But my requirement was much simpler, and I think the solution is also fairly simple as you will see. 

I'll start with a simple function which also covers variables in this hypothetical JSON "programming". Now to make it easier for me I'll freely call it jsonconf and i'll also refer to a hypothetical `parser` as whatever is going to read this configuration and implement the core functions.


```json
{
    "fn": "@equals: arg1, arg2 => bool",
    "arg1": {
      "fn": "@jsonpath: json, path => str",
      "json": {
        "fn": "@globals: name => str",
        "name": "g1"
      },
      "path": "body.response.field1"
    },
    "arg2": "value1"
}
```


In short the `code` we wrote, it takes a jsonpath, a jsonstring and an expected value, if the value at jsonpath in jsonstring matches the expected value it returns true else false.
The jsonstring is not actually a variable to function but a global variable (we will see this more later on).

So let's look at this one by one:
```json
    "fn": "@equals: arg1, arg2 => bool"
```

The key idea with jsonconf is that every json object represents a function call (repeat this in your head a few times)

We use `fn` that tells us what built-in method to call and defines the signature of this block. `@equals` is a built-in method, all the methods start with @ in their names. This is followed by the argument name, this is useful as we can give any arbitrary name in this function block, and then this is followed by `=> bool` which is the type of the return value. Now you may find this a bit weird, why does the return type have data type and not the function arguments, this is nonsense! To that I'll say hold on a minute, if you follow on with the logic that every object is a function call and every function defines the return type with `=>` syntax then it follows that we can infer the data types very quickly with a quick look on the functions call of the arguments (and so can the parser!)

So, yes all objects are functions calls but you can also have fields where value is a json primitive (like string, integer, bool), since this is super common, you can always use this and again inferring type for these is not difficult. However you get another built-in method if you wanted more complex data types, that function is `@literal`. And I hope this helps in seeing how useful the `=>` can be for working with data types.

```json
{
	"fn": "@literal: val => date",
	"val": "2020-04-25"
}
```

Okay, now moving on you will see `arg1` and `arg2` defined in the following fields:

```json
{
    "arg1": {
      "fn": "@jsonpath: json, path => str",
      "json": {
        "fn": "@globals: name => str",
        "name": "g1"
      },
      "path": "body.response.field1"
    },
    "arg2": "value1"
}
```

So starting with `arg1`, this uses another built-in function `@jsonpath` which takes two arguments, `json` and `path` and returns a `str`, and then if you look inside it's another function call which uses 
the `@globals` method that returns a globally defined variables (always look these configs from bottom up). The global variables will be set through some exposed API on the parser, if you have a json string on which you wanted to work on, then go ahead and add it as a global named variable in the parser API so the jsonconf can refer to it without duplication.

```js
let jsonStr = "{ \"header\": { \"type\": \"test\" }, \"body\": { \"response\": { \"field1\": \"actual_field1_value\" } } }";
jsonconf.setGlobal("g1", jsonStr);
```
this can be referred in `jsonconf` with:

```json
{
  "fn": "@globals: name => str",
  "name": "g1"
}
```

Moving on, now we have value in `json` which is a string, `path` is a json primitive and `@jsonpath` is called now with these arguments. This returns the value of field in the json at that path, again
the type specified in the signature would be used for casting. By now it should be hopefully clear what will happen on the uppermost layer.

```json
{      
	"fn": "@equals: arg1, arg2 => bool",
	"arg1": { },
	"arg2": "value1"
}
```

arg1 is now populated with the value at the path `body.response.field` which is `actual_field1_value`, this along with arg2 `value1` will be passed as arguments to `@equals` which returns `false` as they do not match.

Okay now, following this logic, let's think of more complex constructs! like errors!, a program should have ways to panic, and then I want to see how we combine errors; conditionals (as we saw above). Here is a function call which returns err based on a predicate. We use the `@error` construct here:

```json
{
	"error_call": {
		"fn": "@error: predicate, err_msg, err_type => null|err",
		"err_msg": "found an error!",
		"err_type": "generic",
		"predicate": {
			"fn": "@equals: a,b => bool",
			"a": "hello",
			"b": "hello"
		}
	}
}```


This functioncall will return an exception as value of `a` and `b` match, and we get an error stored in `error_call`. Now someone needs to make sense of this return value, and yes, that will be another function! and this is the most useful piece of this puzzle, that makes sense of conditionals we saw before and then error calls we saw just now. All this is glued by ``@ok` and its variants. Let's see it one by one. 

```json
{
	"result": {
		"fn": "@ok data, error_call => str|err",
		"error_call": "{..}",
		"data": "hello world"
	}
}```
As you can see here `@ok` takes 2 arguments, first argument can be of any type and any value and second argument is of error type, if there was any error the value of `error_call` would not be null, in that case `error_call` will be returned otherwise `data` will get returned. the return is either the `data` or `error_call` based on if `error_call` is null or not. 

There is also `@all` and `@any` which are variadic functions and return bool based on input, since error handling is really important it can handle it automatically, if one of the argument is an error_call if the value is `null` that is ignored, otherwise it returns the `err` type, other than errors, `@all` and `@any` do the same thing as you would expect in any other language (all -> true if all arguments are true, any -> if any one argument is true). 

```json 
{
	"result": {
		"fn": "@all arg1, arg2, error_call1, arg3, error_call2 => bool|err",
		"arg1": true,
		"arg2": true,
		"error_call1": " { .. fn: @error ..}",
		"arg3": true,
		"error_call2": " { .. fn: @error ..}"
	}
}```

Now all this is, I think it's a bit interesting to think like that, but one thing we need to consider is performance, there is no way a dynamic piece of string being parsed at runtime can be performant,
either it needs a statical code generation parser, or the parser needs to be very efficient that the overhead at runtime is minimal. I also got interested in a subset of this configuration, like I know function argument aliasing is nice what if we wrote the above block as:

```json 
{
	"result": {
		"fn": "@all: bool|err",
		"arg1": true,
		"arg2": true,
		"error_call1": " { .. fn: @error ..}",
		"arg3": true,
		"error_call2": " { .. fn: @error ..}"
	}
}```

Now as I write it like this it looks more and more appealing to me, this simplifies the job of parser a "lot". And I already repeated an object here represents a functioncall, so not needing to represent argument in signature is not a wild idea. We can go even further and represent `return` type as well and that is again an improvement in some sense.

```json 
{
	"result": {
		"fn": "@all",
		"arg1": true,
		"arg2": true,
		"return": ["err", "bool"]
	}
}```

Let's re-write the first bit of example in this manner,
```json
{
    "fn": "@equals",
    "arg1": {
      "fn": "@jsonpath",
      "json": {
        "fn": "@globals",
        "name": "g1",
	"return": "str"
      },
      "path": "body.response.field1",
      "return": "str"
    },
    "arg2": "value1"
    "return": "bool"
}
```

This still looks a bit off, how do exceptions come into picture here... like `@jsonpath` should return an `err` right if json string doesn't have the value at the given path. To that I say, I don't know, need to think more I suppose.

Some nice to have features:
- Partial functions, and a namespace for storing functions in an object and then some way to referring those partial functions.
