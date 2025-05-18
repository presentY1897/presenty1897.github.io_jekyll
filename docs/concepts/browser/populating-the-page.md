---
layout: page
title: Populating the page
parent: Browser
permalink: /docs/concepts/browser/populating-the-page/
---

# 웹 페이지 채우기(Populating the page)

## 과정

### 탐색(Navigation)

1. [DNS](/docs/concept/network/dns/) 조회(DNS Lookup)
1. TCP 핸드쉐이크(TCP Handshake)
1. TLS 협상(TLS Negotiation)

### 응답(Response)

### 구문 분석(Parsing)

1. DOM 트리 구축(Building the DOM tree)
1. Preload Scanner
1. CSSOM 구축(Building the CSSOM)
1. Javascript Compilation
1. 접근성 트리 구축(Building the Accessibility Tree)

### Render

1. Style
1. Layout
1. Paint
1. 합성(Compositing)

### 상호작용(Interactivity)

---

#### 참조

1. [Mozilla, Populating the page: how browsers work](https://developer.mozilla.org/en-US/docs/Web/Performance/Guides/How_browsers_work)

#### 연관문서

[웹 성능에서 주요한 두 가지 문제(Two major issues in web performance are issues)](/questions/two-major-problems-of-web-performance.md)
