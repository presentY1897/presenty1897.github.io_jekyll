---
layout: post
title: "PostgreSQL의 Extension 시행착오"
description: "PostgreSQL에 새로운 Extension을 구현하는 내용 기록"
comments: true
keywords: "postgreSQL, pgRouting, c++"
tags: postgreSQL, tryanderror
---

# 개요

이 포스트는 postgreSQL으로 새로운 Extension을 구현하면서 겪은 시행착오에 대한 내용입니다.
~~작성 당시의 글쓴이는 정상이 아니기에 이 글의 내용은 중구난방일 가능성이 높습니다.~~

postgreSQL 기반으로 대중교통 경로 탐색 알고리즘을 구현하는 중 postgreSQL Extension을 개발해야 했다.
해당 프로젝트에서 대중교통 경로 탐색 알고리즘은 RAPTOR를 이용했다. RAPTOR를 pgRouting을 참고하여 개발하고자 했다.
RAPTOR는 C#을 이용하여 구현한 적이 있었기에 알고리즘을 이식하는 데는 큰 문제가 없었다.

문제는 postgreSQL에서 Extension을 구축해본적이 없었고, postgreSQL의 extension이 C++ 기반이라는 점이다.
C++은 학부생 때, 포인터 개념 정도까지밖에 이해하지 않아 pgRouting을 이해하는 데에도 시행착오가 있었다.
더군다나 pgRouting에서는 C++의 다양한 확장 기능으로 boost를 쓰고 있었기에 큰 난관이 예상되었다.

다른 방법으로는 기존에 구축한 C#을 MSSQL의 function으로 만들어 MSSQL 기반의 웹서비스를 구현하는 것이다.
그런데 MSSQL은 유료라는 큰 문제가 있었다. 또 장기적으로 보았을 때, postgreSQL 기반으로 함수를 구현해 놓는 게
다른 사람들이 사용하고 검증 받기 편할 것으로 판단되었다.

## 수행 방법

이 포스트를 작성할 때 살펴본 내용은 pgRouting의 소스코드와 postgreSQL에서 extension을 만드는 방법이었다.
처음에 이 프로젝트를 설계했을 때는 pgRouting을 수정하여 RAPTOR를 구현하려고 했었다.
하지만 pgRouting 도 오랫동안 개발되어 온 만큼 짧은 시간 안에 코드를 이해하긴 어려워 보였다.

결국 postgreSQL extension을 예제를 따라 만들고, 차례대로 RAPTOR를 구현해나가기로 했다.
구현하는 과정에서 pgRouting의 소스를 참고하여 최종적으로는 pgRouting에 결합할 수 있는 방법을 찾아보거나,
적어도 pgRouting은 이해할 수 있도록 목표를 잡았다.

## postgreSQL extension

우선 postgreSQL extension을 생성하는 방법을 구글링하면 [postgreSQL의 공식 문서](https://www.postgresql.org/docs/9.1/sql-createextension.html)를 포함하여 몇 가지 링크가 나온다.
~그러나 사실 공식 문서는 크게 도움이 되진 않았다.~ 구현에는 주로 [big elephants](http://big-elephants.com/2015-10/writing-postgres-extensions-part-i/)와 [serveralnines](https://severalnines.com/blog/creating-new-modules-using-postgresql-create-extension)를 참고했다.

### 필요한 파일

우선 postgreSQL의 Extension을 추가하기 위해서는 두 가지 파일이 필요하다.

    1. Control file .control
    2. SQL script file .sql

Control file은 extension에 대한 설명, 그리고 실제로 구현될 함수에 대한 내용이 담긴 sql파일로 이해했다.
그리고 make file 어쩌구가 나오는 데, 이해가 안된다.

[Extension Building Infrasturcture 공식 문서](https://www.postgresql.org/docs/9.1/extend-pgxs.html) 일단 구글링을 하니 이 문서에서 makefile을 extension이 있는 곳에 위치한 뒤 빌드하면 된다고 한다.
문제는 빌드를 어떻게 하느냐. 아니 make은 linux에서 사용하는 명령어라니. 시불탱 window 유저는 어쩌란 거냐. [How to install and use "make" in windows](https://stackoverflow.com/questions/32127524/how-to-install-and-use-make-in-windows)

그래서 링크를 따라 들어가 MinGW를 설치해보았다. 그런데 mingw 만으로는 mingw32-make이 설치되지 앟는다. Package에서 ingw32-base-bin을 선택하고 installation -> update catalogue로 설치를 해주어야했다.
그리고 귀찮지만 "C:\MinGW\bin"를 환경 변수에 넣어주었다.

```cmd
mingw32-make install
```

대망의 실행. 그러나 오류 메세지만 날 반긴다.

```cmd
mingw32-make install
makefile:7: Files/PostgreSQL/11/lib/pgxs/src/makefiles/pgxs.mk: No such file or directory
mingw32-make: *** No rule to make target 'Files/PostgreSQL/11/lib/pgxs/src/makefiles/pgxs.mk'.  Stop.
```

왜 linux가 아니어서...

그래서 방법을 바꾸어 window로 extension을 만드는 법을 검색했다.
[postgres wiki](https://wiki.postgresql.org/wiki/Building_and_Installing_PostgreSQL_Extension_Modules)에서 다행히 visual studio로 빌드하는 법을 소개하고 있었다.
그리고 해당 문서에서 참고하라는 [링크](https://www.2ndquadrant.com/en/blog/compiling-postgresql-extensions-visual-studio-windows/)에서 visual studio 에서 윈도우 dll을 생성하는 방법을 소개하고 있다.
이제 이 문서를 참고하여 visual studio code에서 빌드하면 될 것 같다.

visual studio code에 익숙치 않다보니 빌드에도 많은 시행착오가 있다. 우선 build시 control+shift+B를 통해 task.json 파일을 생성한다. 그리고 해당 파일을 수정하여 원하는 방식으로 조절해야하는 데, dll파일을 참조할 주소가 제대로 먹히지 않는다. 구글링 결과 [github에 비슷한 질문](https://github.com/Microsoft/vscode-cpptools/issues/1889)이 있다. 해당 글에선 task의 args에 넣거나 더 좋은 방법으로는 .sh나 .cmd를 사용하는 걸 추천한다. pgRouting도 sh(쉘 스크립트)로 선언되어있으니 sh를 좀 찾아봐야할 것 같다.

~~아니면 visual code에 c++ 확장 기능을 설치해서 프로젝트를 선언해보는 방법도 있을 것 같다.~~ pgRouting에서 window 환경에서 실행하거나 설치하는 스크립트 파일을 찾아보았다. 하지만 별로 도움이 되지 않는다.
결국 다시 sh에서 사용하는 방법을 찾아보아야.

makefile을 비롯해 compile 전반에 대한 이해가 부족해서, 몇 시간 동안 구글링 해봤지만 계속 included path를 찾지 못한 문제가 나타났다. [이 문서](https://devblogs.microsoft.com/cppblog/building-your-c-application-with-visual-studio-code/#Makefiles)를 정독하면 가능할 것 같긴 한데, 다른 방법을 찾는 게 맞을 지 고민된다.

2019.04.22 3:55 우선 vs code에서 dll을 릴리즈 하는 건 보류하기로 했다. 또 mac os에서 extension만 만들어서 가져오는 방법은 제대로 되지 않았다.
visual studio 에서 build 작업을 하기로 하고, 라이선스 받는 동안 vs code에서 RAPTOR를 C/C++로 구현해야겠다.