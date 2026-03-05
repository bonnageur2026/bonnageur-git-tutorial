# Git/GitHub 실전 튜토리얼

> **대상 독자:** Git을 처음 쓰는 프로그래밍 입문자
>
> **목표:** 내 컴퓨터(로컬)에서 Git으로 버전관리하고, 인터넷의 GitHub(원격)에 올리고/내려받고, 브랜치로 작업을 분리하고, 필요한 파일만 정확히 추적하는 전체 흐름을 "명령어 + 이유 + 사용법" 형태로 정리합니다.
>
> macOS/Linux 기준 예시이며 Windows도 Git Bash/PowerShell에서 거의 동일합니다.

---

## 0. 핵심 개념 10초 정리

- **작업 디렉토리(Working Directory):** 실제 파일이 있는 폴더
- **스테이징 영역(Staging Area):** "다음 커밋에 포함할 변경"을 모아두는 곳 (Index라고도 부름)
- **커밋(Commit):** 스냅샷(버전). 되돌리기/비교의 기준점
- **브랜치(Branch):** 작업 줄기. 기능별/실험별로 분리
- **원격(Remote):** GitHub 같은 서버 저장소(보통 `origin`)
- **푸시(Push):** 로컬 → GitHub 업로드
- **풀(Pull):** GitHub → 로컬로 가져오며(= fetch + merge/rebase) 반영
- **클론(Clone):** 원격 저장소를 통째로 처음 내려받아 로컬 저장소 만들기

---

## 1. 설치/초기 설정 (딱 한 번)

### 1.1 Git 설치 확인

```bash
git --version
```

### 1.2 사용자 정보 (커밋 기록에 찍힘)

```bash
git config --global user.name "당신이름"
git config --global user.email "you@example.com"
```

### 1.3 기본 편집기/줄바꿈 (선택)

```bash
git config --global core.editor "vim"   # 또는 "emacs", "code --wait"
```

### 1.4 설정 확인

```bash
git config --global --list
```

---

## 2. 디렉토리 만들고 Git 시작하기 (로컬 저장소 만들기)

### 2.1 새 프로젝트 폴더 만들기

```bash
mkdir myproject
cd myproject
```

### 2.2 Git 저장소 초기화 (로컬)

```bash
git init
```

**왜 쓰나요?**
이 폴더가 "버전관리 대상"이 됩니다. 내부에 `.git/`(히스토리 DB)이 생성됩니다.

### 2.3 현재 상태 확인 (가장 자주 쓰는 명령어)

```bash
git status
```

### 2.4 .gitignore 먼저 만들기 (권장)

프로젝트를 시작하자마자 `.gitignore`를 설정하면, 나중에 이미 추적 중인 파일을 제외하는 번거로움을 피할 수 있습니다.

```bash
cat > .gitignore << 'EOF'
# macOS
.DS_Store

# Python
__pycache__/
*.pyc

# VSCode
.vscode/

# build artifacts
dist/
build/
EOF
```

```bash
git add .gitignore
git commit -m "Add .gitignore"
```

> **Tip:** 이미 추적 중인 파일을 나중에 무시하려면 아래처럼 추적 해제가 필요합니다.
> ```bash
> git rm -r --cached dist
> git commit -m "Stop tracking dist"
> ```

---

## 3. 파일 저장하고(추적하고) 버전 만들기 (add/commit)

### 3.1 파일 생성

```bash
echo "# My Project" > README.md
```

### 3.2 변경 사항 확인

```bash
git status
git diff
```

### 3.3 스테이징 (add)

```bash
git add README.md
```

**왜 스테이징 영역이 있나요?**
작업 중 바뀐 파일이 여러 개여도 "이번 커밋에 넣을 것만" 골라 담을 수 있습니다.

### 3.4 커밋 (commit)

```bash
git commit -m "Add README"
```

### 3.5 커밋 로그 보기

```bash
git log --oneline --decorate --graph
git log --oneline -5    # 최근 5개만 보기
```

---

## 4. 내 컴퓨터에서 GitHub 인증

> GitHub에 push하려면 먼저 인증이 되어 있어야 합니다. 원격 연결 전에 설정해두세요.

