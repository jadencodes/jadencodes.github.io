---
layout: post
title:  "CV CTF - Over Confident CISO"
date:   2022-08-31 11:52:27 -0600
categories: ctf cloudvillage
tags: defcon ctf cloudvillage
description: "Writeup/Solution to Defcon 2022 Cloud Village CTF: OverConfident CISO (200 points)"
---

## Prompt

Oftentimes, background information (including things like entire fake companies that have been created) will precede the prompt:

>Sensorby LLC is a security boutique which focuses on web app security. It's research and devops team is creating a new web application scanner which they say "finds bugs faster than bug hunters". But one of its security engineers told me personally that while the product is amazing, the companies secops sucks - plaintext passwords in sticky notes, saving credentials in VMs and containers, etc.

Finally, the objective:
>Recently there was as attack where the attackers stole the source code of the scanner because someone made the project public. The issue is now resolved. While the CISO seems to be confident about security of its AWS cloud environment, kops setup and source code management, the security engineer is skeptical.

That's it, that is all the information we are given to go find a 32 character flag _somewhere_ on the internet.

### What we know so far

1. This is the first question (worth 200 points), let's not _overthink_ it
2. The company name is `Sensorby`

However, the largest "push" they have given us is this quote: 
>the companies secops sucks - plaintext passwords in sticky notes, saving credentials in VMs and containers, etc.

From this we assumed we were looking for (at least to start) credentials in VMs or containers.

### Blind searching

At this point it's a lot of guessing, trying different things, documenting and sharing _anything_ even remotely relevant with the team.

