---
layout: post
title: Mac에서 Tomcat설치
#excerpt: "excerpt"
categories: [macos/development environment]
tags: [macos, environment, 개발환경, tomcat]
---

1.Apache Tomcat 다운

아파치 홈페이지에 접속하여 **Core : tar.gz** 버전을 다운로드 한 후 압축 해제한다.
(맥의 기본적인 다운로드 구조 : *'~/Downloads'*, **'~'**의 경우 리눅스에서 **홈** 을 가리키며 보통 *'/Users/[사용자계정]'* 으로 이루어져 있다.)

<!-- more -->

2.디렉토리 위치 변경

맥의 터미널을 켠 후 먼저 **'/usr/local'** 경로가 있는지 확인을 하고, 없을 경우에 아래와 명령어를 통하여 디렉토리 생성 및 톰캣 디렉토리의 위치를 변경해 준다.

*디렉토리생성 명령어*

> sudo mkdir -p /usr/local (* p 옵션의 경우 입력한 경로대로 디렉토리를 생성을 위한 것)

*디렉토리 이동 명령어*

> sudo mv ~/Downloads/apache-tomcat-8.0.47 /usr/local



3.Tomcat 디렉토리 링크

보다 편한 접근을 위해 공통으로 관리하고 있는 루트 라이브러리 디렉토리에 링크(바로가기라고 해야 할까)를 아래의 명령어로 추가해 준다.

>  sudo ln -s /usr/local/apache-tomcat-8.0.47 /Library/Tomcat

만약 기존에 설치해 놓은 톰캣 링크가 있다면 아래 명령어로 제거 후 다시 위의 명령어로 추가한다.

>  sudo rm -f /Library/Tomcat



4.링크에 소유자 및 권한 변경

추가된 링크에 소유자를 아래 명령어를 통해 변경한다.

> sudo chown -R [사용자계정] /Library/Tomcat

이번에는 아래 명령어를 통해 실행권한을 부여해 준다.

> sudo chmod +x /Library/Tomcat/bin/*.sh




