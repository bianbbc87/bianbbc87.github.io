---
title: Linux Permission 해석하는 방법 정리 (rwx-rx-r--)
date: 2025-10-29 20:32:00 +0900
published: true
categories: [Linux, Security]
tags: [permission]
media_subpath: '/assets/img/posts/20251029-config'
---

## Linux File 권한 알아보기

### Example

*아래 코드의 권한을 해석하시오.*

`- r w - r - - r - - @`

<br />

## 문자 위치 해석

1. 첫 문자: 파일 타입
    - `-`: 일반 파일
    - `d`: 디렉토리
    - `l`: 심볼릭 링크
    - `c/b`: 문자/블록 디바이스 등
2. 다음 9문자: 권한 문제
    - owner: positions 2-4 (`r w x` 또는 `-`
    - group: positions 5-7
    - others: positions 8-10
        1. `rw- r-- r--`
3. 마지막 문자(옵션): 추가 표시
    - `+`: ACL (Access Control List) 존재 (주로 GNU/리눅스 `ls` 또는 일부 시스템)
    - `@`: extended attributes (xattr) 존재 - macOS/BSD 표기 (파일에 확장 속성 있음)
        - (macOS에서는 `ls -l`에 `@`가 붙으면 `ls -l@` 로 확장 속성 내용 확인 가능

<br />

## rwx, s, t 권한 문자

| 문자  | 의미           | 설명                               | 숫자 값 |
| --- | ------------ | -------------------------------- | ---- |
| `r` | 읽기 (read)    | 파일 내용을 읽거나 디렉토리 내 파일 목록을 볼 수 있음  | 4    |
| `w` | 쓰기 (write)   | 파일 내용을 수정하거나 디렉토리 내에 파일 생성·삭제 가능 | 2    |
| `x` | 실행 (execute) | 파일이면 실행 가능, 디렉토리면 진입(`cd`) 가능    | 1    |
| `-` | 권한 없음        | 해당 권한이 부여되지 않음                   | 0    |

**예시:**

- owner: `rwx` (읽기/쓰기/실행) (7)
- group: `r-x` (읽기/실행) (5)
- others `r--` (읽기만) (4)

**권한 번호:** `754`

<br />

### 특수 문자

#### 1. **s / S — setuid / setgid**
파일 실행 시, 실행하는 사용자의 권한이 아니라 파일 소유자(owner)나 그룹의 권한으로 실행된다.
즉, 권한을 **임시로 상승**시키는 역할을 한다.

| 기호  | 위치                                 | 의미                        |
| --- | ---------------------------------- | ------------------------- |
| `s` | 소유자(owner)의 실행 위치                  | **setuid (set user ID)**  |
| `s` | 그룹(group)의 실행 위치                   | **setgid (set group ID)** |
| `S` | 실행(`x`) 권한이 없는데 setuid/setgid만 설정됨 | 실행 권한 없음 + 설정 비활성         |

**예시:**
```bash
# /usr/bin/passwd는 root 소유이지만, 일반 사용자가 실행할 때도 root 권한으로 실행된다.
-rwsr-xr-x  1 root root /usr/bin/passwd
```

#### 2. **t / T — Sticky bit**
해당 디렉토리 안의 파일을 소유자만 삭제 가능하게 한다.
즉, 여러 사용자가 쓸 수 있는 디렉토리에서도 서로의 파일을 삭제하지 못하도록 보호할 수 있다.

| 기호  | 위치                             | 의미                    |
| --- | ------------------------------ | --------------------- |
| `t` | others의 실행 위치                  | sticky bit 활성 + 실행 가능 |
| `T` | others 실행권한 없음 + sticky bit 활성 |                       |


**예시:**
```bash
# /tmp는 모든 사용자가 파일을 만들고 쓸 수 있지만, 자기 파일만 삭제할 수 있음.
drwxrwxrwt  9 root root /tmp
```

<br />

## owner, group, others

`owner`: 소유자, 파일을 만든 사용자 혹은 chown으로 지정된 사용자이다.

`group`: **파일이 속한 그룹에 속한 사용자**들에게 적용되는 권한으로, staff 그룹이면 staff 그룹 구성원들은 group에 권한 r—만 적용 받는다.

- 그룹 권한은 협업 시 유용하다. 예를 들어 팀원들이 같은 그룹에 있을 때, group에 rw 권한을 주면 모두 수정이 가능하다.

`others`: 소유자도 아니고, 그룹에도 속하지 않은 나머지 모든 사용자

- 보안상 가장 **제한적인 권한**을 가져야 한다.
- r—이면 모든 사람이 읽을 수는 있지만 수정은 못한다.

<br />

### 권한 적용 우선순위

(1) Owner → (2) Group → (3) Others
즉, 사용자가 여러 그룹에 속해 있어도, **가장 상위에 해당하는 권한이 적용**된다.

<br />

### 사용자 ↔ 그룹 관계

- 그룹은 사용자 (user) 들로만 구성된다.
- 디렉토리나 파일이 그룹의 **구성원**이 될 수는 없다. (member 관계)

**그룹에 사용자 추가하기**

```bash
# devteam 이름의 그룹 생성
sudo groupadd devteam

# eunji 사용자를 devteam 그룹에 추가
sudo usermod -aG devteam eunji
```
<br />

### 파일, 디렉토리 ↔ 그룹 관계

- 파일이나 디렉토리는 하나의 **소유 그룹**에 속할 수 있다. (ownership 관계)
- `chgrp` 명령어로 소유 그룹을 변경 가능

**디렉토리의 소유 그룹 변경하기**
```bash
# project/ 디렉토리의 소유 그룹을 devteam으로 변경
chgrp devteam project/

# project/의 소유 그룹 확인 명령어
ls -l

# 결과
# drwxrwxr-x  3 eunji devteam 4096 Oct 31  project
```

<br />

### OS별 Group 조회 명령

#### 1. 그룹 조회 명령어
```bash
cat /etc/group
```

#### 2. LDAP 등 외부 인증 시스템을 사용하는 경우까지 포함한 그룹 조회
```bash
getent group
```

#### 3. bash 내장(bash에서 지원하는 그룹 이름만 간단히 나열)
```bash
compgen -g
```

#### 4. macOS 한정
```bash
dscl . list /Groups
```

#### 5. windows
```bash
net localgroup
```

<br />

### 그룹에 권한 부여

- `g`: 그룹, `u`: 소유자, `o`: 기타 사용자, `a`: all
- `+`: 더하기 `-`: 빼기 `=`: 덮어쓰기

```bash
chmod g+rwx myfile.txt # 그룹에 읽기/쓰기/실행 권한 추가

chmod g+r file # 그룹의 읽기 권한 추가
chmod g-w file # 그룹의 쓰기 권한 제거
chmod g=rw file # 그룹 권한을 읽기+쓰기만으로 설정
```

#### 파일 또는 디렉토리에 그룹 지정

```bash
# 그룹만 변경
chgrp devteam myfile.txt
```

#### 소유자와 그룹 동시 변경

```bash
chown eunji:devteam myfile.txt
```

#### 재귀 적용 (재귀적으로, 하위 그룹 권한에도 모두 적용하기)

```bash
chown -R eunji:devteam /srv/project
chgrp -R devteam /srv/project
```

#### 기존 사용자에게 추가 그룹 부여

```bash
usermod -aG devteam eunji
```

#### 사용자가 속한 모든 그룹 확인
```bash
groups eunji

# or
id eunji # 기본그룹(GID)와 보조그룹 모두 확인 가능
```

<br />

## 기타 Commands

`-R` : `-R` 혹은 `--recursive` 로 부른다. 붙이면 그 디렉토리 내부의 모든 파일과 폴더에도 동일한 동작을 수행한다. **(재귀)**

ex. `chmod -R 755 /srv/project` : 아래의 모든 하위 폴더와 파일 권한을 755로 바꾼다.

ex. `chmod -R eunji:devteam /src/project` : 아래의 모든 하위 폴더와 파일 소유자(owner)를 eunji, 그룹(group)을 devteam으로 바꾼다.

ex. `chgrp -R devteam /srv/project` : 아래의 모든 하위 폴더와 파일 전부의 그룹 소유자는 devteam으로 변경

ex. `ls -R /src/project` : 하위 모든 디렉토리 내용까지 목록을 한번에 출력

<br />