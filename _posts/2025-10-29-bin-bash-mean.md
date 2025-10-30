---
title: "#!/bin/bash의 뜻은 무엇이고 왜 사용하는걸까 ?"
date: 2025-10-29 21:30:00 +0900
published: true
categories: [Linux, Shell Script]
tags: [bash, shebang, permission]
media_subpath: '/assets/img/posts/20251029-config'
---

## #!/bin/bash의 의미
Linux Permission에서 `rw-` 는 read, write 권한만 가져 execution (실행)이 불가능합니다. 그런데 종종 `rw-` 권한으로도 프로그램 실행이 가능한 경우가 있습니다.

이 경우, 프로그램 코드 최상단에 `#!/bin/bash`가 있는지 확인 해봅시다.

이 한 줄은 단순한 주석이 아니라, 리눅스 커널에게 **이 파일을 어떤 프로그램(Interpreter)으로 해석해야 하는지** 알려주는 약속입니다.
이걸 **Shebang** 이라고 부릅니다. **Shebang(셔뱅)** 은 스크립트 파일 맨 첫 줄에 쓰는 `#!` 로 시작하는 선언문입니다.
- `#` → 영어로 “hash” (혹은 “sharp”)
- `!` → 영어로 “bang”

<br />

## Linux의 직접 실행과 간접 실행
리눅스에서 스크립트를 실행하는 방법에는 두 가지가 있습니다.
**직접 실행(direct execution)**과 **간접 실행(indirect execution)** 입니다.

<br />

### 직접 실행 (direct execution) - `./script.sh`

이 방식은 운영체제 커널이 해당 파일을 **프로그램처럼 직접 실행**하도록 요청하는 것입니다.

```bash
./script.sh
```

**동작 원리:**

1. `./script.sh`를 입력 → shell이 커널에게 exec() 요청
2. 커널은 파일을 확인함
    1. 이 파일에 실행 권한이 있나?
    2. 이게 실행 가능한 바이너리인가 ? 아니면 shebang (#!/bin/bash)으로 지정된 스크립트인가?
3. x 권한이 없으면 커널이 즉시 거부
    1. `zsh: permission denied: ./script.sh`

<br />

### 간접 실행 (indirect execution) - `#!/bin/bash`

bash라는 프로그램이 **대신 파일을 읽고 실행하는 방식**입니다.
- bash는 이미 커널에게 실행 권한이 허락된 실행 파일(binary) 입니다.
- 따라서 bash가 내부적으로 script.sh를 열고(open()), 읽어서(read()), 한 줄씩 해석(interpret)하며 실행합니다.
- 읽은 내용을 명령어처럼 한 줄씩 해석해서 실행합니다.

즉, 이 경우 커널이 아니라 bash 프로세스가 직접 내용을 읽어 실행하므로, 파일에 **읽기 권한(r)**만 있어도 충분합니다.

**동작 원리:**

1. 파일의 첫 두 바이트가 `#!`인지 확인
2. 그렇다면 “이건 바이너리가 아니라 스크립트구나” 라고 판단하고 뒤에 적힌 프로그램을 대신 실행시킨다.
3. 그 뒤에 적힌 프로그램(예: /bin/bash)을 실행하고, 해당 스크립트를 인자로 넘긴다.

**예를 들어:**
```bash
# bash 명령어
#!/bin/bash
echo "hello"

# 커널이 실제로 실행하는 명령어
/bin/bash path/to/script.sh
```

<br />

### bash 설치 여부

- 리눅스 : 거의 기본 탑재 + 기본 쉘
- macOS: 기본적으로 설치 되어 있지만, 최신 버전에서는 기본 쉘은 bash가 아닐 수 있음(zsh)
- 윈도우: 기본적으로 없음. WSL(Windows Subsystem for Linux) 또는 Git Bash로 사용 가능
