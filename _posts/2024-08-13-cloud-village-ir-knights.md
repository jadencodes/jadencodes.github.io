---
layout: post
title:  "CV CTF - IR-Knights Assistant"
date:   2024-08-13 10:52:27 -0600
categories: ctf cloudvillage
tags: defcon ctf cloudvillage
description: "Writeup/Solution to Defcon 2024 Cloud Village CTF: IR-Knights Assistant"
---

## Prompt

I didn't actually save the prompt for this one, however, this was not solved in the traditional way...

## Solution

We accidentally found this flag beforehand while looking for another flag. This generally happens at least once per CTF.

Some reasons this has happened:
- We found a user from one of the fake companies they setup on github that had a different challenge repo for the year
- The AWS owner ID was the same for a few challenges
- Dockerhub or public ecr had multiple repos for the same fake companies but used in different challenges

This one was found for the latter. Specifically, while we were _wayyyy_ overthinking the first problem **Dracarys**, I searched on https://gallery.ecr.aws for "Dracarys" and found this user: https://gallery.ecr.aws/c2y9x9i6?page=1.

This user had 5 image repos, so of course I went through each looking for clues to a different question... While doing so one stood out [irk-assistant](https://gallery.ecr.aws/c2y9x9i6/irk-assistant).

Why? Because it was the only of the 5 repos to have more than "latest" image tags available:

![irk-assistant-tags](/assets/images/ctf/cv2024/irk-assistant-image-tags.png)

Pulling down the earlier version (v0.0.1) and execing in:

```console
$ docker run --rm -it --entrypoint /bin/bash public.ecr.aws/c2y9x9i6/irk-assistant:v0.0.1
root@f2b29b8869cf:/app# ls
Dockerfile  IR-Knights.py  app  requirements.txt
root@f2b29b8869cf:/app# ls -la
total 28
drwxr-xr-x 1 root root 4096 Jul 18 02:32 .
drwxr-xr-x 1 root root 4096 Aug 14 04:39 ..
-rwxr-xr-x 1 root root   46 Jul 18 00:40 .env
-rwxr-xr-x 1 root root  428 Jul 18 02:20 Dockerfile
-rwxr-xr-x 1 root root 3478 Jul 18 00:41 IR-Knights.py
drwxr-xr-x 2 root root 4096 Jul 18 01:26 app
-rwxr-xr-x 1 root root 1638 Jul 13 16:29 requirements.txt
root@f2b29b8869cf:/app# cat .env
FLAG='FLAG-{iSj9Ip7WJN1QZKb4GRs1sygWSHHj6Fsu}'
```

Tada! Sometimes a little luck is required. However, I kept dumbly submitting this for the "Dracarys" question so it took about an hour to realize the mistake.
