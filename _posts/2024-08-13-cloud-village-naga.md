---
layout: post
title:  "CV CTF - Naga"
date:   2024-08-13 12:52:27 -0600
categories: ctf cloudvillage
tags: defcon ctf cloudvillage kubernetes
description: "Writeup/Solution to Defcon 2024 Cloud Village CTF: Naga"
---

## Prompt

> Guided by whispers of forgotten lore and the faint glimmer of treasure, heroes must unravel the mysteries of this serpent deity to claim their glory. Will you emerge victorious, or will the NAGA's coils tighten around your fate? Explore the caves of the cloud, become the serpent king, find the Another cave hidden in the coils, prepare for a journey where every step could be your last. Only the cunning, courageous and forensically minded shall prevail.

http://the-naga-dragon-prod.s3-website.us-east-2.amazonaws.com/


## Solution

We are given a webpage hosted on AWS S3.

![naga webpage](/assets/images/ctf/cv2024/naga-homepage.png)

With nothing really to go on besides that this challenge was hosted in AWS. I began by pulling all the publicly available AMI's across the regions:

```python3
regions="us-west-1 us-west-2 us-east-1 us-east-2 ap-northeast-1 ap-northeast-2 ap-northeast-3 ap-south-1 ap-southeast-1 ap-southeast-2 ca-central-1 eu-central-1 eu-north-1 eu-west-1 eu-west-2 eu-west-3 sa-east-1"

for r in $regions; do
	echo "Starting $r";
	aws --region $r ec2 describe-images --executable-users all
done
```

Saving that output to a file, I started searching for keywords and found a hit on `Patala`:

```json
{
    "Architecture": "x86_64",
    "CreationDate": "2024-07-06T20:08:37.000Z",
    "ImageId": "ami-0ce9f5f15fa7a6eee",
    "ImageLocation": "590183664633/patala-lok",
    "ImageType": "machine",
    "Public": true,
    "OwnerId": "590183664633",
    "PlatformDetails": "Linux/UNIX",
    "UsageOperation": "RunInstances",
    "State": "available",
    "BlockDeviceMappings": [
        {
            "DeviceName": "/dev/sda1",
            "Ebs": {
                "DeleteOnTermination": true,
                "Iops": 3000,
                "SnapshotId": "snap-0934c66a315ba53c7",
                "VolumeSize": 8,
                "VolumeType": "gp3",
                "Throughput": 125,
                "Encrypted": false
            }
        },
        {
            "DeviceName": "/dev/sdb",
            "VirtualName": "ephemeral0"
        },
        {
            "DeviceName": "/dev/sdc",
            "VirtualName": "ephemeral1"
        }
    ],
    "Description": "You are about to enter Patala!",
    "EnaSupport": true,
    "Hypervisor": "xen",
    "Name": "patala-lok",
    "RootDeviceName": "/dev/sda1",
    "RootDeviceType": "ebs",
    "SriovNetSupport": "simple",
    "VirtualizationType": "hvm",
    "BootMode": "uefi-preferred",
    "DeprecationTime": "2026-07-06T20:08:37.000Z"
},
```

Some indications that it may be our candidate was that it was created about a month before defcon and the description is "You are about to enter Patala!".

I spun up a ec2 instance in that region from the AMI (`AWS Console -> EC2 -> AMIS -> <search for ami> -> launch from ami`).

Since this is essentially a snapshot of the filesystem at the time, we can look around. The only file that seems to be touched is `/home/policy.txt`:

```plaintext
{
    Version: 2012-10-17,
    Id: Policy1718960569361,
    Statement: [
        {
            Sid: Stmt1718960557932,
            Effect: Allow,
            Principal: *,
            Action: s3:GetObject,
            Resource: arn:aws:s3:::naga-guards-a-secret-chamber/*,
            Condition: {
                StringLike: {
                    aws:UserAgent: SOME MAGIC SPELL GOES HERE TO OPEN THE CHAMBER
                }
            }
        }
    ]
}
```

This appears to be an amazon s3 [bucket policy](https://docs.aws.amazon.com/AmazonS3/latest/userguide/example-bucket-policies.html).

This is interesting because the policy is for bucket `naga-guards-a-secret-chamber` and the website is hosted from bucket `the-naga-dragon-prod`. Websites hosted in s3 follow this syntax: `http://BUCKET/.s3-website.REGION.amazonaws.com/`.

If we replace `BUCKET` and `REGION` with our new bucket (naga-guards-a-secret-chamber) and known region (us-east-2), we can curl it:

```console
curl http://naga-guards-a-secret-chamber.s3-website.us-east-2.amazonaws.com/

<!DOCTYPE html> 
<html lang="en"> 
<head> 
    <meta charset="UTF-8"> 
    <meta name="viewport" content="width=device-width, initial-scale=1.0"> 
    <link rel="stylesheet" href="styles.css"> 
    <title>गुप्त-कक्ष</title> 
</head> 
<body> 
    <div class="background-image"></div> 
    <div class="content">
        <h1>FLAG-{X3Y8qZ7p4LrD2U9oNvW1oK6mBc5A0FsH}</h1>
    </div> 
</body>
```

Which has our flag in it!
