# g/re/p

### grep without regex

By default grep command would work with regex

***example***

```
$ cat hosts
127.0.0.1       localhost
# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

On grep command The above host file would return the below output

```
$ grep 127.0.0.1 hosts
127.0.0.1       localhost
```

But since the grep uses regular expression by default this command would fail on below condition.

```
$ echo 127x0x0x1 >> hosts
$ grep 127.0.0.1 hosts
127.0.0.1       localhost
127x0x0x1
```

we are adding another similar syntax replacing dots(.) with 'x'. Now since dot is anything in regex grep would return both lines

To resolve this issue we could use the below command to run the grep without regular expression

```
$ grep -F 127.0.0.1 hosts
127.0.0.1       localhost
```

### grep vs fgrep

`grep` uses BRE (Basic Regular Expression) <br>
`grep -F` is called as Fast or Fixed grep
`grep -E` is ERE (Extended Regular Expression)

There is another command line utility `fgrep` which achieve the same as `grep -F`, but to dig it a little more we could find both are same.

```
$ type -a fgrep
fgrep is aliased to `fgrep --color=auto'
fgrep is /usr/bin/fgrep
fgrep is /bin/fgrep
$ cat /usr/bin/fgrep
#!/bin/sh
exec grep -F "$@"
```

`egrep` likewise, 

```
$ type -a egrep
egrep is aliased to `egrep --color=auto'
egrep is /usr/bin/egrep
egrep is /bin/egrep
$ cat /usr/bin/egrep
#!/bin/sh
exec grep -E "$@"
$
```

### Invert search, to select non matching lines

`grep -v` to select non matching lines. (i.e,) Inverting the sense of matching to show the lines other than the specified ones

***example*** <br>
From the below file remove the commented lines (comments starts with '#' pound sign)

```
$ cat services
# Network services, Internet style
#
# Updated from https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml .
#
# New ports will be added on request if they have been officially assigned
# by IANA and used in the real-world or are needed by a debian package.
# If you need a huge list of used numbers please install the nmap package.

tcpmux          1/tcp                           # TCP port service multiplexer
echo            7/tcp
```

To remove the commented lines, which starts with the pound sign (#) we could use,

```
$ grep -v '^#' services

tcpmux          1/tcp                           # TCP port service multiplexer
echo            7/tcp
$
```

But, this would just eliminate the lines starts with # however we could see a blankline. 

### Eliminate the blank lines along with commented lines

```
$ grep -ve '^#' -ve '^$' services
tcpmux          1/tcp                           # TCP port service multiplexer
echo            7/tcp
```

Here, `e` is to match the regex pattern. To remove the comments in start of the line we could use `^#` and to remove empty line we could use `^$` ('^' symbol represent start of the line, '$' represents the empty line or end of the line)

But this might look complicated. This could be simplify with regex groups

### Regex grouping

```
$ grep -Ev '^(#|$)' services
tcpmux          1/tcp                           # TCP port service multiplexer
echo            7/tcp
```

`-E` to use ERE for more regex usage. `()` would be called grouping the regex. `^(#|$)` In this expression, we are stating group of '#' or '$'. 
The expression matches if the line starts with '#' or blankline.

### Few more regex characters

Below are few regex characters could be used with `grep -E`

```
. - Anything
? - Optinal previous character 
* - Any number of character
```

**Any character - .**

Dot (.) is just to represent with anything in the expression. 

***example*** <br>
We want to match the words which start with 'h' and ends with 't'. The could be any character single in the middle

```
$ cat file1
hat
hut
hot
halt
$ grep -E 'h.t' file1
hat
hut
hot
```

If we want to match any two characters in the middle we could use two dots (h..t)

```
$ grep -E 'h..t' file1
halt
```

**Optional preceeding character - ?**

The '?' would match for string with optional character.

***example*** <br>
```
$ cat file1
colour
color
one
two
```
In the above file we want to filter the text 'colour'. But both 'colour' and 'color' are valid words. We could use the below approach

```
$ grep -E 'colou?r' file1
colour
color
```

In the expression we used 'colou?r'. which makes the character berfore '?' an optional one. Thus it matches both the cases.

**0 or more occurance of preceeding character - \***

The * (astrick) character checks for any number of occurences of previous character.

***example*** <br>

The below is the file which has multiple comments but not all of them are aligned properly. few with tab in the beginning and few with space. 

```
$ cat file1
# Comment
        # Comment
   #Comment
File contents
```
If we try our previous examples, It would return only the first character starts with # (pound) sign. which ignores spaces before comments

```
$ grep -E '^#' file1
# Comment
```

The below expression would look for any line starts with 0 or any number of space (\s escape character for space) followed by # sign.

```
$ grep -E '^\s*#' file1
# Comment
        # Comment
   #Comment
```


