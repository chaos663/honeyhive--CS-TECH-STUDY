---
layout: post
title: "DevCulture: Git 워크플로우 마스터 — 브랜치 전략과 협업"
date: 2026-03-16 17:00:40 +0900
excerpt: "Git 워크플로우는 팀의 규모, 프로젝트의 특성, 배포 주기에 따라 다양한 형태로 발전합니다. 이 포스트에서는 가장 널리 사용되는 Git Flow와 최근 각광받는 Trunk-based Development(TBD)를 심층적으로 비교하고, 각 워크플로우의 장단점 및 실제 적용 시 고려사항을 다룹니다. 또한, 코드 리뷰의 중요성, Conventional Commits를 통한 커밋 메시지 표준화, 그리고 Rebase와 Merge의 전략적 활용법을 구"
tags: [CS, cs-study]
image: "/assets/images/thumbnails/2026-03-16-devculture-git-워크플로우-마스터-브랜치-전략과-협업.svg"
series: "HoneyByte"
---
> 효율적인 Git 브랜치 전략으로 팀 협업을 극대화하고 견고한 개발 워크플로우를 구축합니다.

## 핵심 요약 (TL;DR)

Git 워크플로우는 팀의 규모, 프로젝트의 특성, 배포 주기에 따라 다양한 형태로 발전합니다. 이 포스트에서는 가장 널리 사용되는 Git Flow와 최근 각광받는 Trunk-based Development(TBD)를 심층적으로 비교하고, 각 워크플로우의 장단점 및 실제 적용 시 고려사항을 다룹니다. 또한, 코드 리뷰의 중요성, Conventional Commits를 통한 커밋 메시지 표준화, 그리고 Rebase와 Merge의 전략적 활용법을 구체적인 코드 예제와 함께 제시하여 현업 개발자들이 견고하고 효율적인 Git 협업 환경을 구축하는 데 필요한 모든 지식을 제공합니다.

## 왜 알아야 하는가?

Git은 현대 소프트웨어 개발의 핵심 도구이지만, 그 강력함만큼이나 복잡한 협업 시나리오를 만들어낼 수 있습니다. "우리 팀은 어떤 브랜치 전략을 사용해야 할까?", "PR(Pull Request) 리뷰는 어떻게 효율적으로 진행해야 할까?", "Rebase와 Merge 중 어떤 것을 선택해야 할까?"와 같은 질문들은 모든 개발 팀이 마주하는 고민입니다. 잘못된 워크플로우는 불필요한 충돌, 배포 지연, 심지어 코드 품질 저하로 이어질 수 있습니다. 이 포스트는 이러한 질문에 대한 명확한 답변과 실용적인 가이드를 제공하여 팀 생산성 향상과 안정적인 서비스 운영에 기여합니다.

## 개념 이해

Git 워크플로우는 Git을 활용하여 코드 변경 사항을 관리하고 팀원 간 협업하는 방식에 대한 일련의 규칙과 절차입니다. 주로 브랜치 전략, 커밋 메시지 규칙, 코드 리뷰 절차 등으로 구성됩니다.

### Git Flow
Git Flow는 Vincent Driessen이 제안한 브랜치 모델로, 장기적인 릴리즈와 피처 개발에 적합한 복잡하지만 체계적인 구조를 제공합니다. `master`, `develop`, `feature`, `release`, `hotfix` 등 여러 개의 브랜치를 활용하여 안정적인 배포와 병렬 개발을 지원합니다.

### Trunk-based Development (TBD)
Trunk-based Development는 모든 개발자가 `main` 또는 `trunk`라고 불리는 단일 브랜치에 자주 통합(integrate)하는 방식입니다. CI/CD(지속적 통합/지속적 배포) 환경에 최적화되어 있으며, 짧은 개발 주기를 가진 서비스나 마이크로서비스 아키텍처에서 빠르게 배포하고 변경 사항을 반영하는 데 유리합니다. 브랜치 생성 및 병합 오버헤드가 적어 개발 속도가 빠르지만, 엄격한 테스트와 코드 리뷰가 뒷받침되어야 합니다.