### 4.1 HTTPS 방식

요즘은 비밀번호 대신 **Personal Access Token(PAT)** 을 사용합니다.
GitHub Settings → Developer settings → Personal access tokens에서 생성한 뒤, push할 때 비밀번호 대신 입력합니다. 한 번 인증하면 OS 키체인에 저장되기도 합니다.

### 4.2 SSH 방식 (개발자들이 많이 선호)

키 생성:

```bash
ssh-keygen -t ed25519 -C "you@example.com"
```

공개키 확인:

```bash
cat ~/.ssh/id_ed25519.pub
```

그 내용을 GitHub Settings → SSH keys에 등록 후, 원격 URL을 SSH로 설정합니다:

```bash
git remote set-url origin git@github.com:USER/REPO.git
```

---

## 5. 원격 GitHub 저장소 연결 (push 준비)

> 경우는 2가지입니다.
> - **A안:** 로컬에서 먼저 만들고 → GitHub에 올리기
> - **B안:** GitHub에 이미 있는 저장소를 → 로컬로 내려받기(clone)

---

## 6. A안: 로컬에서 만든 프로젝트를 GitHub에 올리기 (처음 1회)

### 6.1 GitHub에서 "빈 저장소" 만들기

GitHub에서 New repository를 생성합니다.

**주의:** 이미 로컬에 README가 있으면 GitHub 생성 시 README 자동 생성은 꺼두는 편이 충돌이 적습니다.

### 6.2 원격(origin) 등록

```bash
git remote add origin https://github.com/USER/REPO.git
```

원격 확인:

```bash
git remote -v
```

### 6.3 기본 브랜치 이름 정리 (main 권장)

```bash
git branch -M main
```

### 6.4 첫 푸시(push) + 업스트림 설정

```bash
git push -u origin main
```

**왜 `-u`가 중요하나요?**
이후부터는 `git push`만 쳐도 기본 대상이 자동으로 `origin main`으로 잡힙니다.

---

## 7. B안: GitHub 저장소를 내 컴퓨터로 내려받기 (clone)

### 7.1 클론 (처음 1회)

```bash
git clone https://github.com/USER/REPO.git
cd REPO
```

**clone이 해주는 것:**
- 원격 내용을 내려받아 로컬 저장소를 만들고
- `origin` 원격이 자동 등록되며
- 기본 브랜치(main/master) 체크아웃까지 됩니다

---

## 8. 브랜치 만들기/이동하기 (가장 중요한 작업 분리법)

### 8.1 브랜치 목록 보기

```bash
git branch          # 로컬 브랜치만
git branch -a       # 원격 브랜치 포함
```

### 8.2 새 브랜치 만들고 이동 (권장 1줄)

```bash
git switch -c feature/login
```

구버전 호환 명령:

```bash
git checkout -b feature/login
```

### 8.3 브랜치 이동

```bash
git switch main
git switch feature/login
```

### 8.4 브랜치 삭제 (로컬)

```bash
git branch -d feature/login     # 병합된 브랜치만 삭제
git branch -D feature/login     # 강제 삭제
```

### 8.5 원격 브랜치에 올리기 (처음 1회)

```bash
git push -u origin feature/login
```

---

## 9. 파일 올리기/내려받기: push / pull / fetch

### 9.1 내 변경사항을 GitHub로 올리기 (push)

```bash
git add .
git commit -m "Implement login"
git push
```

### 9.2 GitHub 변경사항을 내 컴퓨터로 가져와 반영하기 (pull)

```bash
git pull
```

`git pull` = `git fetch`(가져오기) + merge 또는 rebase(내 작업에 합치기)

### 9.3 안전하게 가져오기만(fetch) 하고 싶을 때

```bash
git fetch origin
```

가져온 뒤 차이 확인:

```bash
git log --oneline --decorate --graph --all
git diff main..origin/main
```

그 다음 반영(merge 예시):

```bash
git merge origin/main
```

---

## 10. 디렉토리(폴더) 추적에 대한 중요한 사실

### 10.1 Git은 "빈 폴더"를 추적하지 않습니다

