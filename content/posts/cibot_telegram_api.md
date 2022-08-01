---
title: "CIBot: CounterIntelligence Bot, a cheap approach at keeping your home network under control"
date: 2022-06-07T12:00:00+01:00
draft: false
---

I know shouldn't have been but I was genuinely surprised when, upon login onto my tiny home server, I was greeted by:
```
There have been 400-something unauthorized accesses since your last visit.
```

400-something?!? Shouldn't a siren have gone off, red lights flashing in my lab?</br>
Of course no. Not unless I'd set up something like that... who thought anyone would be interested in my minuscule "server"?</br>
No data, nothing to steal... nothing iapparently interesting but CPU cycles and RAM! Which is precisely what [botnets](https://en.wikipedia.org/wiki/Botnet) and black-hat hackers are after.</br>
</br>
Confronted with the question, "should I shut down access or do something else?" I opted for _something else_.</br>
I had nothing to lose, no precious data, a lot to learni on the other hand. What did I learn then?</br>
Firstly, I learned that I had done my homework right &lt;put big smile here&gt;. Seemingly, nobody had been able to gain access to the box. For various reasons.</br>
To help focus on details, here's an example of the log:
```
<timestamp> <hostname> sshd[6690]: Invalid user miguel from 62.234.92.111 port 46484 
<timestamp> <hostname> sshd[6690]: pam_unix(sshd:auth): check pass; user unknown
<timestamp> <hostname> sshd[6690]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=62.234.92.111
<timestamp> <hostname> sshd[6690]: Failed password for invalid user miguel from 62.234.92.111 port 46484 ssh2
<timestamp> <hostname> sshd[6690]: Received disconnect from 62.234.92.111 port 46484:11: Bye Bye [preauth]
<timestamp> <hostname> sshd[6690]: Disconnected from invalid user miguel 62.234.92.111 port 46484 [preauth]
<timestamp> <hostname> unix_chkpwd[6694]: password check failed for user (root)
```
An initial analysis showed an interesting picture. Multiple IPs, from different geographical locations: my server wasn't being targeted by a single hacker. Possibly, it was being probed by a botnet so that it could join the force to build up some super-DDoS-weapon perhaps.
<img src="images/piechart.png">
Luckily, something had gone wrong for the hackers. Here's a non-exhaustive list:
- no obvious usernames, the hackers were attempting to login with `username + password`, my server didn't have any of the names in their list
- in fact, **no password**!!! I had long disabled password-based access on my server(s), the lab's unwritten rule is key-based access only, here's one [article](https://linuxize.com/post/how-to-setup-passwordless-ssh-login/) as an example
- in addition to that, **root** access had obviously been disabled

YMMV, but there are many best practices around that may fit your use case. Hardening SSH isn't difficult and automating it, through Ansible for instance, is easy.</br>
</br>
Still something was missing, namely my ability to detect the attempts and, possibly, do something about them. Enter [Telegram Bot API](https://core.telegram.org/api).</br>
Wait, didn't you just say that bots are bad? Well, no, not exactly, some can be bad, others can be useful. Telegram Bots are special accounts that do not require an additional phone number to set up. These accounts serve as an interface for code running somewhere on your server.</br>
Which code? Well, that's easy, I've put a simple but fully operational example on GitHub: [CIBot](https://github.com/carmelo0x99/CIBot). Feel free to use it and provide comments if any.</br>
How does the **good** bot operate? Fundamentally, it's composed of three parts:
- `cibot.py`
- `cibot.json`
- `cibot.log`

`cibot.py` is the Python script. Its role is to parse `auth.log` and gather info on unauthorized access attempts. The aggregated info is passed onto a function that consumes Telegram API's `sendMessage` endpoint. The overall result is that, ping!, if necessary a message will pop up on the operator's mobile phone.</br>
`cibot.json` contains the configuration information. Mainly, Telegram API's **token** which is the key to get access to the service.</br>
`cibot.log` is an auto-generated log file recording what's been going on when the script has been triggered.</br>
The recommended way is to run the script through `cron`. Bonus points are granted if the Docker image is used, for a cloud-native approach.</br>

