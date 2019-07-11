---
title:  "Extracting Identity When Using Multiple Authentication Methods in AppSync"
date:   2019-07-11 06:01:00 -0500
categories: AWS AppSync VTL PipelineResolvers Cognito
---

Back in May, Amazon announced support for [multiple authentication methods in AppSync](https://aws.amazon.com/blogs/mobile/using-multiple-authorization-types-with-aws-appsync-graphql-apis/). This was a pretty huge deal because it really opens up a variety of ways to access an AppSync API using an API Key for anonymous user and [Cognito](https://aws.amazon.com/cognito/) when you need users to to authenticate first.

I have found this support led me almost immediately to the challenge of supporting identity information from multiple methods in your [AppSync Resolvers](https://docs.aws.amazon.com/appsync/latest/devguide/configuring-resolvers.html). You can get some metadata about the authenticated user in [$ctx.identity](https://docs.aws.amazon.com/appsync/latest/devguide/resolver-context-reference.html#aws-appsync-resolver-context-reference-identity), but when using API_KEY in particular, $ctx.identity field is not populated at all.

I resolved this by adding the following in my Before template for all [Pipeline Resolvers](https://docs.aws.amazon.com/appsync/latest/devguide/pipeline-resolvers.html).

~~~ vtl
#if( !$util.isNullOrEmpty($ctx.identity.sub) )
  $util.qr($ctx.stash.put("AWS_CURRENT_USER", "${ctx.identity.sub}"))
  $util.qr($ctx.stash.put("AWS_AUTH_TYPE", "AMAZON_COGNITO_USER_POOLS"))
#else
  #foreach($header in $ctx.request.headers.entrySet())
    #if($header.key == "x-api-key")
      $util.qr($ctx.stash.put("AWS_API_KEY", $header.value))
    #end
  #end  
  $util.qr($ctx.stash.put("AWS_AUTH_TYPE", "API_KEY"))
#end

{}
~~~

This allows you to do things like the following in Function Resolvers.

~~~ vtl

#if($ctx.stash.AWS_AUTH_TYPE == "AMAZON_COGNITO_USER_POOLS")
  ...do stuff with $ctx.stash.AWS_CURRENT_USER here...
#end 

#if($ctx.stash.AWS_AUTH_TYPE == "API_KEY")
  ...do stuff with $ctx.stash.AWS_AUTH_TYPE here...
#end 

~~~