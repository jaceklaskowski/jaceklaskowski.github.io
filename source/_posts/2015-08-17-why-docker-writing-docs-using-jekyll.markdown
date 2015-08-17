---
layout: post
title: "Why Docker - writing docs using Jekyll"
date: 2015-08-17 11:06:45 +0200
comments: true
categories: [docker]
sidebar: collapse
keywords: docker, jekyll, documentation
published: true
---
**Spoiler** I'm so much into [Docker](https://www.docker.com/) that I could sing songs about how much it made my life easier. And yours soon, too.

Just today I've got a request to review changes to introduce [Jekyll](http://jekyllrb.com/) as the documentation framework. I was earlier proposing it myself so I knew what the outcome of the review could be - APPROVED.

*But...*

I also knew that the guy who proposed the change was fighting the installation of Ruby gems and Jekyll to have a complete working environment for the documentation system on his own laptop. He was on Linux while I'm on Mac OS. He finally got it sorted out, but the final solution was not satisfactory to me - he installed additional dependencies onto his local machine directly and suggested the very same steps in README so others could follow his steps. I simply couldn't approve it. Sorry.

The alternative was to use Docker and the [docker-jekyll](https://github.com/jekyll/docker-jekyll) image. It makes Jekyll running in a self-contained container with no system-wide interaction with the local machine. And it's officially supported and maintained by the Jekyll project itself.

<!-- more -->

## Using Jekyll inside Docker

The steps to use Jekyll without installing Ruby, gems and other dependencies locally is to use the [docker-jekyll](https://github.com/jekyll/docker-jekyll) image. It's very safe because it's a self-contained image and I'm not "infecting" my local working machine in any way (except installing Docker and pulling the image, but that's acceptable).

Install Docker on your platform and do the following.

Go to the `docs` directory where your documentation lives and execute:

    docker run --rm --volume=$(pwd):/srv/jekyll -t -p 4000:4000 jekyll/stable jekyll build

It will generate the docs from the sources and save the output into the current working directory (under `_site`).

Serve the docs using `jekyll s` which is the default command while spinning up a new docker-jekyll container.

    docker run --rm --volume=$(pwd):/srv/jekyll -t --name=jekyll -p 4000:4000 jekyll/stable

Mind the name for the container - **jekyll** - so it's easier to work with it later on.

    ➜  docs git:(39fd9c9) ✗ docker run --rm --volume=$(pwd):/srv/jekyll -t --name=jekyll -p 4000:4000 jekyll/stable
    Configuration file: /srv/jekyll/_config.yml
                Source: /srv/jekyll
           Destination: /srv/jekyll/_site
          Generating...
                        done.
     Auto-regeneration: enabled for '/srv/jekyll'
    Configuration file: /srv/jekyll/_config.yml
        Server address: http://0.0.0.0:4000/
      Server running... press ctrl-c to stop.

Open http://0.0.0.0:4000/ and have fun!

For people on Mac OS: You are supposed to use [Docker Machine](https://docs.docker.com/machine/) and so the IP address is `docker-machine ip dev` where `dev` is the name of the machine.

Stop the container using `docker stop jekyll`.

    ➜  docs git:(39fd9c9) ✗ docker ps
    CONTAINER ID        IMAGE               COMMAND              CREATED             STATUS              PORTS                    NAMES
    b6bb07d8b8aa        jekyll/stable       "/usr/bin/startup"   26 seconds ago      Up 26 seconds       0.0.0.0:4000->4000/tcp   jekyll
    ➜  docs git:(39fd9c9) ✗ docker stop jekyll
    jekyll
    ➜  docs git:(39fd9c9) ✗ docker ps
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
    ➜  docs git:(39fd9c9) ✗

Happy Dockering!

Let me know what you think about the topic of the blog post in the [Comments](#disqus_thread) section below or contact me at jacek@japila.pl. Follow the author as [@jaceklaskowski](https://twitter.com/jaceklaskowski) on Twitter, too.