[docker containers](https://www.docker.com/) are something I use every day and am very comfortable finding, building, and deploying so I figured I would start there. The first place I looked was the most popular **public** docker container registry: [docker hub](https://hub.docker.com/). I just started blindly searching for anything and everything around `Sensorby`.

Nothing was coming up there, so I started looking at the VM aspect. For this, I started in the cloud provider I am most familiar with: `AWS`. To look for virtual machines, I went to `EC2 -> AMI` and began the same blind searching through community VMs. Again nothing really came up.

### First clue

If it was in another cloud providers VM, I was not the correct person to find it, so I started casting a wider net on the container lookup. There are other registries as well so I started going through them. 

After running out of things to search in `docker hub`, I moved to look at `quay.io`, but the same nothingness. I then had a recollection that ECR (elastic container registry) had announced changes to their public repository `Gallery` somewhat recently so I figured I'd look there. First search of [`Sensorby`](https://gallery.ecr.aws/?searchTerm=Sensorby) on the public registry and **boom** we had something:

![ECR Sensorby search result](/assets/images/sensorby-ecr-gallery.png)

This matched even more perfectly than we could have hoped, including the "LLC" suffix.

I poked around in the `Sensorby LLC` Organization but there wasn't any other images or information.

#### The image

First thing we did was pull and run the image:

```console
docker pull public.ecr.aws/sensorby/serialization-dumper:latest

jaden@monstera:~$ docker run public.ecr.aws/sensorby/serialization-dumper:latest
Usage:
	SerializationDumper <hex-ascii-data>
	SerializationDumper -f <file-containing-hex-ascii>
	SerializationDumper -r <file-containing-raw-data>

Rebuild a dumped stream:
	SerializationDumper -b <input-file> <output-file>
```

So it looks like it's a program that does some sort of serialization.

Hopped in the image and started poking around:
```console
jaden@monstera:~$ docker run --rm -it --entrypoint /bin/sh public.ecr.aws/sensorby/serialization-dumper:latest
~ # env
HOSTNAME=9549b92c61c3
SHLVL=1
HOME=/root
JAVA_VERSION=13-ea+32
TERM=xterm
JAVA_URL=https://download.java.net/java/early_access/alpine/32/binaries/openjdk-13-ea+32_linux-x64-musl_bin.tar.gz
PATH=/opt/openjdk-13/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
JAVA_HOME=/opt/openjdk-13
PWD=/root
JAVA_SHA256=94209163894c77623e1bf5581d5fd39d5cf504eb42b82e243e24daff0f9352b0
~ # history
   0 env
   1 history
~ # ls -la
total 28
drwx------    1 root     root          4096 Sep  1 03:19 .
drwxr-xr-x    1 root     root          4096 Sep  1 03:19 ..
-rw-------    1 root     root            19 Sep  1 03:19 .ash_history
-rw-r--r--    1 root     root         14712 Jul 23 16:14 SerializationDumper.jar

```

I was _hoping_ there would be credentials somewhere in the `env` or possibly a left over history file where a command could be seen but those seem clean. The only thing we have is the `SerializationDumper.jar`, I figured I'd spend another few minutes poking about the filesystem.

Maybe this was the one and only step in the question for once? If the flag is in a file, it is commonly in `flag.txt`:

```console
~ # find / -name "*flag*"
<all just normal files>
...
```

Last check. Although brute force and/or craching is rarely involved, I may as well check:

```console
~ # cat /etc/shadow 
root:!::0:::::
bin:!::0:::::
daemon:!::0:::::
adm:!::0:::::
...
```

Nope.

#### The jar file

At this point we are pretty committed to thinking the `SerializationDumper.jar` is the key to the next step. Maybe there was hard coded credentials?

I grabbed a [java decompiler](http://java-decompiler.github.io/) and dumped the program in it.

Nothing really stood out in the file, it was a fairly complex java program to be sure (also because all of the comments had been removed by the compiler), but the file itself seemed purely functional with nothing hidden.

Here is a snippet from the decompiled `SerializationDumper.class`:
```java
  private void parseStream() throws Exception {
    if (((Byte)this._data.peek()).byteValue() != -84) {
      byte b = ((Byte)this._data.pop()).byteValue();
      switch (b) {
        case 80:
          print("RMI Call - 0x50");
          break;
        case 81:
          print("RMI ReturnData - 0x51");
          break;
        case 82:
          print("RMI Ping - 0x52");
          break;
        case 83:
          print("RMI PingAck - 0x53");
          break;
        case 84:
          print("RMI DgcAck - 0x54");
          break;
        default:
          print("Unknown RMI packet type - 0x" + byteToHex(b));
          break;
      } 
    } 
    byte b1 = ((Byte)this._data.pop()).byteValue();
    byte b2 = ((Byte)this._data.pop()).byteValue();
    print("STREAM_MAGIC - 0x" + byteToHex(b1) + " " + byteToHex(b2));
...
```

Finally the thought came to ya know, google it? [serialization dumper](https://github.com/NickstaDB/SerializationDumper) is (from the README):

> A tool to dump and rebuild Java serialization streams and Java RMI packet contents in a more human readable form.

Just to be safe we checked the content of the files and they were the same. We couldn't just check the SHAs because those would of course be different.

At this point I switch to a different problem after noting the findings in our discord channel.

#### Back to the image

After a few more ~~drinks~~ hours, I went back to the image, at this point both my teammates and myself have checked every environment variable, softlink, and daemon of the **running** container and nothing is sticking out.

I have a thought: we need to see _how_ the image was made. A quick search in Github for the sensorby Dockerfile didn't yield any results so I would have to look at the next best option: `docker history`.

Without going too off topic or in depth, docker images are composed of one or more seperate layers, and each layer is an image, just without a tag.

Docker history shows each of the _layers_ (images) and there is one per Dockerfile instruction. The key part is they store the _difference_ between the layers, not the _absolute_ image (for lack of a better term) at that stage. 

For example, the following `Dockerfile`:

```docker
FROM alpine:latest

EXPOSE 8080

RUN /bin/sh
```

Would generate three individual layers (images), each with additions from the previous layer(s).

Back to the image at hand:

```console
jaden@monstera:~$ docker history public.ecr.aws/sensorby/serialization-dumper:latest 
IMAGE          CREATED       CREATED BY                                      SIZE      COMMENT
c76f6ff58ae2   5 weeks ago   /bin/sh -c #(nop)  ENTRYPOINT ["java" "-jar"…   0B        
<missing>      5 weeks ago   /bin/sh -c rm -rf /usr/src/SerializationDump…   0B        
<missing>      5 weeks ago   /bin/sh -c cd /usr/src/SerializationDumper &…   43.3kB    
<missing>      5 weeks ago   /bin/sh -c #(nop) WORKDIR /root                 0B        
<missing>      5 weeks ago   /bin/sh -c #(nop) COPY dir:3e53921e7d621f972…   138kB     
<missing>      3 years ago   /bin/sh -c #(nop)  CMD ["jshell"]               0B        
<missing>      3 years ago   /bin/sh -c set -eux;   wget -O /openjdk.tgz …   331MB     
<missing>      3 years ago   /bin/sh -c #(nop)  ENV JAVA_SHA256=942091638…   0B        
<missing>      3 years ago   /bin/sh -c #(nop)  ENV JAVA_URL=https://down…   0B        
<missing>      3 years ago   /bin/sh -c #(nop)  ENV JAVA_VERSION=13-ea+32    0B        
<missing>      3 years ago   /bin/sh -c #(nop)  ENV PATH=/opt/openjdk-13/…   0B        
<missing>      3 years ago   /bin/sh -c #(nop)  ENV JAVA_HOME=/opt/openjd…   0B        
<missing>      3 years ago   /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B        
<missing>      3 years ago   /bin/sh -c #(nop) ADD file:0eb5ea35741d23fe3…   5.58MB    file:0eb5ea35741d23fe39cbac245b3a5d84856ed6384f4ff07d496369ee6d960bad in / 
```

The top three layer commands stick out, lets look at them fully (with `--no-trunc` flag):
```console
# final image
/bin/sh -c #(nop)  ENTRYPOINT ["java" "-jar" "./SerializationDumper.jar"]

# second from final
/bin/sh -c rm -rf /usr/src/SerializationDumper

# third from final
/bin/sh -c cd /usr/src/SerializationDumper &&  mkdir out &&  javac -cp ./src ./src/nb/deser/SerializationDumper.java ./src/nb/deser/support/*.java -d ./out &&  jar cvfm SerializationDumper.jar MANIFEST.MF -C ./out/ . &&  mv SerializationDumper.jar /root/
```

It is very normal to use docker containers for building programs and removing the source code afterwards but what sticks out is the second to last layer; The layer that _attempts_ to clean up the build directory with `rm -rf /usr/src/SerializationDumper` after a layer.

If they had done this on the build layer `cd /usr/src/SerializationDumper && mkdir out && .... `, it would've removed the source code from the layer before it was created. However, running the `rm -rf` command on the next layer does indeed delete it from the final image, but it doesn't remove it from the layer before it. We need to look at the third from last layer.

#### Taking apart the container

To get the sublayers we save the image to a `.tar` and then uncompress it:

```console
jaden@monstera:~$ docker save public.ecr.aws/sensorby/serialization-dumper:latest > serialization-dumper.tar
jaden@monstera:~$ tar -xvf serialization-dumper.tar 
```

This creates a folder for each of the layers that have content changes (ie where SIZE is >0B):

```console
08e4b6c99bb09afeaae122f0a2c04b27812ee447df70cc35c25649d002d06154/
08e4b6c99bb09afeaae122f0a2c04b27812ee447df70cc35c25649d002d06154/VERSION
08e4b6c99bb09afeaae122f0a2c04b27812ee447df70cc35c25649d002d06154/json
08e4b6c99bb09afeaae122f0a2c04b27812ee447df70cc35c25649d002d06154/layer.tar
1630cb181263803399fe15f19fa114fcd09c5aa098838f9a3d586162bc8649a3/
1630cb181263803399fe15f19fa114fcd09c5aa098838f9a3d586162bc8649a3/VERSION
1630cb181263803399fe15f19fa114fcd09c5aa098838f9a3d586162bc8649a3/json
1630cb181263803399fe15f19fa114fcd09c5aa098838f9a3d586162bc8649a3/layer.tar
2365571e67964552035079f0d2f434e61dc199ba16dd3e17b7bfe7fd626512fd/
2365571e67964552035079f0d2f434e61dc199ba16dd3e17b7bfe7fd626512fd/VERSION
2365571e67964552035079f0d2f434e61dc199ba16dd3e17b7bfe7fd626512fd/json
2365571e67964552035079f0d2f434e61dc199ba16dd3e17b7bfe7fd626512fd/layer.tar
b98b4b245daec7965f286ec011ca14507978f6da57fd93ff88dce839fb936931/
b98b4b245daec7965f286ec011ca14507978f6da57fd93ff88dce839fb936931/VERSION
b98b4b245daec7965f286ec011ca14507978f6da57fd93ff88dce839fb936931/json
b98b4b245daec7965f286ec011ca14507978f6da57fd93ff88dce839fb936931/layer.tar
c76f6ff58ae2ea0f13fce9685ced9ca9457907b42540915cebabef999c14640e.json
d18798d8c6aab9eabe691cc3eb50c356fd2ffe6a7d2d7dd22df3e2f9d94e9dd4/
d18798d8c6aab9eabe691cc3eb50c356fd2ffe6a7d2d7dd22df3e2f9d94e9dd4/VERSION
d18798d8c6aab9eabe691cc3eb50c356fd2ffe6a7d2d7dd22df3e2f9d94e9dd4/json
d18798d8c6aab9eabe691cc3eb50c356fd2ffe6a7d2d7dd22df3e2f9d94e9dd4/layer.tar
manifest.json
repositories
```

From here I went into each of the folders (layers) alphanumerically and untarred the layer itself:
```console
jaden@monstera:~/08e4b6c99bb09afeaae122f0a2c04b27812ee447df70cc35c25649d002d06154$ tar -xvf layer.tar 
usr/
usr/src/
usr/src/.wh.SerializationDumper
```

Until finally, one layer stuck out:
```console
jaden@monstera:~/2365571e67964552035079f0d2f434e61dc199ba16dd3e17b7bfe7fd626512fd$ tar -xvf layer.tar 
usr/
usr/src/
usr/src/.wh..wh..opq
usr/src/SerializationDumper/
usr/src/SerializationDumper/.git/
usr/src/SerializationDumper/.git/HEAD
usr/src/SerializationDumper/.git/branches/
usr/src/SerializationDumper/.git/config
usr/src/SerializationDumper/.git/description
...
```

This was the third from last layer, the one with the history of:
```console
/bin/sh -c cd /usr/src/SerializationDumper &&  \
mkdir out &&  \
javac -cp ./src ./src/nb/deser/SerializationDumper.java ./src/nb/deser/support/*.java -d ./out &&  \
jar cvfm SerializationDumper.jar MANIFEST.MF -C ./out/ . && \
mv SerializationDumper.jar /root/
```

Specifically this layer has the **source code** in it. But more importantly, it is a full git repository!

After navigating into the subdirectory with the git repository (./usr/src/SerializationDumper) I start to poke around.

Keeping in mind this is largely (or completely) based off of the publicly available `SerializationDumper` from earlier, I wanted to know the recent git commits:

```console
$ git log -p
commit 560139176df9e2121a63d3b893632565f1a719d8 (HEAD -> master, origin/master, origin/HEAD)
Author: DevOps Lead <sensorby-devops@local>
Date:   Sun Jun 12 21:28:45 2022 +0530

    Update readme

diff --git a/README.md b/README.md
index 1c62328..1931f95 100644
--- a/README.md
+++ b/README.md
@@ -11,6 +11,12 @@ Download v1.11 built and ready to run from here: [https://github.com/NickstaDB/S
 
 **Update 19/12/2018:** SerializationDumper now supports rebuilding serialization streams so you can dump a Java serialization stream to a text file, modify the hex or string values, then convert the text file back into a binary serialization stream. See the section below on [Rebuilding Serialization Streams](#rebuilding-serialization-streams) for an example of this.
 
+## Sensorby Notes
+
+Since our system was recently optimized to make our web app scans faster, a lot of internal traffic happens using Java serialization stream. To debug any issue, use this tool to decode the Java serialization to human readable form and them check if its an issue with the code or data passed to it.
+
+**Note:** This tool was not tested across all repositories due to lack of time. However testing it against our own code base asclepius backend showed the tool to be promising. Please reach out to us in our slack channel #project-asclepius.
+
 ## Building
 Run `build.sh` or `build.bat` to compile the JAR from the latest sources.

```

I finally see the keyword I have been searching for to confirm I was on the right track: **Sensorby**.

Here is the full quote from the git commit `560139176df9e2121a63d3b893632565f1a719d8`:

<blockquote>
Since our system was recently optimized to make our web app scans faster, a lot of internal traffic happens using Java serialization stream. To debug any issue, use this tool to decode the Java serialization to human readable form and them check if its an issue with the code or data passed to it.

**Note:** This tool was not tested across all repositories due to lack of time. However testing it against our own code base asclepius backend showed the tool to be promising. Please reach out to us in our slack channel #project-asclepius.
</blockquote>

We are given yet another project to search for: **asclepius**.

Lets look at the `.git/config` to see where this repository is hosted, assuming the other project would be in the same place.

```console
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
[remote "origin"]
	url = https://sensorby:c8qGEYjH4j4kcaECx3QT@bitbucket.org/sensorby-devops/SerializationDumper.git
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
	remote = origin
	merge = refs/heads/master
```

Credentials at last! We see the `sensorby:c8qGEYjH4j4kcaECx3QT@bitbucket.org` telling us the credentials and _where_ the company `Sensorby` likely hosts their codebases.

Lets try and clone the new project assuming the same credentials will work:

```console
$ git clone https://sensorby:c8qGEYjH4j4kcaECx3QT@bitbucket.org/sensorby-devops/asclepius.git
Cloning into 'asclepius'...
remote: Enumerating objects: 9, done.
remote: Counting objects: 100% (9/9), done.
remote: Compressing objects: 100% (5/5), done.
remote: Total 9 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (9/9), 789 bytes | 394.00 KiB/s, done.
```

Going into this directory I poked around and again ran `git log -p` to see recent changes:

```console
commit bf428d2b03eeb750e73d633ae39b52317cb11aae
Author: Adam Gilchrist <sensorby-devops@local>
Date:   Sat Jul 23 22:10:00 2022 +0530

    Update config

diff --git a/config.yml b/config.yml
index 19794eb..ce37b1a 100644
--- a/config.yml
+++ b/config.yml
@@ -1 +1,3 @@
 # You're really close
+
+You have tried really hard!!

commit 85c262cc4763a5c676524612abeda614ca339d66
Author: Adam Gilchrist <sensorby-devops@local>
Date:   Sat Jul 23 22:09:28 2022 +0530

    FLAG-{TnSlqeeEnsWstyBzgzkRpjMKUbnlOymg}

diff --git a/config.yml b/config.yml
new file mode 100644
```

There's the flag, hidden in a git commit message: `FLAG-{TnSlqeeEnsWstyBzgzkRpjMKUbnlOymg}`
