---
layout: post
title: "JSON as a programming language!"
subtitle: 'A thought expirement to explore JSON for nefarious means'
author: "clearnote01"
header-style: text
tags:
  - Json  
  - Programming
---

The basic idea is of adding some selected programming grammar in JSON objects, that would allow us to use JSON for more powerful configuration and describe complex situations without overly complicating the JSON used for configuration. 

I don't know if it's a good idea, and I thought writing about it would help me grasp if it's of any use. After I had defined a simple architecture in my mind, I went to look, if something like this exists, and I found (jsonnet)[https://jsonnet.org] which is not JSON but an altogether new language extending JSON. That does mean that it can be much more powerful, like adding error propogation, imports, arithmetics, etc. But my requirement was much simpler, and I think the solution is also fairly simple as you will see. 

I'll start with a simple function which also covers variables in this hypothetical JSON "programming"

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

So this json object is a function, in fact every json object is a function in this hypothetical language.
