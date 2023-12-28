---
title: Chirpy 테마 Ruby Version Issue 해결기
author: yurijeno
date: 2023-12-28 12:00:00 +0900
categories: [Programming, Jekyll]
tags: [Chirpy, Jekyll, Static Site Generator, Git, Github, Github Actions, Github Workflow, Open Source, Dependency]
# toc: false
# pin: false
# math: false
# mermaid: false
img_path: /assets/img/post/2023-12-28-Chirpy 테마 Ruby Version Issue/
#image:
#  path: Beelinik-MINI-S.jpg
#  width: 400
#  height: 250
  # alt: Responsive rendering of Chirpy theme on multiple devices.
---

## 요약

오픈소스는 위대하지만 종종 의존성과 버전 문제 때문에 귀찮게 한다.

그런데 이런 문제도 누군가는 이미 겪었고, 해결책도 보통은 있다

배운점; 멀쩡하던 코드가 갑자기 배포가 안되면 최근에 특정 모듈이 **새로 버전업 릴리즈되었을 가능성**이 높다

## 배경

오픈소스는 정말 좋은 동네(?)다. 수많은 능력자들이 이미 내가 생각한건 다 만들어놓았고, 내가 겪은 문제도 이미 누군가가 다 겪었고 해결법도 다 공유되고 있는 동네다. 그럼에도 불구하고 오픈소스에서 골치 아픈 상황이 있는데, 제일 사람 귀찮게 하는것은 수많은 라이브러리 의존성으로 인한 버전 문제다.

문제의 발단은 [이전 글]({% post_url 2023-12-26-CCTV 사용 시 먼저 봐야할 법적 사항들 %})을 등록하는 과정에서 터졌다. 현재 블로그를 [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) 테마로 Jekyll로 작성하고 있고, Github Page와 Actions로 배포하고 있는 중이다.

Jekyll(Chirpy 테마)과 Github Page에서 블로그 배포하는걸 아주 간략하게 요약하면
- Chirpy 테마는 github repo를 clone해서 사용하는데
- Chirpy를 클론하면 액션과 워크플로우도 같이 클론된다
  - 참고로 Chirpy는 시작단계 스크립트 돌리면 블로그 배포용의 워크플로우만 남김
- 새 포스트를 작성해서 레포지토리에 push하면 알아서 배포 워크플로우를 밟으면서 페이지 생성하고 새 글이 배포됨

## 문제 발생

그런데 [이전 글]({% post_url 2023-12-26-CCTV 사용 시 먼저 봐야할 법적 사항들 %})을 push하는데 멀쩡하던 워크플로우가 돌아가지 않았다.

![Github Action Failure Result](github-workflow-fail.png)

멀쩡하던 코드가 왜;; 깃허브 액션 들어가서 에러메시지를 보니 다음과 같다

```
...

An error occurred while installing google-protobuf (3.25.1), and Bundler cannot
continue.

In Gemfile:
  jekyll-theme-chirpy was resolved to 6.2.3, which depends on
    jekyll-archives was resolved to 2.2.1, which depends on
      jekyll was resolved to 4.3.2, which depends on
        jekyll-sass-converter was resolved to 3.0.0, which depends on
          sass-embedded was resolved to 1.69.5, which depends on
            google-protobuf
Error: The process '/opt/hostedtoolcache/Ruby/3.3.0/x64/bin/bundle' failed with exit code 5
```

후...딱 봐도 뭔가 버전 차이로 의존성 꼬인 문제다. 이런건 구글링해도 온갖 경우가 튀어나와서 오히려 찾기가 어렵다.

## 문제 원인 파악

다행히 Chirpy Repo에 나랑 동일한 문제를 겪는 issue [#1429](https://github.com/cotes2020/jekyll-theme-chirpy/issues/1429)가 등록되어있었다.

타이밍 한번 기가 막히게도 글을 등록하던 시점의 딱 하루전, 25일에 [Ruby 3.3.0](https://www.ruby-lang.org/en/news/2023/12/25/ruby-3-3-0-released/)이 릴리즈되면서 버전이 꼬여버린것이다

Chirpy 코드에서 문제가 되는 `pages-deploy.yml` 파일을 보면 다음 부분이 보인다

```yaml
...
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      # ...
      - name: Setup Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: 3
          bundler-cache: true
...
```

옳거니. Workflow 명세서에는 `ruby-version`을 `3`으로만 명시해서 최신버전이 그때그때 달라진 것이었다.
- 24일 전까지는 최신버전인 Ruby 3.2가 돌다가
- Ruby 3.3이 릴리즈된 25일 이후부터는 Ruby 3.3으로 돌면서 기존 코드가 다 호환이 안되었던 것

## 해결

이미 [#1429](https://github.com/cotes2020/jekyll-theme-chirpy/issues/1429)에 해결책이 다 적혀있어서 `pages-deploy.yml`을 다음과 같이 바꿔서 포스트 릴리즈 성공했다; `ruby-version` 을 `3`이 아닌 `3.2.2`로 수정

```yaml
...
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      # ...
      - name: Setup Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: 3.2.2
          bundler-cache: true
...
```

Chirpy에서는 이렇게 [커밋](https://github.com/cotes2020/jekyll-theme-chirpy/commit/c45e0311552e320417bb3d6cab4296d678e14037)했으니 참고할 것
- `.github/workflows/ci.yml` 파일은 Chirpy 테마 자체에 대한 개발용도로 사용되는 워크플로우
- 블로그 운영 시에는 `.github/workflows/pages-deploy.yml` 파일만 바꿔주어도 됨
  - 초기화 스크립트 돌리면 `.github/workflows/pages-deploy.yml.hook` 파일이 `.github/workflows/pages-deploy.yml`로 rename