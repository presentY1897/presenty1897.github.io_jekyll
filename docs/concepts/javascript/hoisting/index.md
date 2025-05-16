---
layout: page
title: Hoisting
parent: Javascript
permalink: /docs/concept/javascript/hoisting/
---

# 호이스팅(Hoisting)

| Javascript Interpreter가 코드를 실행하기 전에 functions, variables, classes 혹은 imports를 scope의 최상단에서 선언(declaration)하는 것 처럼 보이는 현상을 말한다.

Javascript에서 변수는 선언, 초기화, 할당의 과정을 거친다.

1. 실행 컨텍스트의 Execution Phase에서 실행 컨텍스트에 변수와 함수를 등록하며 이 과정에서 선언이 이루어진다.
1. 선언된 변수를 사용하기 위해 메모리를 확보하는데 일반적으로 `undefined`를 할당한다. `var`는 Execution Phase에서 여기까지 진행하고, `let`과 `const`는 런타임 과정에서 진행한다.
1. 할당할 부분을 메모리에 저장하고 메모리의 주소를 초기화되어있는 변수에 할당한다.

{% include_relative temporal-dead-zone.md %}

{% include_relative functional-hoisting.md %}

{% include_relative class-hoisting.md %}

{% include_relative import-hoisting.md %}

---

# 참조

1. [Mozilla, Hoisting](https://developer.mozilla.org/en-US/docs/Glossary/Hoisting)
1. [변수 선언 초기화 할당](https://velog.io/@tess/%EB%B3%80%EC%88%98-%EC%84%A0%EC%96%B8-%EC%B4%88%EA%B8%B0%ED%99%94-%ED%95%A0%EB%8B%B9)
