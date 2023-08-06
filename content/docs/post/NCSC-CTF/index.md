---
title: NCSC 3rd Edition CTF - Web Challenges Write-up
# Summary for listings and search engines
summary: NCSC CTF Write-up / Web Challenges 
# Link this post with a project
# projects: []

# Date published
date: "2022-02-28T00:00:00+01:00"

# Date updated
#lastmod: "2020-12-13T00:00:00Z"

# Is this an unpublished draft?
#draft: false

# Show this page in the Featured widget?
featured: false

# Featured image
# Place an image named `featured.jpg/png` in this page's folder and customize its options here.
 
 
 # placement: 2
  #preview_only: false

authors:
- Wassila Chtioui

tags:
- PHP exploitation
- web exploitation
- CTF write-up

categories:
- Web
- CTF
---

# NCSC 3rd Edition CTF
## _Web Challenges Write-up_

### Introduction 
And here we are finally having the long awaited National CyberSecurity Congress in its 3rd edition. And of course I didn't miss the chance to attend. 
So it's basically a 3 days event organized every year by Securinets Club INSAT. And there's always a CTF in the second day.
I decided to join, thus here's a write up of the 3 web challenges I solved.

### Welcome To Web Universe -  812 Points - 17 Solves 
In this task we were given the source code and this url page.

![alt text](https://i.imgur.com/pLxVTpd.png)

Since we have the source code the first thing to do was to check it.
We find a directory named nginx, where there is a file nginx.conf
``` 
server {
listen 80;
server_name welcome.task;

location /leet {
  proxy_pass http://api:5000/v1/;
}

access_log off;
error_log /var/log/nginx/error.log error;
}
```
Before digging any deeper it's important to read some articles about nginx in order to understand the configuration file we found.
This [link](https://docs.nginx.com/nginx/admin-guide/web-server/web-server/?_bt=573371260706&_bk=&_bm=&_bn=g&_bg=134448916480&gclid=EAIaIQobChMIj-SWzoDY9QIV65JmAh2gdQcYEAAYASAAEgJ6MvD_BwE) was very helpful actually.

So the location directive in our code indicates that having /leet proxy the request to this url http://api:5000/v1/ 

All we have to do now is to dig more in the souce code in order to understand better the location of the flag.

the directory src contains a python file main.py that contains the following..

```
from flask import Flask
import os

flag=os.getenv("FLAG")
app = Flask(__name__, static_url_path='/static/')
@app.route("/v1/status")
def index():
        return "Everything is good afaik"

@app.route("/flag")
def flag():
        return flag

if __name__ == '__main__':
        app.run()
``` 

We notice that /v1/status prints the page we have currently in the web site, and we find that our goal is to access /flag to get our flag.
Yet, /leet takes us to the directory /v1 which is not what we need. /flag is in the previous directory. the trick is simple here, all we need to do is to use ../ to move in to the previous directory.

This is the payload I used: http://20.119.58.135:4567/leet../flag 

### Broken Pingyy  -  645 Points - 23 Solves 

###### `` I've developed a simple web app to ping any IP/domain but i had some problems in my code .. I'm a newbie web developer so i'll give the source code to help me. flag location => /flag Link: http://20.119.58.135:789/ `` 

![alt text](https://i.imgur.com/lcOkrB6.png)

In this task, the first thing I did was to enter the url and try to ping some ip addresses, yet, there's always an error and none of the domains or ips I tried worked. In this case since we have the source code I must take a look at it, maybe I find something that will help me figure out where this error came from.

```
<!DOCTYPE html>
<html>
<head>
        <title>PINGYY</title>
        <link rel="stylesheet" type="text/css" href="styles.css">
        <link href="https://stackpath.bootstrapcdn.com/bootstrap/5.0.0-alpha1/css/bootstrap.min.css" rel="stylesheet">
        <script type="text/javascript" src="https://stackpath.bootstrapcdn.com/bootstrap/5.0.0-alpha1/js/bootstrap.min.js"></script>
</head>
<body>
<div class="container">

        <div class="row height d-flex justify-content-center align-items-center">
                <div class="col-md-8">
                        <center><h1>This service is currently under maintenance !!</h1></center>
                <div class="search"> <i class="fa fa-search"></i>
                        <form method="POST" action="index.php" >
                                <input type="text" class="form-control" name ="ip" placeholder="www.google.com">
                        <button class="btn btn-primary">Ping</button>
                        </form>
                </div>
                <?php

                                        if(isset($_POST['ip'])){
                                                $cmd=$_POST['ip'];
                                                $clean=escapeshellarg($cmd);
                                        if($output = shell_exec("bash -c 'ping -c1 -w3 $clean" ))
                                                echo "<pre>$output</pre>";
                                        else
                                                echo "an error has occurred !";


                }
                                ?>
                </div>

        </div>


</div>
</body>
</html>       
``` 
So the line that interests us most is: 
``  $output = shell_exec("bash -c 'ping -c1 -w3 $clean" ) `` 

The fist thing I noticed was the ' that was missing before the argument (our input), just to make sure I went back to the web site again and tried to ping a random ip address but this time with adding a ' before the ip and yuup it did work.

The next step was to figure out how to exploit the function shell_exec in order to perform an RCE. 
So I tried to execute the command ls, the idea was to end the previous command with ; and indeed it worked perfectly. 

![alt text](https://i.imgur.com/hdUKBIH.png)

We know that the flag is in /flag, so all we need to do is to enter the command `` cat /flag `` and get our flag..
![alt text](https://i.imgur.com/vAgQpXs.png)

### Weird Php - 876 Points - 14 Solves

In this task we were only given the following php code:
``` 
<?php

highlight_file(__FILE__);
include ("secret.php");

if (isset($_GET['a'])&&isset($_GET['b'])&&isset($_GET['c'])) {
    $array = json_decode($_GET['a']);
    if ($array->key == SECRET) {
        if ( ($_GET['b'] !== $_GET['c']) && (md5($_GET['b']) === md5($_GET['c']) ) ) {
            echo FLAG;
        }else {
            echo "👍👍👍👍👍👍👍👍👍";
        }
    } else {
        echo "Try harder bb :)";
    }
}

?> 
``` 

So we need to enter three GET parameters a, b and c that respect some rules indicated in the code above.
I followed 2 steps in order to get the flag.
The first step which is the first condition: `` $array->key == SECRET `` : 
a is a json that has a key named 'key' and which value corresponds to SECRET which is not written between quotes " " so when entering the GET parameter as a string it will not match, the idea here is to use type juggling, which is to simply enter an integer. 

So here's the trick, the concept is that in order to compare two values with different types, one of them is converted to the type of the other one. 
For a in depth understanding here's a great article:
https://medium.com/swlh/php-type-juggling-vulnerabilities-3e28c4ed5c09 
So we set the value of the key parameter to 0.

2nd condition: 
Concering b and c, we have === ; it's a strict comparaison but the values shouldn't be the same.
Knowing that we can't calculate the hash of an array, all we need to do is to set b and c as arrays.
and here's our payload: `` http://X.X.X.X:276/?a={"key":0}&b[]=hh&c[]=yy ``

![alt text](https://i.imgur.com/INAm90a.png)

### Conclusion
All in all, the event was just incredible, I enjoyed the different activities that took place such us the conferences and the workshops. The challenges were also very interesting that's why I highly recommand that if ever there was a 4th edition you must take part of it.  

