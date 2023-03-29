---
title: Github 블로그 생성 (macOS, Chirpy 테마)
date: 2023-03-29 21:09:00 +0900
categories: [ETC]
tags: [설치 가이드]
pin: true
comments: true
---

윈도우와 맥북에 모두 설치하는 데 약 이틀이 걸렸다. 나중에 다시 설치할 때를 대비해서 기록해 두려고 한다. 설치과정은 [Chirpy 공식 가이드](https://chirpy.cotes.page/posts/getting-started/)[^chirpy_guide] 및 각 프로그램 공식 가이드를 따랐다.

윈도우는 이것저것 설치하고 지우기를 반복하다보니 결과적으로 뭘 했는지 알 수가 없어서 기록하지 못했다.

## 0. 사전 환경설정
homebrew → git, ruby, jekyll gem, node.js 설치

### 0.1. Homebrew 설치[^homebrew_official]

```zsh
# homebrew 설치
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
# 이미 설치되어있는 경우 최신버전으로 업데이트
brew update
```

### 0.2. Git 설치[^git_official]

```zsh
# git 설치
brew install git
# git 설치 확인
git --version
```

### 0.3. Ruby 설치[^jekyll_official]
#### ruby 설치

```zsh
# chruby(ruby version manager) 설치
brew install chruby ruby-install xz
# ruby 설치 (supported by Jekyll version) 
ruby-install ruby 3.1.3
# ruby 설치 확인
ruby -v
```
#### chruby를 자동으로 사용하도록 설정(zsh)

```zsh
echo "source $(brew --prefix)/opt/chruby/share/chruby/chruby.sh" >> ~/.zshrc
echo "source $(brew --prefix)/opt/chruby/share/chruby/auto.sh" >> ~/.zshrc
echo "chruby ruby-3.1.3" >> ~/.zshrc
```
> bash는 `~/.zshrc` 를 `~/.bash_profile` 로 바꿔서 적용한다.
{: .prompt-tip }

#### Jekyll gem 설치

```zsh
gem install jekyll
```

### 0.4. node.js (+ npm) 설치[^nodejs_official]

```zsh
# node.js (+ npm) 설치
brew install node
# node, npm 설치 확인
node -v
npm -v
```

## 1. Chirpy 테마 fork
[fork **Chirpy**](https://github.com/cotes2020/jekyll-theme-chirpy/fork), and then rename it to `USERNAME.github.io` (USERNAME 부분을 깃허브 계정으로 바꿔서 생성)

## 2. fork한 프로젝트를 로컬 저장소로 clone

```zsh
git clone <remote_repository_url> <local_repository_path>
```

## 3. 프로젝트 초기화
### 3.1. 초기화 명령어 실행

```zsh
bash tools/init
```
**위 명령어 실행 시 변경사항**
1. 저장소 마지막 태그 확인  
2. 불필요한 샘플 파일, 깃허브 관련 파일 삭제  
3. js 파일 빌드 후 **`assets/js/dist/`{: .filepath}**에 저장 & Git에 추가  
4. 위 사항에 대한 커밋

### 3.2. 발생 에러
Already initialized.
: 프로젝트 루트 디렉터리 외 경로에서 (tools 폴더 내부 등) init 실행 시 출력된다.  
👉 .github 폴더가 있는 프로젝트 루트에서 진행한다.
    
fatal: No names found, cannot describe anything.
: `git describe --tags` 명령어에 의해 저장소에 tag가 없을 때 발생하는 오류이다.  
👉 tag를 생성해준다.

    ```zsh
git tag <tag_name>
    ```

## 4. **`_config.yml`** 파일 수정
`url`, `avatar`, `timezone`, `lang` 등 파일 내부 주석을 참고해서 작성한다.

## 5. Running Local Server
- **프로젝트 루트 디렉터리**에서 실행
- 원격서버에 업로드하기 전 확인하는 용도로 활용
- 접속 주소 : <http://127.0.0.1:4000/>

```zsh
# Installing Dependencies
bundle 
# Running local server
bundle exec jekyll s
```

## 6. GitHub Actions 를 이용한 Deploy
**진행 전 확인사항**
- 깃허브 무료 사용자라면 저장소가 public 상태인가?
- `_config.yml` 파일에 `url` 을 변경했는가?
- `Gemfile.lock` 파일을 commit 했는가?
- 로컬 서버가 꺼져있는가?  

위 사항에 대한 확인이 끝났다면 **lock-file 업데이트**를 진행한다. 루트 디렉터리에서 실행한다.

```zsh
bundle lock --add-platform x86_64-linux
```
    
## 7. **Pages** 서비스 설정
1. 블로그 github 사이트 > settings > pages > source section > 드롭다운에서 <kbd>github Actions</kbd> 선택
2. 프로젝트 commit & push > workflow 발생 > Actions 탭에서 _Build and Deploy_ running 상태 확인 > 빌드 성공 > 자동 Deploy

**⭐️이제 URL(https://USERNAME.github.io)로 접속 가능하다⭐️**

## 8. 포스트 작성 및 업로드
Markdown 작성법 및 아래 사이트를 참고하여 포스트를 작성한 후 `_posts` 디렉터리에 넣고 업로드하면 끝!
- [Writing a New Post](https://chirpy.cotes.page/posts/write-a-new-post/)
- [Text and Typography](https://chirpy.cotes.page/posts/text-and-typography/)

## 9. Reference
[^chirpy_guide]: [Chirpy 설치 안내](https://chirpy.cotes.page/posts/getting-started/)
[^homebrew_official]: [Homebrew 설치 안내](https://brew.sh/index_ko)
[^git_official]: [Git 설치 안내](https://git-scm.com/download/mac)
[^jekyll_official]: [jekyll 환경설정 가이드](https://jekyllrb.com/docs/installation/)
[^nodejs_official]: [node.js 설치 파일 다운로드](https://nodejs.org/en)
