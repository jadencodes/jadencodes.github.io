---
layout: post
title:  "CV CTF - Big Bad Darkweb"
date:   2022-09-09 22:45:27 -0600
categories: ctf cloudvillage
tags: defcon ctf cloudvillage fcrackzip
description: "Writeup/Solution to Defcon 2022 Cloud Village CTF: Big Bad Darkweb (400 points)"
---

## Prompt

>Awesome Tech Inc. has an amazing security team, some of the known names in the hacker forums and community. There CISO has asked them to demoing a new security tool into their tooling due to the insistence of CEO and CFO. The tool markets themselves as to provide complete visibility into the cloud environments.

Also included was a file, containing an email thread.

**Hint:**
>The security engineer is a member of hak5 forums.

### File

```eml
From: Security Engineer <talented-security-engineer@awsometech.in>
To: Sales Engineer <sales-engineer@overhyped-security-vendor.com>
CC: "Mehh.. Security Manager" <manager@awseometech.inc>, "VP Sales"
	<vpofsales@overhyped-security-vendor.com>
Subject: Awesome Tech Inc. -  Connector Credentials
Thread-Topic: Awesome Tech Inc. -  Connector Credentials
Thread-Index: AQHYrAa6wFWBlUFBzEmRuOLLJIZZrQ==
X-MS-Exchange-MessageSentRepresentingType: 1
Date: Tue, 9 Aug 2022 15:48:21 +0000
Message-ID:
	<TY0PR02MB5945EFB71C55BB565DEACE48DA629@TY0PR02MB5945.apcprd02.prod.outlook.com>
Content-Language: en-IN
X-MS-Has-Attach: yes
X-MS-Exchange-Organization-SCL: -1
X-MS-TNEF-Correlator:
X-MS-Exchange-Organization-RecordReviewCfmType: 0
msip_labels:
Content-Type: multipart/mixed;
	boundary="_002_TY0PR02MB5945EFB71C55BB565DEACE48DA629TY0PR02MB5945apcp_"
MIME-Version: 1.0

--_002_TY0PR02MB5945EFB71C55BB565DEACE48DA629TY0PR02MB5945apcp_
Content-Type: text/plain; charset="iso-8859-1"
Content-Transfer-Encoding: quoted-printable

Hello,=0A=
=0A=
I am sending over the credentials for the connector from our account that y=
ou guys can use to and run your script to audit. =0A=
=0A=
This is a password protected zip and I will be sharing the password on your=
 phone. =0A=
=0A=
Thanks.=

--_002_TY0PR02MB5945EFB71C55BB565DEACE48DA629TY0PR02MB5945apcp_
Content-Type: application/zip; name="credentials.zip"
Content-Description: credentials.zip
Content-Disposition: attachment; filename="credentials.zip"; size=2068;
	creation-date="Tue, 09 Aug 2022 15:44:50 GMT";
	modification-date="Tue, 09 Aug 2022 15:48:22 GMT"
Content-Transfer-Encoding: base64

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

--_002_TY0PR02MB5945EFB71C55BB565DEACE48DA629TY0PR02MB5945apcp_--
```

The email addresses in this thread are humorous: "talented-security-engineer@awsometech.in", "sales-engineeer@overhyped-security-vendor.com". An interesting note is that domain is misspelled _sometimes_: "awseometech.inc". We initially thought this might be a clue and dig some dead ended hunting to the domains.

The most relevant thing in this thread is clearly the attached file. The sender states that the zip file is password protected itself and provides credentials to the tool in their demo account. The password for the zip file will be shared over a text message, to separate the "knowledge" from the "possession".


### Making the zip file

The attachment metadata gives us the following information:

- Filename: `credentials.zip`
- Encoding: `base64`

With these two pieces, we can make the zip file:

```console
echo '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' \
| base64 -d > credentials.zip
```

We can verify that it is actually a zip file by checking the first few bytes.

```console
xxd credentials.zip | head
00000000: 504b 0304 1400 0900 0800 98be 0855 ffca  PK...........U..
00000010: bd9a 6e06 0000 1a09 0000 1a00 1c00 636f  ..n...........co
```