### To match the whole word `-w`

Imagine we have the below file

```
$ cat file1
rate
rated
rat
elaborate

```

If we match with the text 'rat' it would return all the lines which contains 'rat' in beginning, middle and end of the word.

```
$ grep 'rat' file1
rate
rated
rat
elaborate
```

But our goal is just to get line contain the whole word 'rat'. we could use `-w` to achieve that

```
$ grep -w 'rat' file1
rat
```

### To show only the matches `-o`

Sometimes we just want to view only the word which matches the expression.

```
$ cat hosts
127.0.0.1       localhost
# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

The above file have ip address. we just need to view only the ip address which matches '127.0.0.1' not the whole line. `-o` displays only the matched word not the entire line

```
$ grep -F -o '127.0.0.1' hosts
127.0.0.1
```

### Ignore case `-i`

Some files have mixed camel case characters which wont be displayed with normal pattern. 'camelCase' is not same as 'camelcase'. to search by ignoring the case we could use `-i`

***example***<br>
```
$ cat file1
rate
rated
RAT
elaborate
$ grep -w 'rat' file1
$
```

Since 'rat' is not same as 'RAT' the above expression failed to retrieve the desired word. we can ignore the case by '-i' attribute

```
$ grep -w -i 'rat' file1
RAT
```

### show few lines of After the match

Some cases the log files are very large where you might miss few details after the match. In such cases we can use -A followed by number of lines to display after-context.

***example***

/var/log/apt/history.log is log file keeps the package installation and removal details as history in debian based distro. The file looks like below,

```
$ cat /var/log/apt/history.log

Start-Date: 2022-07-11  10:39:59
Commandline: apt-get install apt-transport-https
Requested-By: sundar (1000)
Install: apt-transport-https:amd64 (2.4.5)
End-Date: 2022-07-11  10:40:00

