---
title: "Policies: Repository Branching"
permalink: policies_repobranching.html
folder: policies
sidebar: general_sidebar
---

The git branching and workflow strategy we will be using is mostly in line with [OneFlow](https://www.endoflineblog.com/oneflow-a-git-branching-model-and-workflow) with some slight variations called out below.

## 메인 브랜치

 메인은 항상 존재하고, 절대 강제로 푸시하면 안 되는 기본 브랜치입니다. 메인 브랜치는 항상 작동하는 상태를 유지해야 하며, CI(Continuous Integration) 빌드가 성공적으로 수행되어야 합니다 (예: 빌드, 분석, 테스트가 모두 통과). 나머지 모든 브랜치는 의도적으로 짧은 수명을 가지도록 설계되었으며, 더 이상 필요하지 않을 경우 삭제해야 합니다.

메인 브랜치는 언제나 빌드 가능한 상태를 유지하지만, 항상 최신으로 배포된 공식 패키지의 상태를 반영하는 것은 아닙니다. 특정 배포된 패키지의 코드 상태를 파악하려면 그 패키지에 대한 태그를 참조해야 합니다. 이 규칙은 각각의 프로그래밍 언어 레포지토리에 따라 다를 수 있습니다. 예를 들어 파이썬이나 자바스크립트처럼 소스 코드를 직접 참조할 수 있는 언어들은 메인 브랜치가 항상 최신으로 배포된 공식 패키지와 동일한 상태를 유지하려는 경향이 있습니다. 그러므로 각 언어 레포지토리의 구체적인 지침을 참고해야 합니다.

## 포크 상태에서 작업하기

메인 레포지토리의 브랜치 혼잡을 줄이고, 다른 권한을 가진 기여자와 커뮤니티 회원들 모두에게 공통의 작업 흐름을 가능하게 하기 위해, 구성원들은 메인 레포지토리를 포크하여 작업을 진행해야 합니다. 포크에서의 작업이 준비되면 Pull Request를 메인 레포지토리에 제출할 수 있습니다.

포크를 사용하는데 대한 간단한 시작 지침을 위한 다음 몇가지 섹션을 참조하십시오. 또한 Github에서 더 자세한 문서를 보려면 [관련문서](https://help.github.com/en/articles/working-with-forks)를 참조하십시오. 

### 포크된 레포지토리 복제하기

메인 레포지토리에서 포크 버튼을 클릭하여 본인의 포크를 생성한 후에는 아래의 명령어를 사용하여 로컬 레포지토리를 복제하고 설정할 수 있습니다.

```bash
# clone your forked repo locally which will setup your origin remote
git clone https://github.com/<your-github-username>/azure-sdk-for-<lang>.git

# add an upstream remote to the main repository.
cd azure-sdk-for-<lang>
git remote add upstream https://github.com/Azure/azure-sdk-for-<lang>.git

git remote -v
# origin    https://github.com/<your-github-username>/azure-sdk-for-<lang>.git (fetch)
# origin    https://github.com/<your-github-username>/azure-sdk-for-<lang>.git (push)
# upstream  https://github.com/Azure/azure-sdk-for-<lang>.git (fetch)
# upstream  https://github.com/Azure/azure-sdk-for-<lang>.git (push)
```
위의 명령어들을 실행한 후에는 로컬로 복제된 레포지토리와 origin이라는 이름의 본인이 포크한 레포지토리를 위한 원격 레포지토리, 그리고 upstream이라는 이름의 메인 레포지토리를 위한 원격 레포지토리가 모두 설정되어 있어야 합니다.

### 메인 레포지토리로부터 최신 변경 사항을 로컬 및 포크된 레포지토리로 동기화

포크 상태에서 작업하는 것이 강력하게 권장되므로 메인 브랜치에 직접 변경 사항을 커밋하는 것은 지양해야 합니다. 대신 메인 브랜치을 사용하여 구성원들의 다양한 레포지토리를 동기화하는 것이 가능합니다. 이 섹션의 지침은 메인을 동기화 브랜치로 사용한다는 가정하에 작성되었습니다.

```bash
# switch to your local main
git checkout main

# update your local main branch with what is in the main repo
git pull upstream main --ff-only

# update your forked repo's main branch to match
git push origin main
```

이 시점에서 세 개의 레포지토리(로컬, origin, upstream)가 모두 일치하고 동기화되어야 합니다.

로컬 또는 origin의 메인 브랜치가 실수로 동기화에서 벗어나는 것을 방지하기 위해 --ff-only (fast-forward only) 옵션을 사용합니다. 이 옵션은 메인 레포지토리에 이미 존재하지 않는 커밋이 있는 경우 실패하게 됩니다. 이와 같은 상태에 빠지게 된다면, 가장 간단한 해결 방법은 로컬의 메인 브랜치를 강제로 리셋하는 것입니다.

```bash
# Warning: this will remove any commits you might have in your local main so if
# you need those you should stash them in another branch before doing this
git reset --hard upstream/main

# If you also have your forked main out of sync you might need to use the force option when you push those changes
git push origin main -f
```
### 브랜치 생성 및 포크로 푸시하기

로컬의 메인 브랜치가 upstream의 메인 브랜치와 동기화된 후, 새로운 브랜치를 생성하고 작업을 진행할 수 있습니다.
```bash
git checkout <branch-name>

# Make changes
# Stage changes for commit
git add <file-path> # or * for all files
git commit

git push origin <branch-name>
```
이 시점에서는 GitHub의 메인 레포지토리로 이동하여, 푸시한 변경 사항으로 pull request를 생성하는 링크를 확인할 수 있어야 합니다.

_Tip_: 몇몇 사람들은 빠르게 스테이지하고 간단한 메시지와 함께 동시에 커밋하는 것을 선호합니다. 그럴 경우 아래의 명령어를 사용할 수 있습니다.

```bash
git commit -am "Commit message"
```
`-a`는 변경된 모든 파일을 커밋하라는 의미이므로 작업 디렉토리에 수정된 다른 파일이 없도록 주의해야 합니다. `-m`은 커맨드 라인에서 커밋 메시지를 전달할 수 있게 해주어 빠른 커밋에 유용하지만, 리뷰를 받고 메인 리포지토리에 병합하고자 하는 변경사항을 푸시할 때는 구성된 에디터를 사용하여 더 나은 커밋 메시지를 작성하고자 할 수 있습니다

### 최신 메인 브랜치 위에 변경 사항 rebase하기

작업 중인 변경사항이 있으며 이를 메인 브랜치의 최신 변경사항으로 업데이트하고 싶다면, 로컬의 메인 브랜치를 동기화한 후에 다음 명령어들을 실행합니다.

```bash
git checkout <branch-name>

# The rebase command will replay your changes on top of your local main branch
# If there are any merge conflicts you will need to resolve them now.
git rebase main

# Assuming there were new changes in main since you created your branch originally rebase will rewrite the commits and
# as such will require you to do a force push to your fork which is why the '-f' option is passed.
git push origin <branch-name> -f
```

_Tip_: 변경 사항을 합치고자 한다면, -i 옵션을 rebase 명령어에 추가하여 대화형 모드로 전환하고 합치고자 하는 커밋을 선택할 수 있습니다. 대화형 모드에 대한 정보는 [관련 문서](https://www.git-scm.com/docs/git-rebase#_interactive_mode)를 참고하십시오. 

## Feature 브랜치

독립적인 작업을 위해 사용자들은 메인 브랜치로부터 브랜치를 생성하고 그것들을 본인의 로컬 또는 포크된 레포지토리에 유지해야 합니다. 이러한 브랜치의 이름은 `feature` 명명 규칙을 따를 필요는 없고 대신 사람들이 만드는 일련의 변경 사항에 맞춰야 합니다. 작업이 준비되면 변경 사항은 메인 위에 리베이스되어야 하며, 메인 레포지토리에 pull request가 제출되어야 합니다.

메인 브랜치에 합쳐지기 전에 동일한 변경 사항들에 대해 여러 명이 협업해야 하는 경우, 공유를 위해 feature 브랜치를 메인 레포지토리에 푸시할 수 있습니다. 협업자들은 해당 브랜치에 변경 사항을 함께 푸시하거나, 메인 브랜치로 이동할 준비가 될 때까지 그것에 대한 pull request를 제출할 수 있습니다. 또한, feature 작업이 준비되면 변경 사항은 메인 위에 리베이스되어야 하며, 메인 레포지토리의 메인 브랜치에 pull request를 제출해야 합니다. 마지막으로 feature 작업에 대한 pull request가 완료되면, 해당 브랜치는 메인 레포지토리에서 삭제되어야 합니다.

메인 레포지토리에 푸시된 feature 브랜치는 `feature/<feature name>`과 같이 명명되어야 합니다.

## Release 태깅

각 패키지를 릴리즈할 때마다, 해당 패키지의 이름과 버전을 포함하는 고유한 git 태그가 생성됩니다. 이 태그는 패키지를 생성한 코드의 커밋을 표시하는데 사용됩니다. 이 태그는 Hotfix 브랜치를 통한 서비스 제공 및 특정 버전의 코드 디버깅에 사용됩니다.

태그의 형식은 `<package-name>_<package-version>`이어야 합니다.

**_Note:_** Release 태그는 변경되지 않아야 합니다. 푸시한 후에는 업데이트하거나 삭제하지 않아야 합니다. 예외적인 경우에 태그를 업데이트하거나 삭제해야 하는 경우, 엔지니어링 시스템 팀에 연락하여 가능한 대안에 대해 논의하시기 바랍니다.

## Release 브랜치

우선 순위에 따라 다음 세 가지 유형의 Release 브랜치가 존재할 수 있습니다:

- `main`
- `release/<release name>`
- `feature/<feature name>`

대부분의 Release가 메인에서 직접 이루어지므로, 일반적으로 Release 브랜치를 별도로 만들 필요는 없습니다. 그러나 경우에 따라 패키지를 안정화시키기 위해 브랜치를 잠그는 작업이 필요할 수 있습니다. 이런 상황에서는 `release/<release name>`라는 이름의 Release 브랜치를 생성하여 메인 레포지토리에 푸시합니다. 이런 방식을 채택하는 이유는 메인 브랜치가 다른 작업을 처리하는 것을 방해하지 않기 위해서입니다. 모든 변경이 완료되고 Release 빌드가 생성된 후에는, 해당 브랜치는 메인으로 병합(Rebase가 아님)되어야 합니다. 이때 Release 커밋에 달린 태그도 포함되어야 합니다. 이 과정이 완료되면 Release 브랜치는 삭제되어야 합니다.

메인 브랜치나 Release 브랜치 외부의 다른 브랜치에서 패키지의 베타 버전을 Release해야 하는 경우도 있을 수 있습니다. 이러한 경우에는 메인 레포지토리의 Feature 브랜치를 사용해야 합니다 (Feature 브랜치에서 안정된 Release를 진행하지 않아야 하므로, 버전이 베타로 올바르게 표시되었는지 확인해야 합니다). Release는 동일한 방식으로 수행되어야 하며, Release 태그의 생성도 포함되어야 합니다. 유일한 차이점은 Release 빌드가 메인 대신 이 Feature 브랜치를 가리키게 될 것이라는 점입니다. 작업이 완료되면 이 Feature 브랜치는 일반적으로 권장되는 Feature 브랜치에 대한 Rebase 대신 메인에 병합되어야 합니다. 이를 통해 Feature 브랜치를 삭제한 후에도 메인에서 Release 태그를 유지할 수 있습니다.

메인 외부에서 Release를 진행할 경우, 버전 번호가 정확하며 메인 또는 동일한 패키지를 Release하게 될 수 있는 다른 브랜치와 충돌하지 않도록 유의해야 합니다.

## Hotfix 브랜치

특정 버전의 코드에 Hotfix를 적용해야 하는 경우가 있을 수 있습니다. 이런 경우에는 `hotfix/<hotfix name>`의 형태로 브랜치 이름을 생성해야 합니다. 여기서 `<hotfix name>`은 최소한 패키지나 서비스의 이름과 간단한 설명 또는 버전 번호를 포함해야 합니다. 이 브랜치는 Hotfix를 적용하고자 하는 특정 버전을 가리키는 git release 태그로부터 생성되어야 하며, 메인 레포지토리에 푸시되어야 합니다.

```bash
# If you need help finding the exact case-sensitive tag name:
git tag -l <package-name>*

git checkout -b hotfix/<hotfix name> <package-name>_<package-version>
git push upstream hotfix/<hotfix name>
```

Hotfix 브랜치가 생성된 후에는 평소에 사용하던 작업 프로세스(즉, 새 브랜치를 생성하거나 포크에서 동일한 이름의 브랜치에서 작업하는 등)를 이용해야 합니다. 변경사항을 만들고 패키지에 대한 [버전 관리 지침](releases.md#package-versioning)에 따라 버전 번호를 증가시켜야 합니다. 만약 수정사항이 이미 메인 브랜치나 다른 브랜치에 있다면, 작업 중인 브랜치로 해당 사항들을 체리픽(`git cherry-pick <sha>`)할 수 있고, 그렇지 않다면 필요한 코드 수정을 진행해야 합니다. 모든 변경사항이 완료되면 메인 레포지토리에서 생성한 `hotfix/<hotfix name>` 브랜치로 Pull Request를 제출해야 합니다. 브랜치가 `hotfix/*` 명명 규칙을 따르면, Pull Request 검증은 자동으로 시작됩니다. CI가 성공적으로 수행되면, 수정된 사항을 `hotfix/<hotfix name>` 브랜치로 병합할 수 있습니다.

변경사항이 `hotfix/<hotfix name>` 브랜치로 병합된 후, 메인에서 사용하는 것과 동일한 Release 프로세스를 사용하여 해당 브랜치에서 Release를 생성할 수 있습니다. 그러나 빌드를 대기열에 추가할 때 브랜치 이름을 반드시 `hotfix/<hotfix name>`로 설정해야 합니다.

만약 변경사항이 main에서 체리픽(cherry-pick)되지 않았고, main에서 필요하다면 `hotfix/<hotfix name>` 브랜치에서 `main`으로 병합(`git merge hotfix/<hotfix name>`)해야 합니다. 병합할 때는 `main`의 버전 번호를 유지하고, CHANGELOG 항목이 날짜 그리고 버전별로 정렬되어 있는지 확인해야 합니다.

Hotfix가 배포되고 변경 사항이 `main`으로 병합되면, `hotfix/<hotfix name>` 브랜치를 삭제해야 합니다. 필요한 경우, 나중에 마지막 Release 태그로부터 항상 재생성할 수 있습니다.