`0x04034b50` (little endian) is the correct [file signature](https://en.wikipedia.org/wiki/ZIP_(file_format)) for a zip file, so our first four bytes are correct: `504b 0304`.


And let's do a quick test to see if it is a password protected zip:

```console
$ unzip credentials.zip 
Archive:  credentials.zip
[credentials.zip] connector-credentials.json password: 
password incorrect--reenter: 
```

Ok, so they haven't lied to us, we do indeed have a zip file that _likely_ contains the "connector credentials". How do we get the missing credentials?

### Brute force

First, we need some software that can guess zip file passwords for us. A quick search showed me [fcrackzip](https://www.kali.org/tools/fcrackzip/), so I installed it.

```console
$ sudo apt install fcrackzip
```

It is extremely rare for CTFs to require pure brute force: 0000, 0001, 0002, ..., 9999; since _generally_ these challenges are about testing skills and not "who has a more expensive machine", but hey, nothing is impossible, so I gave `fcrackzip` a try:

```console
$ fcrackzip -b -u credentials.zip
```

The `-b` flag is for brute force and the `-u` flag is to use `unzip` command to verify passwords (without this I was getting a lot of false positives).

After a few minutes of nothing I killed the command since I assumed at this point there was more to the challenge. However, a fellow teammate got a chonky 32 core EC2 instance and let it run there overnight, but also failed.

I decided to try a different and more likely approach: the dictionary attack. Instead of guessing `aaaa` to `zzzz` I would provide a list of common/possible passwords to try like "password", "pizza", "password1!", etc... These type of cracks _are_ common in CTFs since the number of guesses being made is however long your list is (say 100k passwords) rather than the entire keyspace and therefore the difference between 4 and say 32 cores is negligible.

There are lots of password wordlist out there, but I picked the infamous [rockyou.txt](https://www.kaggle.com/datasets/wjburns/common-password-list-rockyoutxt):

```console
$ fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt credentials.zip
```

This took a couple of minutes on my laptop to complete (almost 140 million passwords) and failed; since it was one of the largest collective lists of passwords, I determined we were likely missing another part of the puzzle.


### The hint

The hint provided on day 2 was the key part. Without it, I'm not sure if we would've ever gotten it. Heck, even with it, it still took a long time.

>The security engineer is a member of hak5 forums.


I had never been to the [Hak5 forums](https://forums.hak5.org/), but this was _clearly_ where they were pushing us with the hint. At first, I didn't really know what I was looking for, but I was assuming that one of the names in the email would be involved, such as "awesometech.in" or "talented-security-engineer". After a bunch of searching (which turned out to be rate limited and thus even slower) I looked at the [staff page](https://forums.hak5.org/staff/) and still nothing stuck out. After clicking through some categories like [security](https://forums.hak5.org/forum/43-security/) I noticed that there weren't _that_ many posts; maybe 1-5 per day?

So I spent the next couple of hours painstakingly combing over every single post between the start of the ctf, and a couple of months prior.

Absolutely nothing. Not even a similar keyword...

I had a thought: what do I know about hak5?

>Hak5 makes hardware tools

They make awesome hacker tools like the [wifi pinapple](https://shop.hak5.org/products/wifi-pineapple) and [usb rubber ducky](https://shop.hak5.org/products/usb-rubber-ducky).

This leads me to a second question: what does their hardware need to run?

>Firmware

So I checked their [download](https://downloads.hak5.org/) page and didn't find anything that stood out, it was all firmware. I wondered if they didn't have accompanying software or helpers like wordlists, so I finally googled: "hak5 wordlist".

Money, the first result is a github repo with a [hak5 wordlist](https://github.com/7z-sec/Wordlist/blob/master/Hak5.txt). There isn't any helpful information in this repository, and it hasn't been touched in years, but I figure why not give it a shot.

```console
$ fcrackzip -u -D -p /usr/share/wordlists/hak5.txt credentials.zip 


PASSWORD FOUND!!!!: pw == H4k5.0r9P455w0rd&#33;
```

### The credentials

It was a lot of time an effort to get this far, but I feared that there was more to this problem.

With the password, we can unzip the archive:

```console
$ unzip credentials.zip 
Archive:  credentials.zip
[credentials.zip] connector-credentials.json password: 
  inflating: connector-credentials.json
```

All that for a single file: `connector-credentials.json`

```json
{
  "type": "service_account",
  "project_id": "rapid-gadget-352619",
  "private_key_id": "3a8dc28da3aacbbe2f894c5270c40ca7394e6b56",
  "private_key": "-----BEGIN PRIVATE KEY-----\nMIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQDHcG3EpzCnYRhr\n/D7H2+Iqt/Vitjo68M9RwgyjrqguM6CtE7lsjdQ+RYggulakMdbprM3RlT6GicTV\nwuAnEoYE6sAgUyEzq4183IO/FprJUPtHBdTieu3fzh0aEO5//mLRGFwQg75hvqla\nPP75vW7qNINDPiOlhDry48Abpdf/of2S3nYryM7e+HphwGz+CciaKVfTYfUOtzKY\ngyeTi4ooEYE30FF8i/aKAZUnwbOmkX/NTkDZXqPd0iw+kuWm63mUa+vDvDuZdAkZ\naKZdwgGSwRHxZ6x8KfVIbC/n3iE/aztSkbTEL8qOkN2GOvLgidtpOfPk376aXsMY\nioJilSeJAgMBAAECggEAB1rs0hb0OcYK+CeL5VkaqrvmUDhw0RM9YseNlFyeKKG1\njCXZMv5PjKeJ5SLDdtJ85C5f7aSWXdtB77wRl3X3d0IKPsmbZLwPz6IpIdsv70Wo\nlgvVDYWzqmB1TQyWY9DHnOxsBR1mCurfdMZ2r6frdGEmUqIS6eP5/GgcbICbJNBS\n1Sm/rG2JJppUs7SXeXurGoJOwwZJ3luIYpMXzzKpKWcmW9/DsPnnuOm9SzcTw20Z\njPruQYhAlnPMCTKRTzqV/TYDPZVKFWpTkhzpXjcmwoRZW7+UebilEeKDRendnDo8\nlswL6rNriCpk8X+Io/jb6VOUCixHHTZEIMzxkOtzoQKBgQD+1fcLsIzU+vnoPKgM\nxAhJTgTp4kCrje852wRhq01XcvoJAA2TC6d4mqDo8UrdwgSPNAwz1dbwNsZAm/Ry\n8FEGyB7MBeVV/HTnG6WvqPl57DqL6piBb9umkLjCdy4j1z6rtMlW+9+TImfVYrA0\ndN6ihZOgMWQ4A8NsLgT3NZ3yoQKBgQDIWa0qO2n9QaizX3b8Z2l4CngaSMWk0wzj\n0Hzf2lOiiX2wM6/9TNAQYcO6fpTrJ89UuiFEJMz/02c5JVnO04w4FD3b9z50PcyC\nfl2oyRLMgYkFeuXUH5woT2yoSjzYaxk+9d7H3CDmO0PHITEvIDTgFWkKewZ9sBah\nznV+VrVz6QKBgFPjj63Tcqjx7a6buR5qseefvVJY3r0avjOne6vDPnSZLuIjmFRd\ns8Wp8Wp9dA3IPsP9eD7gGB9/iIfgTvo/Tg0Td7/l+PbzYnBp04Md9vJB54wDsCx7\n7CzK22d44EGAK+tOWjE+PP0siE3gbOz3xApwOoaze7BM3NoR1CSlC9fBAoGBAJ2i\nUSjlTmNBAeb/ubKl+snEEvM7RqaEl6O3KklGkn9UBlxYjqORiDMbeNCHP8w1ql9T\ng1EGU3UFdDX2OU8OC0kkQ/eJ9M2owfv6SN7ANdZKJPD23VWk+UyOEUPoBS+SNG7h\nLMO7YvdCsfU/HF+jy3Zz4g2o9lZ18ZilxLP+rQ3hAoGAUCHa5DeslPDHtzrLPRft\nbUSArNGPMzz8p27uUj1SX16HYXgUu1gaikUt1H1iWh+d+yUDByxpmEyBhSWpmSxF\nU5BFR0rdo2ntcSpf77Xc8TFIOHEGvkMoGDPe7WTWwsckz7928ThnrSMR270oBPWZ\nTCFcubRCSYJk4t/AxiW9PDs=\n-----END PRIVATE KEY-----\n",
  "client_email": "connector@rapid-gadget-352619.iam.gserviceaccount.com",
  "client_id": "103377401300816111022",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/connector%40rapid-gadget-352619.iam.gserviceaccount.com"
}
```

I hadn't seen one of these files before, but from it, we get a few pieces of information:
- It's a service account
- It's on googles cloud
- The project id is `rapid-gadget-352619`
- The client id `103377401300816111022`
- An email `connector@rapid-gadget-352619.iam.gserviceaccount.com`

A bit of googling around for "gcp service account auth" brings up [this](https://cloud.google.com/sdk/gcloud/reference/auth/activate-service-account) relevant page that included a command to configure our gcp client with a file like the one above:

```console
$ gcloud auth activate-service-account --key-file=connector-credentials.json

Activated service account credentials for: [connector@rapid-gadget-352619.iam.gserviceaccount.com]
```

This configured our client with the respective information listed earlier.

I didn't know pretty much any gcloud CLI commands, so I poked around with a few.

```text
access-approval          apigee                   billing                  compute                  datastore                edge-cloud               firestore                ids                      ml                       organizations            recommender              service-directory        transcoder 
access-context-manager   app                      bms                      config                   datastream               emulators                functions                info                     ml-engine                org-policies             redis                    services                 transfer 
active-directory         artifacts                builds                   container                debug                    endpoints                game                     init                     monitoring               policy-intelligence      resource-manager         source                   version 
ai                       asset                    certificate-manager      database-migration       deploy                   essential-contacts       healthcare               iot                      network-connectivity     policy-troubleshoot      resource-settings        spanner                  workflows 
ai-platform              assured                  cheat-sheet              data-catalog             deployment-manager       eventarc                 help                     kms                      network-management       privateca                run                      sql                      workspace-add-ons 
alpha                    auth                     cloud-shell              dataflow                 dns                      feedback                 iam                      logging                  network-security         projects                 scc                      survey                   
anthos                   beta                     components               dataplex                 docker                   filestore                iap                      memcache                 network-services         pubsub                   scheduler                tasks                    
api-gateway              bigtable                 composer                 dataproc                 domains                  firebase                 identity                 metastore                notebooks                recaptcha                secrets                  topic
```

My first thought was that since the prompt was talking about a demo service, perhaps the next step was finding a running server or an image.

```console
$ gcloud compute instances list
ERROR: (gcloud.compute.instances.list) Some requests did not succeed:
 - Required 'compute.instances.list' permission for 'projects/rapid-gadget-352619'

$ gcloud compute images list

WARNING: Some requests did not succeed.
 - Required 'compute.images.list' permission for 'projects/rapid-gadget-352619'
```

No permission on these. At this point my thought is that maybe this account is overprivileged, meaning we had access to another gcloud service (or as gcloud cli calls them "groups") that we weren't intended to.

We started manually iterating over a few of them until we finally had permission on one: `gcloud iam`. Our thought is that maybe we can list the exact roles our account has access to lead us to the next steps.

```console
$ gcloud iam roles list --project=rapid-gadget-352619 
---
description: 'Created on: 2022-06-19'
etag: BwXhyPOg-aA=
name: projects/rapid-gadget-352619/roles/CustomRole
title: gcp-storage-challenge
---
description: 'Created on: 2022-06-19'
etag: BwXhyXLLn84=
name: projects/rapid-gadget-352619/roles/CustomRole234
title: storage-challenge
---
description: 'Created on: 2022-07-15'
etag: BwXj0uxXAOs=
name: projects/rapid-gadget-352619/roles/CustomRole290
title: op
---
description: FLAG-{lIP67x26UvRwFU3H3IUoXSqrDSCXy965}
etag: BwXlvhslP4M=
name: projects/rapid-gadget-352619/roles/CustomRole355
stage: GA
title: connector
```

It turns out that this was the last step, and at first glance we almost missed it!
The flag is the description of `CustomRole355`: `FLAG-{lIP67x26UvRwFU3H3IUoXSqrDSCXy965}`

