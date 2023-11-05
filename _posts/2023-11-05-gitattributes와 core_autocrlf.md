---
title: .gitattribute 파일과 core.autocrlf 설정
author: yurijeno
date: 2023-11-05 17:00:00 +0900
categories: [Programming, Git]
tags: [Computer, Programming, Git, Github, VCS, .gitattributes, 삽질경험, core.autocrlf]
# toc: false
# pin: false
# math: false
# mermaid: false
img_path: /assets/img/post/filename/
#image:
#  path: Beelinik-MINI-S.jpg
#  width: 400
#  height: 250
  # alt: Responsive rendering of Chirpy theme on multiple devices.
---


## Background (Trouble)

문제의 발단은 [Chirpy Theme](https://github.com/cotes2020/jekyll-theme-chirpy)을 클론 하고 `tools/init` 스크립트를 돌리는데 자꾸 에러가 나는 것이었음. Windows OS의 `\r\f` 방식의 개행문자를 만나면서 `bash`가 에러를 뿜어대고 있었다

우선 먼저 git 설정을 살펴봤다. `core.autocrlf`가 `false`인 것을 확인했는데 이는 레포지토리에서 그대로 가져오라는 의미이다; `\n` 이면 `\n` 으로, `\r\f`면 `\r\f`로 로컬로 가져온다

```bash
$ git config core.autocrlf
false
```

그런데 이번 경우는 이상하게 로컬로 클론된 것이 개행문자가 `\n` 에서 `\r\f` 로 바뀌었다.

**구글링해서 알아본 결과 범인은 `.gitattributes` 파일이었다**

> - 당시 윈도우 OS에서 git-windows를 사용해 클론하고 있었다.
> - 아래를 보면 알겠지만 이것도 매우 중요한 원인
{: .prompt-info }


## Reason of the trouble

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

어차피 Jekyll Local Build를 WSL에서 수행하고 있었기 때문에, 처음 클론을 수행할때 WSL Linux git에서 클론을 수행하여 문제 해결


## References
- https://git-scm.com/docs/gitattributes
- https://github.com/cotes2020/jekyll-theme-chirpy