### Conventional Commits
Conventional Commits는 커밋 메시지에 대한 경량의 규약으로, `feat:`, `fix:`, `docs:`, `chore:` 등 특정 타입을 접두사로 사용하여 커밋의 의도를 명확히 합니다. 이는 변경 이력을 쉽게 파악하고, 자동화된 릴리즈 노트를 생성하며, 프로젝트의 버전을 관리하는 데 큰 도움을 줍니다.

```mermaid
graph TD
    A[개발 시작] --> B(Feature 브랜치 생성: feature/my-new-feature)
    B --> C{코드 개발 및 커밋}
    C --> D(Pull Request 생성)
    D --> E{코드 리뷰}
    E -- 승인 --> F(Merge to develop or main)
    F --> G(테스트 및 배포)
    G --> H[다음 개발]

    subgraph Git Flow (long-lived branches)
        master -- (stable) --> develop
        develop -- (new features) --> feature-branch
        develop -- (release prep) --> release-branch
        master -- (bug fix) --> hotfix-branch
    end

    subgraph Trunk-based Development (short-lived branches)
        main -- (daily integrate) --> feature-branch-short-lived
        feature-branch-short-lived --> main
    end
```

## 환경 설정

Git은 대부분의 개발 환경에 기본적으로 설치되어 있습니다. 필요한 경우 다음 명령어를 사용하여 설치합니다.

```bash
# macOS (Homebrew)
brew install git

# Debian/Ubuntu
sudo apt-get install git

# Windows (Git Bash)
# git-scm.com 에서 설치 파일을 다운로드하여 설치
```

사용자 정보 설정은 필수입니다.

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
git config --global core.editor "vim" # 또는 선호하는 에디터 (예: code --wait)
```

Conventional Commits를 위한 커밋 템플릿을 설정할 수 있습니다. `~/.gitmessage` 파일을 생성하고 다음 내용을 추가합니다.

```
<type>(<scope>): <subject>

<body>

<footer>
```

이후 다음 명령어로 템플릿을 적용합니다.

```bash
git config --global commit.template ~/.gitmessage
```

## 구현

### Step 1: Git Flow 워크플로우 맛보기

Git Flow는 `git-flow` 확장 도구를 통해 쉽게 사용할 수 있습니다.

```bash
# git-flow 설치 (macOS)
brew install git-flow

# 새로운 Git Flow 저장소 초기화
git flow init -d # -d 옵션으로 기본 브랜치명 사용 (master, develop)
```

피처 브랜치에서 작업하고 develop으로 병합하는 예제입니다.

```bash
# 새로운 피처 개발 시작
git flow feature start my-awesome-feature

# (코드 수정 및 커밋)
git add .
git commit -m "feat: Add new user profile page"

# 피처 개발 완료 및 develop 브랜치로 병합
git flow feature finish my-awesome-feature
```

### Step 2: Trunk-based Development (TBD) 시뮬레이션

TBD는 별도의 도구 없이 Git의 기본 명령어로 운영됩니다. 핵심은 `main` 브랜치에 자주, 작게 커밋을 통합하는 것입니다.

```bash
# main 브랜치에서 최신 변경사항 가져오기
git checkout main
git pull origin main

# 짧은 기능 브랜치 생성
git checkout -b feature/small-task

# (코드 수정 및 커밋)
git add .
git commit -m "feat: Implement quick login fix"

