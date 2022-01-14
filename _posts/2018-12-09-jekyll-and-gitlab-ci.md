---
layout: post
title:  "Creating Jekyll Pages with Gitlab-CI"
date:   2018-12-09 21:20:00 +0100
categories: docker gitlab gitlab-runner jekyll
---

I just started that blog and decided to use [Jekyll](https://jekyllrb.com/) for it. 

With that came also the desire to check the changes into my own gitlab repo and then have the CI run update my page automatically. 

I had a look into [gitlab-pages](https://docs.gitlab.com/ee/user/project/pages/) but decided to try something else.

I installed a gitlab runner (https://docs.gitlab.com/runner/install/docker.html) as a docker container and gave that a try. Unfortunately this defaults to run docker in docker and for security reasons I am really not comfortable with that. So I decided to just create a docker image on top of `https://hub.docker.com/r/gitlab/gitlab-runner/`, which can run ruby and bundler as the gitlab-runner user.

Here is the Dockerfile:

```
FROM gitlab/gitlab-runner:latest

RUN apt-get update && apt-get -y dist-upgrade && \
    apt-get install -y ruby ruby-dev libssl-dev build-essential && \
                apt-get clean && \
                rm -rf /var/lib/apt/lists/*
RUN gem install bundler
```

That's essentially it. This and the following config.toml and `.gitlab-ci.yaml` made it work:


config.toml
```
concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "ruby"
  url = "http://gitlab"
  token = "XXXXXXXXXXXXXXXXXXXXX"
  executor = "shell"
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
```


.gitlab-ci.yaml
```
create:
  script:
  - bundle install --path ~/bundle
  - bundle exec jekyll build -d /pages/destination
  tags:
    - ruby
  only:
  - master
```

The directory `/pages/destination` now contains all the static files and I just run a `nginx:alpine` container that serves those. 