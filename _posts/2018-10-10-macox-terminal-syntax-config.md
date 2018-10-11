---
layout: post
title: Terminal Syntax 설정
#excerpt: "excerpt"
categories: [MacOS/Development Environment]
tags: [macos, terminal, syntax]
comments: true
image:
  feature: macos2.gif
---

MAC의 기본적인 터미널은 단일 색상이 default 이기 때문에 가독성이 떨어진다.
아래의 설정으로 리눅스 터미널과 같이 색상으로 가독성을 높일 수 있다.

<!-- more -->

### 1.파일에 색상 입히기 

~/.bash_profile 파일에 아래와 같이 alias 항목을 추가해 준다.

> alias ls = 'ls -FG'

여기서 F옵션은 파일 목록에서 파일의 유형을 다르게 표시해주는 기능이고, G옵션은 컬러를 사용하겠다는 옵션이다.

추가를 했다면 아래와 같이 source 명령어로 현재 터미널에 적용 되도록 한다.

> source ~/.bash_profile




### 2. vi 편집기에 syntax highlighting 기능 켜기

~/.vimrc 파일을 열어 아래의 항목을 추가해 준다.

> filetype on
> syntax on

만약 해당 파일이 홈(~)에 존재하지 않는다면 아래 경로를 통해 복사해서 사용하면 된다.

> /usr/share/vim/vim80/vimrc_example.vim