# Pull Request (또는 Merge Request) 생성 후 리뷰
# (리뷰 후) main 브랜치로 병합
git checkout main
git merge feature/small-task --no-ff # --no-ff로 merge commit 남기기
git branch -d feature/small-task # 기능 브랜치 삭제
git push origin main
```

### Step 3: Rebase vs Merge의 전략적 선택

Rebase와 Merge는 브랜치 이력을 통합하는 두 가지 주요 방법입니다.

**Merge 예제:**

```bash
git checkout feature/new-design
git merge main # main 브랜치의 변경사항을 feature 브랜치로 가져옴
```
`git merge`는 새로운 병합 커밋을 생성하여 브랜치들이 언제 병합되었는지 기록을 남깁니다.

**Rebase 예제:**

```bash
git checkout feature/new-design
git rebase main # feature 브랜치의 모든 커밋을 main 브랜치의 최신 커밋 위로 재배치
```
`git rebase`는 커밋 이력을 선형적으로 유지하여 깔끔하게 보이지만, 기존 커밋의 해시를 변경하므로 이미 공유된 브랜치에서는 사용하지 않는 것이 좋습니다.

## 실행 및 테스트

Git 워크플로우 자체는 "실행"보다는 "규칙 준수"에 가깝습니다. 하지만 `git log` 명령어를 통해 커밋 이력을 확인하여 워크플로우가 잘 적용되었는지 검증할 수 있습니다.

```bash
# 모든 브랜치의 커밋 이력을 그래프 형태로 확인
git log --all --decorate --oneline --graph
```

Conventional Commits 규칙 준수 여부는 `git log` 메시지를 통해 수동으로 확인하거나, `commitlint`와 같은 도구를 사용하여 자동화할 수 있습니다.

## 설계 포인트

Git 워크플로우 선택 시 다음 설계 원칙을 고려해야 합니다.

| 접근법 | 장점 | 단점 | 적합한 상황 |
|--------|------|------|------------|
| **Git Flow** | 안정적인 릴리즈 관리, 명확한 역할 분리 | 복잡한 브랜치 관리, 병합 충돌 가능성 증가 | 장기 프로젝트, 정기적인 릴리즈 주기, 엄격한 버전 관리 필요 |
| **Trunk-based Development** | 빠른 배포, 지속적인 통합, 간단한 브랜치 전략 | 빈번한 테스트 및 코드 리뷰 필수, 작은 단위 개발 강제 | 애자일 개발, 마이크로서비스, SaaS, CI/CD 환경 |
| **Rebase** | 깔끔하고 선형적인 커밋 이력 | 이미 공유된 커밋에 적용 시 위험, 복잡한 충돌 해결 | 개인 작업 브랜치 정리, Merge PR 전에 `main`에 rebase |
| **Merge** | 변경 이력 보존, 비파괴적 | 병합 커밋으로 이력 복잡해질 수 있음 | 팀 내 공유 브랜치 통합, PR 병합 기본 전략 |

## 주의사항 & 흔한 실수

*   **공유된 브랜치에 Rebase 금지**: 이미 `push`하여 팀원들과 공유된 브랜치에 `rebase`를 하면 다른 팀원들의 이력이 꼬일 수 있습니다. 이는 "황금 규칙"입니다.
*   **Merge Conflict 방치**: 병합 충돌을 해결하지 않고 넘어가는 것은 심각한 문제를 야기합니다. 충돌 발생 시 즉시 해결하고 충분히 테스트해야 합니다.
*   **커밋 메시지 부실**: "fix bug", "update"와 같은 모호한 커밋 메시지는 나중에 변경 이력을 추적하기 어렵게 만듭니다. Conventional Commits와 같은 규칙을 따르세요.
*   **브랜치 관리 소홀**: 사용하지 않는 브랜치를 방치하면 저장소가 지저분해지고 혼란을 초래합니다. 기능 개발이 완료되면 브랜치를 삭제하는 습관을 들이세요.

## 관련 포스트

- [Git으로 협업하기: Pull Request 잘 쓰는 법](/2025/01/15/pull-request-best-practices/) — 효과적인 PR 작성 및 리뷰 가이드

## 레퍼런스

### 영상
- [A successful Git branching model (Original article)](https://nvie.com/posts/a-successful-git-branching-model/) — Vincent Driessen의 원문. Git Flow의 탄생 배경과 철학.
- [Trunk Based Development vs Git Flow: Which is best for you?](https://www.youtube.com/watch?v=AP9C9gJ6eY4) — Pluralsight의 Git Flow와 TBD 비교 영상.

### 문서 & 기사
- [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) — Conventional Commits의 공식 문서.
- [Atlassian Git tutorial: Rebase vs Merge](https://www.atlassian.com/git/tutorials/merging/rebase-vs-merge) — Atlassian의 Rebase와 Merge 비교 가이드.

---

*이 포스트는 [HoneyByte](https://blog.honeybarrel.co.kr) 시리즈의 일부입니다.*
