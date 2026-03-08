
---

## 📄 터미널 예쁘게 꾸미기 — Oh My Zsh

기본 터미널은 밋밋합니다. **Oh My Zsh**를 설치하면 터미널이 예쁘고 편해집니다! ✨

---

### 🎯 이번에 배울 것

| 순서 | 내용 |
| --- | --- |
| 1 | Oh My Zsh 설치 |
| 2 | 테마 바꾸기 |
| 3 | 인기 테마 Powerlevel10k 설치 |

---

## Part 1: Oh My Zsh 설치하기

### 한 줄이면 끝!

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

설치 후 터미널이 바로 달라집니다:

```
변경 전:  username@MacBook ~ %
변경 후:  ➜  ~
```

🎉 화살표가 생겼어요!

> 💡 **컴퓨터 공학 개념 — 설정 파일 (Configuration File)**
>
> Oh My Zsh는 홈 폴더의 **`~/.zshrc`** 파일로 설정을 관리합니다.
>
> ```
> ~/.zshrc  ← 터미널이 켜질 때마다 이 파일을 읽습니다
> ```
>
> `.`으로 시작하는 파일은 **숨김 파일(Hidden File)**입니다.
> Finder에서는 안 보이지만 `ls -a`로 볼 수 있어요!
>
> 많은 프로그램이 설정을 이런 **dotfile(점 파일)**로 관리합니다:
>
> | 파일 | 역할 |
> | --- | --- |
> | `~/.zshrc` | Shell(터미널) 설정 |
> | `~/.gitconfig` | Git 설정 |
> | `~/.ssh/config` | SSH 설정 |

---

## Part 2: 테마 바꾸기 🎨

### Step 1: 설정 파일 열기

```bash
nano ~/.zshrc
```

> `nano`는 터미널에서 쓰는 간단한 **텍스트 편집기**입니다.
> 저장: `Ctrl + O` → `Enter`, 종료: `Ctrl + X`

### Step 2: 테마 이름 바꾸기

파일 안에서 `ZSH_THEME` 줄을 찾으세요:

```bash
ZSH_THEME="robbyrussell"     # ← 기본 테마
```

원하는 테마 이름으로 바꿉니다:

```bash
ZSH_THEME="agnoster"         # 인기 1위! 파워라인 스타일
```

### Step 3: 적용하기

```bash
source ~/.zshrc
```

> 💡 **컴퓨터 공학 개념 — `source` 명령어**
>
> `source`는 설정 파일을 **다시 읽어서 적용**하는 명령어입니다.
>
> 터미널은 처음 열릴 때만 `~/.zshrc`를 읽기 때문에,
> 파일을 수정한 후에는 `source`로 "새로고침"해야 합니다.
> (또는 터미널을 껐다 켜도 됩니다!)

### 추천 테마 목록

| 테마 | 스타일 |
| --- | --- |
| `robbyrussell` | 기본 (심플) |
| `agnoster` | 파워라인 (인기!) |
| `avit` | 깔끔 |
| `refined` | 미니멀 |
| `af-magic` | 컬러풀 |

> 💡 전체 테마 목록: [Oh My Zsh 테마 갤러리](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)
> 직접 보고 마음에 드는 걸 골라보세요!

---

## Part 3: Powerlevel10k (가장 인기 있는 테마! 🏆)

더 예쁘고 정보가 풍부한 테마를 원하면 **Powerlevel10k**를 추천합니다.

### Step 1: 폰트 설치 (아이콘 표시에 필요)

```bash
brew install --cask font-meslo-lg-nerd-font
```

> Homebrew가 없다면:
> ```bash
> /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
> ```

> 💡 **컴퓨터 공학 개념 — 패키지 매니저 (Package Manager)**
>
> **Homebrew**는 macOS의 **패키지 매니저**입니다.
> 프로그램을 설치/삭제/업데이트를 명령어 한 줄로 해결합니다.
>
> ```bash
> brew install 프로그램이름      # 설치
> brew uninstall 프로그램이름    # 삭제
> brew update                  # 업데이트
> ```
>
> 앱스토어가 앱을 관리하듯, Homebrew는 **개발 도구**를 관리합니다.
> Linux에서는 `apt`(Ubuntu)나 `yum`(CentOS) 같은 패키지 매니저를 씁니다.

### Step 2: 터미널 폰트 변경

- **터미널.app**: 환경설정 → 프로파일 → 텍스트 → 서체 변경 → `MesloLGS NF` 선택
- **iTerm2**: Preferences → Profiles → Text → Font → `MesloLGS NF` 선택

### Step 3: Powerlevel10k 설치

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git \
  ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

### Step 4: 테마 변경

```bash
nano ~/.zshrc
```

`ZSH_THEME` 줄을 바꿉니다:

```bash
ZSH_THEME="powerlevel10k/powerlevel10k"
```

적용:

```bash
source ~/.zshrc
```

### Step 5: 설정 마법사

처음 실행하면 자동으로 **설정 마법사**가 나타납니다!

```
Does this look like a diamond? → y
Does this look like a lock? → y
...
```

취향에 맞게 선택하면 끝! 나중에 다시 설정하고 싶으면:

```bash
p10k configure
```

---

### 🛠️ 요약

```bash
# 1. Oh My Zsh 설치
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# 2. 테마 바꾸기 (간단)
nano ~/.zshrc
# ZSH_THEME="agnoster" 로 변경
source ~/.zshrc

# 3. Powerlevel10k (고급)
brew install --cask font-meslo-lg-nerd-font
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git \
  ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
nano ~/.zshrc
# ZSH_THEME="powerlevel10k/powerlevel10k" 로 변경
source ~/.zshrc
```

---

### 📚 학습 흐름

```
📘 가이드 1: Shell 기초 명령어       → 터미널 열기, 폴더 이동, 파일 관리
📘 가이드 2: 파일 보기 & 꿀팁       → 검색, 파이프, 단축키
📘 가이드 3: 터미널 꾸미기 (이번!)   → Oh My Zsh 설치, 테마 변경
```

---
