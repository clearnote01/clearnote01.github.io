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

The basic idea is of adding some selected programming grammar in JSON objects, that would allow us to use JSON for more powerful configuration and describe complex situations without overly complicating the JSON used for configuration. 

I don't know if it's a good idea, and I thought writing about it would help me grasp if it's of any use. After I had defined a simple architecture in my mind, I went to look, if something like this exists, and I found (jsonnet)[https://jsonnet.org] which is not JSON but an altogether new language extending JSON. That does mean that it can be much more powerful, like adding error propogation, imports, arithmetics, etc. But my requirement was much simpler, and I think the solution is also fairly simple as you will see. 

I'll start with a simple function which also covers variables in this hypothetical JSON "programming". Now to make it easier for me I'll freely call it jsonconf and i'll also refer to a hypothetical `parser` as whatever is going to read this configuration and implement the core functions.


```json
{
    "globals": {
      "g1": "{
        \"header\": {
          \"type\": \"test\"
        },
        \"body\": {
          \"response\": {
            \"field1\": \"actual_field1_value\"
          }
        }
      }"
    },
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


In short the `code` we wrote takes a jsonpath, a jsonstring and an expected value, if the value at jsonpath in jsonstring matches the expected value it returns true else false.
The jsonstring is not actually a variable to function but a global variable, this can be defined as a `globals` field as we have done but more likely through some exposed APIs in the parser. But once a global variable has been set it can be referred by the `@globals` function (we will see this more later)

So let's look at this one by one:
```json
    "fn": "@equals: arg1, arg2 => bool",
```

The key idea with jsonconf is that every json object represents a function call (repeat in your head a few times)

We use `fn` that tells us what function to built-in method to call and defines the signature of this block. `@equals` is a built-in method, all the methods start with @ in their names. This is followed by the argument name, this is useful as we can give any arbitrary name in this function block, and then this is followed by `=> bool` which is the type of the return value. Now you may find this a bit weird, why does the return type have data type and not the function arguments, this is nonsense! To that I'll say hold on a minute, if you follow on with the logic that every object is a function call and every function defines the return type with `=>` syntax then it follows that we can infer the data types very quickly with a quick look on the arguments of the function (and so can the parser!)

So, yes all objects are functions calls but you can also have fields where value is a json primitive (like string, integer, bool), since this is super common, you can always use this and again inferring type for these is not difficult. However you get another built-in method if you wanted more complex data types, that function is `@literal`. And I hope this helps in seeing how useful the `=>` can be for working with data types.

```json
{
	"fn": "@literal val => date",
	"val": "2020-04-25"
}
```

Okay, now moving on you will see `arg1` and `arg2` defined in the following fields:

```json
    "arg1": {
      "fn": "@jsonpath: json, path => str",
      "json": {
        "fn": "@globals: name => str",
        "name": "g1"
      },
      "path": "body.response.field1"
    },
    "arg2": "value1"
```

So starting with `arg1`, this uses another built-in function `@jsonpath` which takes two arguments, `json` and `path` and returns a `str`, and then if you look inside it's another function call which uses 
the `@globals` method that returns a globally defined variables (always look these configs from bottom up). Practically we wouldn't have g1 already encodeded into the jsonconfig as in this example but actually set by some exposed API on the parser, so you have a json string on which you wanted to work on, then go ahead and add it as a global named variable in the parser API so the jsonconf can refer to it without any duplication.

```js
let jsonStr = "{ \"header\": { \"type\": \"test\" }, \"body\": { \"response\": { \"field1\": \"actual_field1_value\" } } }";
jsonconf.setGlobal("g1", jsonStr);
```

Moving on, now we have value in `json` which is a string, `path` is a json primitive and `@jsonpath` is called now with these arguments. This returns the value of field in the json at that path, again
the type specified in the signature would be used for casting. By now it should be hopefully clear what will on the uppermost layer.

```json
{   ...
	"fn": "@equals: arg1, arg2 => bool",
	"arg1": { .. },
	"arg2": value1"
}
```

arg1 is now populated with the value at the path `body.response.field` which is `actual_field1_value`, this along with arg1 `value1` will be passed as arguments to `@equals` which returns `false` as they do not match.



TODO: `@match_any|all`, `err` as a date type.
