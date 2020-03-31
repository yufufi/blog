---
layout: post
title: Migrating to dotnet core 3.1 from 2.2
author: yufufi
categories: [programming]
tags: [dotnet, core, aspnet, programming, migration]
---

I had to migrate bunch of `aspnet core` projects from 2.2 to 3.1 recently. I managed to automate a small portion of the process to get things rolling:

``` fish
fd Dockerfile | xargs sed -i '' "s#sdk:2.2-alpine3.10#sdk:3.1.101-alpine3.10#g"
fd Dockerfile | xargs sed -i '' "s#aspnet:2.2.7-alpine3.10#aspnet:3.1.1-alpine3.10#g"
fd csproj | xargs sed -i '' "/Microsoft.AspNetCore.App/d"
fd csproj | xargs sed -i '' "/Microsoft.AspNetCore.All/d"
rg -l TargetFramework | xargs sed -i '' "s#<TargetFramework>netcoreapp2.2</TargetFramework>#<TargetFramework>netcoreapp3.1</TargetFramework>#g"
```

Here's what they do in order:
* Replace `sdk` image in all `Dockerfile` with the 3.1 image.
* Replace runtime `aspnet` image in all `Dockerfile` with 3.1 image.
* Remove all instances of `Microsoft.AspNetCore.App` from project files.
* Remove all instances of `Microsoft.AspNetCore.All` from project files.
* Replace `netcoreapp2.2` as target framework with `netcoreapp3.1`.

Aftewards I started kicking builds and fixing issues one by one. For a complete list of breaking changes from 2.2 to 3.1 see [here](https://docs.microsoft.com/en-us/dotnet/core/compatibility/2.2-3.1). In my case it entailed these:

* `IHostingEnvironment` under `Microsoft.AspNetCore.Hosting` is now obsolete and is replaced by `IHostEnvironment` under `Microsoft.Extensions.Hosting`. `IHostingEnvironment` under `Micorosft.Extensions.Hosting` is also obsolete and is replaced by `IWebHostEnvironment`  under `Microsoft.AspNetCore.Hosting` and drives from the new `IHostEnvironment`. There is a good blog post explaining the change in detail with some history [here](https://andrewlock.net/ihostingenvironment-vs-ihost-environment-obsolete-types-in-net-core-3/).

* Endpoint Routing is on by default. This change moves the responsibility of endpoint routing from `MVC` middleware to a dedicated middleware. This basically allows other middlewares to have access to routing information as it will be stored in `HttpContext`. For more info on the topic see [here](https://aregcode.com/blog/2019/dotnetcore-understanding-aspnet-endpoint-routing/). If you want to disable this functionality, you can do simply by:
``` c#
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc(options =>
    {
        options.EnableEndpointRouting = false;
    });
}
```

* `Json.NET` is not included in `aspnet core 3.1` by default anymore. If you want to keep using it you can add the package to your project via:

    `dotnet add <your_project_csproj_file> package Microsoft.AspNetCore.Mvc.NewtonsoftJson`

    And to configure it:  
``` c#
services.AddNewtonsoftJson(opt =>
{
    opt.SerializerSettings.ContractResolver = null;
});
```

* `EnableRewind` is replaced with `EnableBuffering`. This is the method to call if you want multiple middlewares to read request body.

That's all. Good luck!
