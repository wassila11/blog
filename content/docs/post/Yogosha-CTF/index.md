---
title: Yogosha CTF – Uchiha or Evil ? Web Challenge Write-up (1000 pts)

# Summary for listings and search engines
summary: Yogosha Chritmas CTF Write-up / First Web Challenge 
# Link this post with a project
# projects: []

# Date published
date: "2021-01-01T00:00:00+01:00"

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
- Hash length extension
- web exploitation
- CTF write-up

categories:
- Web
- CTF
---

# Yogosha CTF -- Uchiha or Evil ? Web Challenge Write-up (1000 pts)

Did you have one of those days where you must prepare for your exams, yet an interesting ctf challenge is up and you can't help yourself dedicating your focus and your time playing.. ?
Last weekend was the announcement of Yogosha Christmas Challenge, more than 300 players joined. I ranked 15th so I thought about sharing a write-up.
It was basically a scenario based on the story of saving Konoha Village ( The author was a real Naruto fan ).
The welcome challenge was an OSINT challenge, a very amusing one where you just need to find the user account on flicker, download the photo he shared then read its meta-data.
The next challenge which was a lot more interesting had 3 steps in order to get the flag.
## The First Step:
When reading the meta-data of the previous challenge, I found the flag along with the URL that took me to the next challenge.
	![alt text](https://i.imgur.com/YPqE4zx.png)

When entering the website a simple web page is loaded, I looked at the source code but I found nothing interesting, there were no user input so the next thing I thought about was adding /robots.txt to the url.
First of all let me explain why I did so.
In fact robots.txt is included in the source files of most websites, it contains the rules destined for bots to follow, lets say the "code of conduct" of that website.
To learn more about it you can visit:
https://www.cloudflare.com/learning/bots/what-is-robots.txt/ 
and here we go !
		![alt text](https://i.imgur.com/YGH9TsJ.png) 

User-agent is one of the headers sent in a web request, simple, all we have to do is to intercept the request and change the User-agent to Uchiha in order to access to the file /read.php 
I did that using Burp-suite.
![alt text](https://i.imgur.com/VBhxQxZ.png)

When forwarding the request an another webpage is loaded.. On to the next step !! 

## The Second Step:
I took a look at the source code and I noticed a default value set to be transfered in a post request, the hint contained the word state hash so when I googled it I found that hash-length extention attack is what I need to solve this task.
Hash-length extention attack takes advantage of the weekness of MAC hashes (Message authentication code) which apply a hash function at the concatination of a secret key and a message H(secret ‖ message). The idea here is to abtain a valid hash without knowing the secret key and this is done by using Hash(message) and the length of the secret key to calculate Hash(message ‖ AddedData).
So I searched for a tool to get this done and found hash_extender. 
Here's the Github link: https://github.com/iagox86/hash_extender 
Since I don't know the length of the secret key, I thought about brute forcing it. Indeed, I saved the output of the execution of the hash_extender tool in a file and wrote a python script that sends a post request each time with a different secret key length. 
Here's the php code where we can find how the Post request is being processed, so the filenames are separated with ":" that's why the data we need to add to read the guinjutsu.php file is ":guinjutsu.php". The hash algorith used is sha256.
![alt text](https://i.imgur.com/T8cqa8x.png) 
**The command to execute :**
``` hash_extender => sudo ./hash_extender --data 'read.php' --secret-min=1 --secret-max=50 --append ':guinjutsu.php' --signature 184b5d255817fc0afe9316e67c8f386506a3b28b470c94f47583b76c7c0ec1e5 --format sha256 > newhash.txt ```


**The script:**  
``` 
import requests
import urllib 
url = "http://3.141.159.106/read.php"
contenue_fichier = open("newhash.txt").read() 
split1 = contenue_fichier.split("Type: sha256")

for char in split1[1::]:
        split2 = char.split("\n") 
        #print(split2)  
        sig = split2[2].split(" ")[2]
        sec = split2[3].split(" ")[2]
        payload = sig + "|" + sec.decode("hex")
        r=requests.post(url,data={"string": payload}, headers={"User-Agent":"Uchiha"})
        if "Verification Failed! You didn't awaken your sharingan!" not in  r.text :
                print(urllib.quote_plus(payload)) 
``` 



and there we go ! 
![alt text](https://i.imgur.com/EKoGvvs.png) 

So I sent the request with burpsuit using this parameter, I took a look at the response and an another php code that takes us to the final step is found.
![alt text](https://i.imgur.com/5cNdsZD.png) 
## The Third Step:
Looking at this php code, we can understand the behaviour of the website in order to read the file secret.txt.
![alt text](https://i.imgur.com/oqHE5lZ.png) 
We notice that it uses a function parse_url(). Thus, this function breaks the url up into various components.  
A simple check I did to understand more the behaviour of this function and the way we should send our request was the command php -a.
![alt text](https://i.imgur.com/DKxlQCI.png)


Next I took a look at the function ```strpos($par['scheme'],'http')``` which is a kind of a filter that require the parameter scheme to have http in it.
For more infos about strpos() you can check:
https://www.php.net/manual/en/function.strpos.php 

Yet a problem is that the function file_get_contents() must receive a file in order to be able to read secret.txt.
The solution here is that file_get_contents() reads the file content everytime it receives an unknown scheme. So all we need to do is to to "fool" the function strpos to return true then pass an unknown scheme to file_get_contents().

And now the final touch is to pass the parameters in a POST request in a sort of a way that check_url() returns true.
![alt text](https://i.imgur.com/qFVYUQB.png) 

## Conclusion
Arriving at the end of this article, I hope you enjoyed it and I enormously thank you for reading all of it, I really appreciate that actually since this is my first CTF Write-up and the first time I post something in my blog. 
Yogosha christmas CTF was really fun and a good one to start with :D 
Unfortunately I couldn’t fully participate due to my exams :( Yet it was really fruitful, interesting and I learnt so much from it. 
If you have any questions please don't hesitate to contact me on twitter, facebook or by email, I’ll be very glad to help.
