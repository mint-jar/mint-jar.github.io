---
title: "Docker Multi-stage build" 
date: 2021-08-11T20:01:08+02:00
tags: ["today-i-learned", "docker"]
draft: false
---

In our latest project I encountered some new quirks within Docker while using multi stage builds for the first time.


 <!--more-->

At first glance, it looks really simple:

```docker
FROM alpine:latest AS base
WORKDIR /test
RUN touch base

FROM base AS develop
RUN touch develop

FROM base AS production
RUN touch production
```

The idea behind this is that you can target which stage you want and get the correct result.

So when I run the following command with this example:

```sh
docker build --target production -t mulibuild/example:latest . 
```

And check the touched files

```sh
docker run -it mulibuild/example:latest find . -maxdepth 1 -type f
```

I will see the following:

```sh
./production
./base
```

As you can see, we only have the result of the base and production stage. Everything is working as expected. 

But what if I introduce a command that fails?

```docker
FROM alpine:latest AS base
WORKDIR /test
RUN touch base

FROM base AS develop
RUN break develop # <-- Will break now!

FROM base AS production
RUN touch production
```

When I run the same build command, as before it will fail now. Strange right?

This is due to the way docker builds from a docker file. Even if you create a multistage file it will run all stages until the target and combine the results of every build target as described. So in our example:

1. base, develop and production stages are build
2. base and production are combined

If I move the broken stage to the end it will work.

```docker
FROM alpine:latest AS base
WORKDIR /test
RUN touch base

FROM base AS production
RUN touch production

FROM base AS develop
RUN break develop
```

When running the build command, it will no longer fail as the build will stop once it reached its target.

1. base and production are build
1. develop is skipped as the target has been reached before