Git은 폴더 자체가 아니라 **파일**을 추적합니다.
빈 폴더를 유지하고 싶다면 보통 이렇게 합니다:

```bash
mkdir data
touch data/.gitkeep
git add data/.gitkeep
git commit -m "Keep data directory"
```

---

## 11. 브랜치를 main에 합치기 (merge)

### 11.1 feature 브랜치 작업 완료 후

```bash
git switch main
git pull
git merge feature/login
git push
```

### 11.2 충돌(conflict)이 나면

충돌이 발생하면 해당 파일에 아래와 같은 표시가 생깁니다:

```
<<<<<<< HEAD
내 코드 (현재 브랜치의 내용)
=======
상대 코드 (merge 대상 브랜치의 내용)
>>>>>>> feature/login
```

이 표시를 직접 편집해서 원하는 코드만 남긴 뒤:

```bash
git add 충돌해결한파일
git commit
git push
```

---

## 12. 자주 쓰는 되돌리기/복구 명령어 (실전 필수)

### 12.1 스테이징 취소 (add 취소)

```bash
git restore --staged README.md
```

### 12.2 작업 디렉토리 변경 취소 (수정 되돌리기)

```bash
git restore README.md
```

### 12.3 마지막 커밋 메시지만 수정 (내용 그대로)

```bash
git commit --amend -m "Better message"
```

### 12.4 특정 커밋으로 파일만 되돌리기

```bash
git restore --source <commit_hash> -- path/to/file
```

### 12.5 커밋 자체를 "되돌리는 커밋" 만들기 (안전)

```bash
git revert <commit_hash>
```

공개된 브랜치(main)에 이미 push한 커밋을 없애고 싶을 때는 **revert가 가장 안전**합니다 (히스토리 강제 변경 없음).

### 12.6 작업 임시 저장 (stash)

브랜치를 바꿔야 하는데 아직 커밋하기엔 이른 변경사항이 있을 때:

```bash
git stash              # 현재 변경사항 임시 저장
git switch main        # 다른 브랜치로 이동
# ... 급한 작업 처리 ...
git switch feature/x   # 원래 브랜치로 복귀
git stash pop          # 임시 저장한 변경사항 복원
```

임시 저장 목록 보기:

```bash
git stash list
```

---

## 13. 원격에서 최신 브랜치 목록/정리 (협업 시 유용)

원격 브랜치 조회:

```bash
git fetch --all
git branch -r
```

원격에서 삭제된 브랜치 정리:

```bash
git fetch --prune
```

---

## 14. 추천 워크플로우

### 14.1 혼자 작업 (가장 단순)

1. main에서 새 브랜치

```bash
git switch main
git pull
git switch -c feature/x
```

2. 작업 → 커밋

```bash
git add .
git commit -m "Work on x"
```

3. 원격에 올리기

```bash
git push -u origin feature/x
```

4. main에 합치기

```bash
git switch main
git pull
git merge feature/x
git push
```

### 14.2 팀 작업 (권장 흐름)

feature 브랜치 → GitHub Pull Request(PR) → 코드리뷰 → main에 merge

명령어는 위와 동일하고, "merge 결정"을 GitHub UI에서 하게 됩니다.

---

## 15. 체크리스트

**디렉토리 만들기**
- 로컬 폴더: `mkdir`, `cd`
- Git이 폴더만 추적 안 함 → `.gitkeep` 같은 파일 필요할 수 있음

**브랜치 만들기**
- 생성+이동: `git switch -c 브랜치명`
- 원격에도 만들기: `git push -u origin 브랜치명`

**파일 저장/올리기**
- 변경 확인: `git status`, `git diff`
- 스테이징: `git add ...`
- 버전 생성: `git commit -m "..."`
- 업로드: `git push`

**파일 꺼내기/내려받기**
- 처음 전체 내려받기: `git clone ...`
- 최신 반영: `git pull`
- 가져오기만: `git fetch`

---

## 16. 최소 치트시트 (진짜 자주 쓰는 것만)

```bash
git status
git diff
git add .
git commit -m "message"
git log --oneline --graph --decorate
git push
git pull

git switch -c feature/x
git switch main
git merge feature/x

git stash
git stash pop
```
