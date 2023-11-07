---
title: .gitattributes 파일로 인한 git의 자동 개행문자 변경(CR, LF, CRLF)
author: yurijeno
date: 2023-11-05 17:00:00 +0900
categories: [Programming, Git]
tags: [Computer, Programming, Git, Github, VCS, .gitattributes, 삽질경험, core.autocrlf]
# toc: false
# pin: false
# math: false
# mermaid: false
img_path: /assets/img/post/2023-11-05-gitattributes와 core_autocrlf/
#image:
#  path: Beelinik-MINI-S.jpg
#  width: 400
#  height: 250
  # alt: Responsive rendering of Chirpy theme on multiple devices.
---

## Summary

- `.gitattributes` 파일은 `$ git config core.autocrlf` 설정보다 우선순위가 높다.
- 내가 git의 자동 개행문자 변경을 허용하지 않았는데도 git clone 후 개행문자 차이로 버그가 발생했다면 다음 설정을 확인해야한다.
  1. `$ git config --global core.autocrlf`
  2. `$ git config core.autocrlf`
  3. `$ cat .gitattributes`


## Background

### Operating System 간 개행문자 차이

컴퓨터에서는 문서에서 줄을 바꾸는 행위를 개행문자로 표시한다. 다음과 같은 문서를 예를 들어보자.

```markdown
이건 첫째줄 입니다
이건 둘째줄 이구요
```

이 문서는 실제로는 "이건 첫째줄 입니다(개행문자)이건 둘째줄 이구요"로 저장되어 있다

문제는 개행문자가 윈도우와 유닉스계열(Mac, Ubuntu, Centos ...)간 차이가 있다는 것이다. 윈도우는 CR(carriage return: `\r\n`) 데이터를 사용하고 유닉스 계열은 LF(line feed: `\n`) 데이터를 사용한다.
- Windows: `이건 첫째줄 입니다\r\n이건 둘째줄 이구요`
- **nix  : `이건 첫째줄 입니다\n이건 둘째줄 이구요`

### git의 제공 기능: 개행문자 자동변경

이런 차이를 해소하기 위해서 git은 사용자가 개행문자를 자동으로 변경하도록 허용할지 아닐지 결정할 수 있도록 해준다. 오픈소스 계열은 대부분 유닉스 환경 기준으로 코드가 작성되기 때문에, 윈도우에서 개발할때 이 설정을 사용할일이 많을것이다. 실제로 Git Windows Client를 설치할 때 다음과 같은 설정화면이 중간에 떠서 개행문자 변경 행동을 설정한다.

![git windows 설치화면](git windows 설치화면.png)

요즘은 그래도 에디터들이 개행문자 스타일에 맞추어주기 때문에 나는 그냥 `Checkout as-is, commit as-is`를 사용하고 있었다.

## Trouble

문제의 발단은 [Chirpy Theme](https://github.com/cotes2020/jekyll-theme-chirpy)을 윈도우 운영체제안에서 클론 하고 `tools/init` 스크립트를 돌리는데 자꾸 에러가 나는 것이었다. Bash 쉘로 실행을 하니 개행문자가 잘못되어있다면서 에러가 발생하였다.(Windows OS의 `\r\n` 방식의 개행문자를 만나면서 `bash`가 에러를 발생)

우선 먼저 git 설정을 살펴봤다. `core.autocrlf`가 `false`인 것을 확인했는데 이는 레포지토리에서 그대로 가져오라는 의미이다
- 'Checkout as-is, commit as-is'
- `\n` 이면 `\n` 으로, `\r\n`면 `\r\n`로 로컬로 가져온다

```bash
$ git config core.autocrlf
false
```

그런데 이번 경우는 이상하게 로컬로 클론되면서 개행문자가 `\n` 에서 `\r\n` 로 바뀌고 있었다.


## Troubleshooting
**삽질하면서 알아본 결과 범인은 `.gitattributes` 파일이었다**

보통 `.gitignore`는 지겹게 봐도 `.gitattributes` 파일은 흔하게 보이는 파일은 아니다. 전체 다 설명하려면 밑도 끝도 없으므로, Chirpy Theme 레포지토리에 있는 [.gitattributes](https://github.com/cotes2020/jekyll-theme-chirpy/blob/master/.gitattributes) 파일을 살펴보자.

```bash
# Set default behavior to automatically normalize line endings.
* text=auto

# Force bash scripts to always use LF line endings so that if a repo is accessed
# in Unix via a file share from Windows, the scripts will work.
*.sh text eol=lf

# Force batch scripts to always use CRLF line endings so that if a repo is accessed
# in Windows via a file share from Linux, the scripts will work.
*.{cmd,[cC][mM][dD]} text eol=crlf
*.{bat,[bB][aA][tT]} text eol=crlf

# Denote all files that are truly binary and should not be modified.
*.png binary
*.jpg binary
*.ico binary
```

내가 실행하려던 파일은 `tools/init` 파일이었는데 맨 첫두줄을 보면 확장자가 없는 파일은 `auto`로 적용(**개행문자 자동변경**)된다

레포지토리에 있는 `.gitattributes` 설정이 내 `git config` 보다 우선함을 알 수 있다.

> 원래 `tools/init.sh` 파일이었는데 Chirpy가 업데이트 되면서 확장자가 빠지고 `tools/init`으로 수정되었음
> - 그래서 패치전에는 `*.sh text eol=lf` 룰에 의해 강제적으로 LF(`\n`)으로 긁어왔는데
> - 패치되면서 `* text=auto` 룰에 적용받아 `core.autocrlf='auto'`처럼 행동한 것이다
{: .prompt-info }


## Solution

어차피 Jekyll Local Build를 WSL에서 수행하고 있었기 때문에, 처음 클론을 수행할때 WSL Linux git에서 클론을 수행하여 tools/init 스크립트를 그대로 들고오도록 하였다.
> - Chirpy 의 Contributor 들이 일부러 확장자를 제거한데에는 이유가 있을거 같아서 내 클라이언트 환경을 바꾸는 선택지를 택함
{: .prompt-info }


## References
- https://git-scm.com/docs/gitattributes
- https://github.com/cotes2020/jekyll-theme-chirpy