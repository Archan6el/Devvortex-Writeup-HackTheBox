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

Before fuzzing the log in, I first google to see if there are any exploits for joomla, and I do find an exploit on exploit-db at (https://www.exploit-db.com/exploits/51334)

![image](https://github.com/Archan6el/Devvortex-Writeup/assets/91164464/4f16fb3b-9e1a-426f-ade7-f67da3be428a)

Its an exploit written in ruby. I create a file, `joomla-exploit`, paste the code in, and run it. The file requires one argument, which is the URL of the website. The exploit works, and provides me with information about the database used, but more importantly, log in credentials:

![image](https://github.com/Archan6el/Devvortex-Writeup/assets/91164464/ea6d751c-c497-496d-8c46-2bb8253348f7)

I use these crednetials on the log in page found earlier, and I am logged in:

![image](https://github.com/Archan6el/Devvortex-Writeup/assets/91164464/3beb8247-b2df-49a5-a490-1daf2199be42)

