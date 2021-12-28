---
title: "Fixing ERROR: toomanyrequests: Too Many Requests. in your Gitlab Continuous Integration Jobs"
layout: post
subtitle: Fix Gitlab pipelines
description: "Fixing ERROR: toomanyrequests: Too Many Requests. in your Gitlab Continuous Integration Jobs"
image: /assets/img/uploads/toomanyrequests.jpg
date: '2021-01-07 07:28:05 +0900'
optimized_image: /assets/img/uploads/toomanyrequests.jpg
category: fundementals
tags:
  - ansible
  - linux
  - book
author: Maverick Stoklosa
---

You may have seen errors like these in your Gitlab CI

```
ERROR: toomanyrequests: Too Many Requests.
```

This is because starting November 2, 2020 Docker has introduced a 250 request per 6 hours rate limits for free accounts.

So now you have a team full of people waiting for their jobs to finish in Gitlab so they can deploy to production. How do we get out of this mess?

The cheapest way is to purchase a Pro account https://www.docker.com/pricing

![toomanyrequests2](/assets/img/uploads/toomanyrequests2.png))

Once you have your pro account you can go on your server and execute

```
docker login
```

After you entered your credentials it will generate a

```
~/.docker/config.json
```

file with something like this in it

```
{
        "auths": {
                "https://index.docker.io/v1/": {
                        "auth": "sHh6ARbxJGDwU4d2V"
                }
        },
        "HttpHeaders": {
                "User-Agent": "Docker-Client/18.06.1-ce (linux)"
        }
}
```

You will need the first `auths` part which you will transform into a DOCKER_AUTH_CONFIG variable to be used inside your

```
/etc/gitlab-runner/config.toml
```

file. That looks like this:

```
environment = ["DOCKER_AUTH_CONFIG={\"auths\":{\"index.docker.io/v1/\":{\"auth\":\"sHh6ARbxJGDwU4d2V\"}}}"]
```

Next you configure your gitlab-runners. For each runner append the

```
environment = ["DOCKER_AUTH_CONFIG={\"auths\":{\"index.docker.io/v1/\":{\"auth\":\"sHh6ARbxJGDwU4d2V\"}}}"]
```

in the `[[runners]]` section like so:

```
[[runners]]
  name = "gitlab-runner"
  url = "https://gitlab.company.com/ci"
  token = "JpmhpI09mr3MJin0f"
  environment = ["DOCKER_AUTH_CONFIG={\"auths\":{\"index.docker.io/v1/\":{\"auth\":\"sHh6ARbxJGDwU4d2V\"}}}"]
  executor = "docker"
  [runners.docker]
    tls_verify = false
    image = "ruby:2.1"
    privileged = false
    disable_cache = false
    volumes = ["/cache", "/home/gitlab-runner/.ssh:/home/gitlab-runner/.ssh:rw", "/home/gitlab-runner/supporting_files:/home/gitlab-runner/supporting_files:rw"]
    shm_size = 0
  [runners.cache]
```

Finally restart your runners

```
systemctl gitlab-runner restart
```

And your rate limit is now removed.
