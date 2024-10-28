## CTF 정보
- CTF: HeroCTF 2024
- 문제 이름: free_shell
- 점수: 50
- 난이도: very easy

## Description
Your goal is to find a valid solution to acquire a shell on the remote server.
Once you have discovered a valid solution locally, you can test it on:
> Deploy on deploy.heroctf.fr

- Format : Hero{flag}
- Author : xanhacks

## Solution
```python
#!/usr/bin/env python3

import os
import subprocess


print("Welcome to the free shell service!")
print("Your goal is to obtain a shell.")

command = [
    "/bin/sh",
    input("Choose param: "),
    os.urandom(32).hex(),
    os.urandom(32).hex(),
    os.urandom(32).hex()
]

subprocess.run(command)
```

shell command를 input으로 받아서 실행한다. 
원래는 command list에 있는 명령어를 ,를 기준으로 구분하는데, 입력에서 띄워써서 입력하면 구분하지 못해서 그대로 파일명으로 찾는 것 같다.

예를 들어, -c ls 라고 입력하면 `/bin/sh -c ls`로 실행될 것이라 생각했는데 
```shell
sikk@gimnamlyeong-ui-MacBookPro free_shell % python3 free_shell.py 
Welcome to the free shell service!
Your goal is to obtain a shell.
Choose param: -c ls
/bin/sh: - : invalid option
```

이런 식으로 오류가 발생한다.

`-c` 옵션만 작성할 경우 리스트에 들어가 있는 랜덤한 해시 값을 실행하도록 되어 있어, 해시에 해당하는 명령을 실행하고자 하기 때문에 에러가 발생한다.


## Exploit Algorithm
`/bin/sh` 옵션 중에 `-s` 옵션은 입력을 받아서 명령어로 실행하는 옵션이다.

> Read commands from the standard input.
>
> Reference: https://pubs.opengroup.org/onlinepubs/007904975/utilities/sh.html

즉, 입력 값에 `-s` 옵션을 작성하면 sh 쉘이 실행되면서 원하는 명령을 실행해볼 수 있다.

![alt text](image.png)


## Flag
`Hero{533m5_11k3_y0u_f0und_7h3_c0223c7_p424m3732}`