Start-Date: 2022-07-11  10:41:11
Commandline: apt-get install dart
Requested-By: sundar (1000)
Install: dart:amd64 (2.17.5-1)
End-Date: 2022-07-11  10:41:22
```

In the above log I need to find the package name 'dart' and who requested to install/uninstall it (Requested-By line has the userid of requestor).

If we use traditional grep command, we just see the text matches not the 'Requested-By' line. The requested by line is just after the context 'dart'.  

```
$ grep -E 'dart' /var/log/apt/history.log
Commandline: apt-get install dart
Install: dart:amd64 (2.17.5-1)
```

By using -A1 we could view the next line of the context

```
$ grep -EA1 'dart' /var/log/apt/history.log
Commandline: apt-get install dart
Requested-By: sundar (1000)
Install: dart:amd64 (2.17.5-1)
End-Date: 2022-07-11  10:41:22
```


### show few lines of Before the match

With the same example in above section, we need to view the start date of the install/uninstall. The line just before the context we could use -B1.

```
$ grep -EB1 'dart' /var/log/apt/history.log
Start-Date: 2022-07-11  10:41:11
Commandline: apt-get install dart
Requested-By: sundar (1000)
Install: dart:amd64 (2.17.5-1)
```

### Lines both Before and After the context

Let's take the same example for this section as well. Suppose if we want the install/uninstall history of particular user with the full block of text (all the five lines). The user id is in 3rd line there are 2 lines before and 2 lines after. we could display both with the context -C2 in grep command. 


```
$ grep -C2 'sundar' /var/log/apt/history.log
Start-Date: 2022-07-11  10:39:59
Commandline: apt-get install apt-transport-https
Requested-By: sundar (1000)
Install: apt-transport-https:amd64 (2.4.5)
End-Date: 2022-07-11  10:40:00
--
Start-Date: 2022-07-11  10:41:11
Commandline: apt-get install dart
Requested-By: sundar (1000)
Install: dart:amd64 (2.17.5-1)
End-Date: 2022-07-11  10:41:22
$
```
Here, every block is separated and has all the installation request by user 'sundar'


### Finding the ports in use with regex

In linux `/etc/services` file contains the information about the services that client application might use on the computer. The file contains service name, port number, Protcol it uses. 

Our assignment is to identify the services running in 4 digit port number using tcp protocol. 

```
$ grep -E '\s[0-9]{4,4}/tcp' /etc/services
socks           1080/tcp                        # socks proxy server
proofd          1093/tcp
rootd           1094/tcp
openvpn         1194/tcp
rmiregistry     1099/tcp                        # Java RMI Registry
lotusnote       1352/tcp        lotusnotes      # Lotus Note
ms-sql-s        1433/tcp                        # Microsoft SQL Server
ingreslock      1524/tcp
datametrics     1645/tcp        old-radius
sa-msg-port     1646/tcp        old-radacct
kermit          1649/tcp
groupwise       1677/tcp
radius          1812/tcp
radius-acct     1813/tcp        radacct         # Radius Accounting
cisco-sccp      2000/tcp                        # Cisco SCCP
nfs             2049/tcp                        # Network File System
```

Let's breakdown the expression further. <br>

- \s - space before the port number. Without this we will get all the ports more than the range.
- [0-9] - Regular Expression to specify any number from 0-9
- {4,4} - {min,max} should have minimum 4 and maximum 4 numbers. since we wanted to specify only 4 digit ports

### Filtering words ends with digit

To filter for words ends with any digit in the file.

```$ grep -E '[0-9]$' words
A3
High5
We3
me2
you2
```

### Find the words starts with 'm' ends with 'e' with 'r' in the 3rd position

Now that we saw multiple regex example, lets try this exercise. To work on that, we could use `/usr/share/dict/words` which is a word file commonly found in many distribution.

```
$ grep -E '^m.r..e$' /usr/share/dict/words
marble
marine
mirage
morale
morgue
morose
myrtle
```

### Filter all the IP's from files in a directory

Below is the list of files with IP addresses.

```
$ ls -ltr IPAddr/
total 8
-rw-r--r-- 1 sundar sundar   82 Jul 30 20:22 hosts
-rw-r--r-- 1 sundar sundar 1123 Jul 30 20:23 hostfile
```

```$ grep -E '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' IPAddr/*
IPAddr/hostfile:   IPv4 Address. . . . . . . . . . . : 192.168.29.228
IPAddr/hostfile:   Subnet Mask . . . . . . . . . . . : 255.255.255.0
IPAddr/hostfile:                                       192.168.29.1
IPAddr/hostfile:   IPv4 Address. . . . . . . . . . . : 172.18.0.1
IPAddr/hostfile:   Subnet Mask . . . . . . . . . . . : 255.255.240.0
IPAddr/hosts:127.0.0.1 localhost
IPAddr/hosts:127.0.0.1 view-localhost
IPAddr/hosts:127.0.0.1 kubernetes.docker.internal
```

Let's break down things, we are creating four segments of `[0-9]{1,3}\.` for every octet. <br>

- [0-9]{1,3} - Any number with 1 to 3 digits
- \. - escaping character '.'
- IPAddr/* - Passing all the files within the directory as argument

The output would return the filename and the filtered line with IP address. You can use `-o` to just find get the IP addresses along with filename like below

```
$ grep -Eo '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' IPAddr/*
IPAddr/hostfile:192.168.29.228
IPAddr/hostfile:255.255.255.0
IPAddr/hostfile:192.168.29.1
IPAddr/hostfile:172.18.0.1
IPAddr/hostfile:255.255.240.0
IPAddr/hosts:127.0.0.1
IPAddr/hosts:127.0.0.1
IPAddr/hosts:127.0.0.1
```

### Email id filter from large file

There is a html page source file (page_source), we just need to extract the email addresses from them. Let's use the knowledge gained from this section and create a regex for this activity.

For that we need to analyse the possibilities of email id patterns. 

***Example***
```
firstname.lastname@domain.com
firstname_789@domain.com
firstname-lastname.1993@domain.com
```
From the above examples, email id has three sections

- First section with any number of alphabets and symbols like . (dot), - (hyphen), _ (underscore)
- Second section separated with '@'
- Second section 'domain' could be any number of alphabets
- Third section separated with dot (.)
- Third section has any alphabets with 2 to 3 characters (com, net, in, uk)

Below would be the regular expression to filter email ids from any file

```
$ grep -Eo '[a-Z0-9._-]*[-.]*\@[a-z]*\.[a-Z]{2,3}' page_source
samantha@samanthabrown.com
samantha@gmail.com
Jane.Smith@outlook.com
saritaagarwalgoyal@gmail.com
MayurDikShit@example.com
s@example.com
Google@example.com
president@yourdomain.com
party@college.edu
ironman@timgarage.com
jamesbond@xyzdetectiveagency.com
you@yourdomain.com
press@yourdomain.com
suggestions@yourdomain.com
CEO@yourdomain.com
services@yourdomain.com
info@yourdomain.com
biz@yourdomain.com
hi@example.com
editor@example.com
HR@example.com
hello@yourdomain.com
growth@yourdomain.com
visibility@example.com
samantha@samanthabrown.com
samantha@gmail.com
Hi@myname.com
Hi@myname.com
Hello@myname.com
Hello@myname.com
Me@myname.com
Me@myname.com
yourfirstname@yourdomain.com
yourfirstname@yourdomain.com
HR@yourdomain.com
HR@yourdomain.com
```

### Few useful grep attributes

- `-c` - Match count
- `-n` - output with line number