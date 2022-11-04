# SEEDlabs: Race Condition Vulnerability Lab

#### Ozgur Ural
#### Student ID: 2564455


## 2 Environment Setup

### 2.1 Turning Off Countermeasures
You can achieve that using the following command:

```sh
$ sudo sysctl -w fs.protected_symlinks=0
$ sudo sysctl fs.protected_regular=0
```

### 2.2 A Vulnerable Program
The following program is a seemingly harmless program. It contains a race-condition vulnerability.

```sh
/* vulp.c */
#include <stdio.h>
#include <unistd.h>
int main()
{ 
  char *fn = "/tmp/XYZ";
  char buffer[60];FILE *fp;
  /* get user input */
  scanf("%50s", buffer );
  if (!access(fn, W_OK))
  {
    fp = fopen(fn, "a+");
    fwrite("\n", sizeof(char), 1, fp);
    fwrite(buffer, sizeof(char), strlen(buffer), fp); fclose(fp);
  }
  else printf("No permission \n");
}
```

Prepare the program:
```sh
gcc -o vuln vuln.c
sudo chown root vuln
sudo chmod 4755 vuln
```


### 3 Task 1: Choosing Our Target
```sh
# x means the password is stored in /etc/shadow
root:x:0:0:root:/root:/bin/bash
```
We will add a root user in /etc/passwd with the help of this vulnable SUID program.

There is a magic value used in Ubuntu live CD for a password-less account, and the magic value is U6aMy0wojraho. If we put this value in the password field of a user entry, we only need to hit the return key when prompted for a password.

Let’s add one entry to /etc/passwd as below:
```sh
rambo:U6aMy0wojraho:0:0:test:/root:/bin/bash
```

```sh
[10/12/22]seed@VM:~/race-cond-lab$ su rambo
Password:
root@VM:/home/seed/race-cond-lab# id
uid=0(root) gid=0(root) groups=0(root)
```


### 4 Task 2: Launching the Race Condition Attack

#### 4.1 Task 2.A: Simulating a Slow Machine
In this task, I used a symbolic link to change the meaning of /tmp/XYZ because I could notchange it simply because it is hardcoded into the program.

```sh
[10/12/22]seed@VM:~$ ln -sf /dev/null /tmp/XYZ
[10/12/22]seed@VM:~$ ls -ld /tmp/XYZ
lrwxrwxrwx 1 seed seed 12 Oct 22 22:20 /tmp/XYZ -> /dev/null
```sh


#### 4.2 Task 2. B: The Real Attack
In this task, after the script terminated, I logged in as a test user and it had root privileges.XYZ file had root privileges.


#### 4.3 Task 2.C: An Improved Attack Method
This is mainly to make up for the experiment Task2.A The flaw in
Principle analysis ：
stay Tasks2.A Never succeed in , This is because there is also a competition window in the attack code we write , Is in the unlink() After unlinking , But in execution symlink() Before linking to the next file , The context in this short window closes , Previously set SetUID The program may be in this interval fopen() To a root file , and tmp Under folder sticky bit So that this file can only be modified by the owner of the file , Even if it points to globally writable dev/null, Can't execute （ Because the owner becomes root）

resolvent ： Atomization unlink() as well as symlink() sentence
Method 1： Not added usleep() Functional improved.c Program, then execute the attack program first. Then executing the script program can get the experimental results in a short time .

Method 2 ： Add... At the end usleep() function

The attack succeeded , Just say this , see passwd if there test The user has been added.

```sh
[10/12/22]seed@VM:~$ cat /etc/passwd | grep test
test:U6aMy0wojraho:8:0:test:/root/bin/bash
```

And test , Can I get it root jurisdiction , Discovery is possible.

```sh
[10/12/22]seed@VM:~$ su test
Password:
root@VM:/seed#
```
It should be noted here that the protection mechanism for links should be kept closed at all times. 

## 5 Task 3: Countermeasures

## 5.1 Task 3.A: Applying the Principle of Least Privilege

```sh
/* vulp3.c */
#include <stdio.h>
#include <unistd.h>
#include <string.h>
int main()
{ 
  char *fn = "/tmp/XYZ";
  char buffer[60];
  FILE *fp;
  /* get user input */
  uid_t euid = geteuid();
  uid_t uid = getuid();
  seteuid(uid);
  if (!access(fn, W_OK))
  {
    fp = fopen(fn, "a+");
    fwrite("\n", sizeof(char), 1, fp);
    fwrite(buffer, sizeof(char), strlen(buffer), fp); 
    fclose(fp);
  }
  seteuid(euid);
}
```

The above vulnerable progrm shows that we downgrade the privieges befroe the checks and then revise the privileges at the end of the program. If we try the same attack again, it doesn't work.


## 5.2 Task 3.B: Using Ubuntu’s Built-in Scheme

As we mentioned in the initial setup, comes with a built-in protection scheme against race condition 
attacks.
In this task, you need to turn the protection back on using the following command:
```sh
$ sudo sysctl -w fs.protected_symlinks=1
```

After we turn on the sticky bit protection and perform th same attack, the attack is not successful as we cannot follow symlinks from the /tmp directory. The protection scheme worked since in this case, the follower is root, and owner of the /tmp directory is root and the symlink owner is seed. From the above screenshot, it can be observed that access will be denied. This isn’t a good protection mechanism as this has a few limitations. The mechanism works only for sticky bit directories like /tmp or /var/tmp. So the attacker can exploit the race condition in other directories and gain access. Firstly, it works only for directories where sticky bit is enabled. Secondly, the protection mechanism denies access only in a couple of cases as shown as in the above screenshot. In case 5 where the follower is root, the owner of the directory is seed and symlink owner is seed. The attack is successful in that case. This can be exploited and race condition would work in a directory owned by root and root owned file can be modified.

