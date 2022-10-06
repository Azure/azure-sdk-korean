---
title: "주요 컨트리뷰션 - Python type hint add"
keywords: contribution
permalink: contribution_typehint.html
folder: contribution
sidebar: general_sidebar
---

주요 컨트리뷰션 중 Python type hint에 관하여 설명합니다
Azure-sdk-for-python의 Version Drop으로 인해서 함수의 주석에서 type hint로 전환하여 refactor하는 기여입니다.

## 해야할 일
1. 아직 type hint로 전환하지 않은 파일을 찾습니다.
2. [Python Typing docs](https://docs.python.org/3/library/typing.html)를 참고하여 함수에 typing을 적용합니다.
3. 변경사항을 확인하고 git에 반영하여 추가합니다. 

## 참고 PR
- [ServiceBus/ampq에 적용된 type hint(3.6 기준)](https://github.com/Azure/azure-sdk-for-python/pull/26046)
- [ServiceBus/aio에 적용된 type hint(3.6 기준)](https://github.com/Azure/azure-sdk-for-python/pull/25900)
