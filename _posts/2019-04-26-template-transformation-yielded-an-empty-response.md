---
title:  "AppSync Error: Template transformation yielded an empty response"
date:   2019-04-19 08:15:19 -0500
categories: AWS AppSync VTL
---

If you see this error, make sure that your AppSync VTL resolver is returning something. If it doesn't need to return anything, just return an empty object.

For example, suppose you are doing some sort of pre-processing in a pipeline resolver, like this:

~~~ vtl
## build person
#set($person = {})
$util.qr($person.put("id",              $util.autoId()))
$util.qr($person.put("firstName",       $ctx.args.input.firstName) )
$util.qr($person.put("lastName",        $ctx.args.input.lastName) )

## stash person
$util.qr($ctx.stash.put("person",   $person) )
~~~

Makes sure you drop a little empty object in there so that something gets returned. Like this:

~~~ vtl
## build person
#set($person = {})
$util.qr($person.put("id",              $util.autoId()))
$util.qr($person.put("firstName",       $ctx.args.input.firstName) )
$util.qr($person.put("lastName",        $ctx.args.input.lastName) )

## stash person
$util.qr($ctx.stash.put("person",   $person) )

{} ## make sure you add this at the end
~~~

