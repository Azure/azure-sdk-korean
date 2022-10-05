---
title: "주요 컨트리뷰션 - CSpell"
keywords: contribution
permalink: contribution_cspell.html
folder: contribution
sidebar: general_sidebar
---

주요 컨트리뷰션 중 CSpell 관하여 설명합니다

## 사전 준비
macOS 사용자들은 PowerShell 을 사용하기 위해, [여기](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-macos?view=powershell-7.2)를 참고하여 설치를 합니다


## 해야할 일
1. [Node.js](https://nodejs.org/en/download/) 설치되어 있는지 확인합니다
2. `.vscode/cspell.json` 에서 `ignorePaths` 항목에 해당 라이브러리를 삭제 하여, 그 모듈이 더이상 무시되지 않고 검사가 될 수 있도록 만듭니다. 예를 들어, **sdk/storage/azure-storage-blob-changefeed/**. 
3. 저장소 루트 위치에서 다음과 같이 실행합니다
   ```powershell
    ./eng/common/spelling/Invoke-Cspell.ps1 -ScanGlobs "sdk/storage/azure-storage-blob-changefeed/**"
    ```
4. [False positives](http://aka.ms/azsdk/engsys/spellcheck) 내용을 참고하여, 오타를 수정합니다
5. 아까전 `.vscode/cspell.json` 에서 `ignorePaths` 수정에 해당한 라이브러리에 대해 변경사항을 확인합니다. `.vscode/cspell.json` 을 포함한 변경사항을 git 에 반영하여 추가합니다

## 참고 PR
- [머지된 EventHub 라이브러리 CSpell PR](https://github.com/Azure/azure-sdk-for-python/pull/26218)
- [머지된 다른 라이브러리 CSpell PR들](https://github.com/Azure/azure-sdk-for-python/pulls?q=is%3Apr+is%3Amerged+cspell)
