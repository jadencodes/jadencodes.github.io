---
layout: post
title:  "CV CTF - My Bucket Is Yours"
date:   2022-09-01 17:00:00 -0600
categories: ctf cloudvillage
tags: defcon ctf cloudvillage
description: "Writeup/Solution to Defcon 2022 Cloud Village CTF: My bucket is yours (200 points)"
---

## Prompt

The original prompt for challenge 2 `My Bucket Is Yours` was:

>There is some code that was written for this challenge that is floating somewhere on the internet.

So we have narrowed it down to the entire internet.

The key part here is actually the _title_ of the challenge, specifically the word **bucket**.

**NOTE:** After no teams had solved this during the first 12 hours of the competition - likely due to the broadness of the prompt - cloud-village appended and tweaked the initial prompt to include slightly more information. I didn't have the foresight to document what it was changed to but I have the following notes:

- Codebase
- Python

### What we know so far

1. There is likely a script or codebase involved
2. There is likely a storage solution that uses the name **bucket** like `AWS S3` or `Google Storage` involved

Not a lot, but not nothing either. At this point there are two likely scenarios based on the prompt and title: the codebase is stored in a bucket OR, the bucket is referenced in the code base.

### GitHub search

GitHub search can be quite useful if you know what you are doing. Regardless, I was able to narrow down the repositories with the search term [cloud-village](https://github.com/search?l=Python&q=cloud-village&type=Repositories) and selecting `Python` as the primary repository language I was looking for.

This gave me two results:

![Github search results](/assets/images/github-cloud-village-my-bucket.png)

The first one of which was updated a mere _week_ before Defcon 30: [justmorpheus/Cloud-Village-2022](https://github.com/justmorpheus/Cloud-Village-2022)

The only thing in the repository is file `app.py`:

### The file

```python
import boto3
import uuid

#Import variables
#Wrong bucket uploaded
ENV GCP="<variable>"

# boto3 offers two different styles of API - Resource API (high-level) and
# Client API (low-level). Client API maps directly to the underlying RPC-style
# service operations (put_object, delete_object, etc.). Resource API provides
# an object-oriented abstraction on top (object.delete(), object.put()).
#
# While Resource APIs may help simplify your code and feel more intuitive to
# some, others may prefer the explicitness and control over network calls
# offered by Client APIs. For new AWS customers, we recommend getting started
# with Resource APIs, if available for the service being used. At the time of
# writing they're available for Amazon EC2, Amazon S3, Amazon DynamoDB, Amazon
# SQS, Amazon SNS, AWS IAM, Amazon Glacier, AWS OpsWorks, AWS CloudFormation,
# and Amazon CloudWatch. This sample will show both styles.
#
# First, we'll start with Client API for Amazon S3. Let's instantiate a new
# client object. With no parameters or configuration, boto3 will look for
# access keys in these places:
#
#    1. Environment variables (AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY)
#    2. Credentials file (~/.aws/credentials or
#         C:\Users\USER_NAME\.aws\credentials)
#    3. AWS IAM role for Amazon EC2 instance
#       (http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)

s3client = boto3.client('s3')

# Everything uploaded to Amazon S3 must belong to a bucket. These buckets are
# in the global namespace, and must have a unique name.
#
# For more information about bucket name restrictions, see:
# http://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html
bucket_name = 'python-sdk-sample-{}'.format(uuid.uuid4())
print('Creating new bucket with name: {}'.format(bucket_name))
s3client.create_bucket(Bucket=bucket_name)

# Now the bucket is created, and you'll find it in your list of buckets.

list_buckets_resp = s3client.list_buckets()
for bucket in list_buckets_resp['Buckets']:
    if bucket['Name'] == bucket_name:
        print('(Just created) --> {} - there since {}'.format(
            bucket['Name'], bucket['CreationDate']))

# Files in Amazon S3 are called "objects" and are stored in buckets. A
# specific object is referred to by its key (i.e., name) and holds data. Here,
# we create (put) a new object with the key "python_sample_key.txt" and
# content "Hello World!".

object_key = 'python_sample_key.txt'

print('Uploading some data to {} with key: {}'.format(
    bucket_name, object_key))
s3client.put_object(Bucket=bucket_name, Key=object_key, Body=b'Hello World!')

# Using the client, you can generate a pre-signed URL that you can give
# others to securely share the object without making it publicly accessible.
# By default, the generated URL will expire and no longer function after one
# hour. You can change the expiration to be from 1 second to 604800 seconds
# (1 week).

url = s3client.generate_presigned_url(
    'get_object', {'Bucket': bucket_name, 'Key': object_key})
print('\nTry this URL in your browser to download the object:')
print(url)

try:
    input = raw_input
except NameError:
    pass
input("\nPress enter to continue...")

# As we've seen in the create_bucket, list_buckets, and put_object methods,
# Client API requires you to explicitly specify all the input parameters for
# each operation. Most methods in the client class map to a single underlying
# API call to the AWS service - Amazon S3 in our case.
#
# Now that you got the hang of the Client API, let's take a look at Resouce
# API, which provides resource objects that further abstract out the over-the-
# network API calls.
# Here, we'll instantiate and use 'bucket' or 'object' objects.

print('\nNow using Resource API')
# First, create the service resource object
s3resource = boto3.resource('s3')
# Now, the bucket object
bucket = s3resource.Bucket(bucket_name)
# Then, the object object
obj = bucket.Object(object_key)
print('Bucket name: {}'.format(bucket.name))
print('Object key: {}'.format(obj.key))
print('Object content length: {}'.format(obj.content_length))
print('Object body: {}'.format(obj.get()['Body'].read()))
print('Object last modified: {}'.format(obj.last_modified))

# Buckets cannot be deleted unless they're empty. Let's keep using the
# Resource API to delete everything. Here, we'll utilize the collection
# 'objects' and its batch action 'delete'. Batch actions return a list
# of responses, because boto3 may have to take multiple actions iteratively to
# complete the action.

print('\nDeleting all objects in bucket {}.'.format(bucket_name))
delete_responses = bucket.objects.delete()
for delete_response in delete_responses:
    for deleted in delete_response['Deleted']:
        print('\t Deleted: {}'.format(deleted['Key']))

# Now that the bucket is empty, let's delete the bucket.


print('\nDeleting the bucket.')
bucket.delete()
```

This script uses the [boto3 library](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html) which is _generally_ used to communicate with AWS services like S3. However, it _can_ be [configured](https://gist.github.com/gleicon/2b8acb9f9c0f22753eaac227ff997b34) to use other storages like google storage. This file appears to be a practice of the `s3client`, doing the following:

- Create a random bucket name with the prefix `python-sdk-sample-`
- List the buckets available
- Uploads a file to the previously created bucket
- Generates and prints out a presigned URL
- Finally deletes all the data in the created bucket and finally the bucket itself

This is all fine and dandy, but these two lines are suspicious:

```python
#Wrong bucket uploaded
ENV GCP="<variable>"
```

The comment indicates a known mistake in the code, but more importantly the variable `GCP` likely stands for "Google Cloud Platform". Not to mention, `ENV` is not valid python... This indicates that we should expect to look in Google Storage buckets for the flag.

### Git history

We can open the commit history in GitHub and see two commits:

![my buckets commit history](/assets/images/my-bucket-commit-history.png)

The earliest of the two commits initially adds the following line:

```python
ENV GCP="cloud-village-2022/"
```

This presumably simulates a common mistake of hard coding an environment variable meant to be added during a deploy.

### Putting it all together

Now that we have the bucket name and the storage type, we can use [gsutil](https://cloud.google.com/storage/docs/gsutil) to look for buckets.

```console
$ gsutil cat gs://cloud-village-2022/flag.txt
FLAG-{eqEl4wEAbx69uyp4v7nAFgyj9bDztzv9}
```

The flag is `FLAG-{eqEl4wEAbx69uyp4v7nAFgyj9bDztzv9}`!
