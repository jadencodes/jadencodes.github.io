---
layout: post
title:  "CV CTF - Mozilla Matrix"
date:   2024-08-13 11:52:27 -0600
categories: ctf cloudvillage
tags: defcon ctf cloudvillage
description: "Writeup/Solution to Defcon 2024 Cloud Village CTF: Mozilla Matrix"
---

## Prompt

>A legendary artifact known as "The Mozilla Matrix" has resurfaced after millions of years of safeguarding by the Ocean Protectors concealed within a container surrounded by a protective aura. Sorcerers from different galaxies have been summoned to retrieve the secrets within the Matrix's inner sanctum. Do you have what it takes to become the greatest sorcerer to exist in the universe?

We are given a zip file named `Mozilla_Matrix.zip`

### Clues

The name of the challenge gives us the major hint on where to start. "Mozilla", the creators best known for Firefox.

## Solution

Unzipping the `Mozilla_Matrix.zip` file gives us a gibberish directory `sv7i9wlm.MozillaMatrix` with lots of files in it. The ones that stand out are sqlite files such as:

```console
$ tree sv7i9wlm.MozillaMatrix/ | grep sqlite$
├── bounce-tracking-protection.sqlite
├── content-prefs.sqlite
├── cookies.sqlite
├── domain_to_categories.sqlite
├── favicons.sqlite
├── formhistory.sqlite
├── permissions.sqlite
├── places.sqlite
├── protections.sqlite
├── storage.sqlite
├── suggest.sqlite
├── webappsstore.sqlite
```

We can open these one by one and look for data. They included things such as places visited and the one that stuck out was visiting the mozilla password store. This is the built it system that stores your passwords if you click "save" after logging into a website so next time you visit they are remembered.

The passwords were also in one of the sqlites, but they are **encrypted**.

However, there is a tool to crack them called [firefox_decrypt](https://github.com/unode/firefox_decrypt).

We downloaded and ran this tool against the profile:

```console
$ python3 firefox_decrypt.py sv7i9wlm.MozillaMatrix/

Username: 'path: /TlF8KNeH6d username: f34e60ec-bdcd-4821-9f5b-902db18db2b0'
Password: '1UA4>C41eE@M=_4n4U:.sL'
```
There were lots of json files for storing data as well, again after some poking around we found an interesting one `logins.json`:

```json
{
  "nextId": 6,
  "logins": [
    {
      "id": 5,
      "hostname": "https://mozillamatrix.dragons-in-the.cloud",
      "httpRealm": null,
      "formSubmitURL": "",
      "usernameField": "",
      "passwordField": "",
      "encryptedUsername": "MHIEEPgAAAAAAAAAAAAAAAAAAAEwFAYIKoZIhvcNAwcECKURO2ywGqkWBEhCvChZj5Rt65g69RF1Qd/91NHtCtxhR+swJ6KcGBHfGOzXlq+CylSR6pvPo5Se8lpjAArR2yMzxp9siASYrunW1mXabYVPPuA=",
      "encryptedPassword": "MEIEEPgAAAAAAAAAAAAAAAAAAAEwFAYIKoZIhvcNAwcECAtKGEAlTubCBBidkBCkEJ3BX9K+cRTzYhrmBpodGwRefHw=",
      "guid": "{071cdb86-4be5-40b0-9a8e-e9d29f9d2715}",
      "encType": 1,
      "timeCreated": 1719734759912,
      "timeLastUsed": 1719734759912,
      "timePasswordChanged": 1720149199020,
      "timesUsed": 1,
      "syncCounter": 3,
      "everSynced": false,
      "encryptedUnknownFields": null
    }
  ],
  "potentiallyVulnerablePasswords": [],
  "dismissedBreachAlertsByLoginGUID": {},
  "version": 3
}
```

So now we had a username, password, and url.

Going to the url was an error, "Method not allowed", meaning that `GET` wasn't valid. After iteratting over the other methods we found that `POST` was valid but the path wasn't.

We then included the `path: /TlF8KNeH6d` data in the request:

```console
curl -X POST http://mozillamatrix.dragons-in-the.cloud/TlF8KNeH6d

Access Denied
```

We had the correct url but were missing some sort of credential. We then tried lots of things such as using basic auth headers both with and without base64 encoding:

```console
curl -X POST -H "Authorization: Bearer f34e60ec-bdcd-4821-9f5b-902db18db2b0:1UA4>C41eE@M=_4n4U:.sL" http://mozillamatrix.dragons-in-the.cloud/TlF8KNeH6d

Access Denied
```

Finally, we tried something stupid. Since headers can be _anything_ we tried just setting the non standard headers of `username` and `password`:

```console
curl -X POST -H "username: f34e60ec-bdcd-4821-9f5b-902db18db2b0" -H "password: 1UA4>C41eE@M=_4n4U:.sL" http://mozillamatrix.dragons-in-the.cloud/TlF8KNeH6d

FLAG-{}
```
