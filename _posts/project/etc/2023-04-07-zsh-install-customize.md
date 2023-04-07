---
title: "Zsh 설치 및 커스텀 (Oh My Zsh)"
date: 2023-04-07
categories: [Project, etc_Project]
tags: [zsh]
---

![Untitled](/assets/img/project/zsh-install-custom/Untitled.png)

# Zsh

## zsh 설치

- for macOS ([Homebrew](https://brew.sh/) 필요)
    - Homebrew 설치 (기존에 설치되지 않은 경우)
        
        ```bash
        /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
        ```
        
    
    ```bash
    brew install zsh
    ```
    
- for Linux (Ubuntu, Debian)
    
    ```bash
    sudo apt get update && \
    sudo apt install -y zsh
    ```
    

## 기본 Shell 변경

- 방법 1. chsh 사용
    
    ```bash
    chsh -s $(which zsh)
    ```
    
- 방법 2. passwd 파일 수정
    
    ```bash
    sudo vi /etc/passwd
    ```
    
    - 다음과 같이 유저의 기본 shell이 `/bin/bash`로 되어 있는 부분을 `/bin/zsh`로 변경한다.
    
    ```
    ...
    ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
    lxd:x:999:100::/var/snap/lxd/common/lxd:/bin/false
    taehooki:x:1001:1001:,,,:/home/taehooki:/bin/bash
    ```
    
- logout 후 login
    - `.zshrc` 관련 설정창이 나오면 2(Recommended) 입력

# Oh my zsh

## oh my zsh 설치

```bash
sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## 테마 적용 (powerlevel10k)

- git clone
    
    ```bash
    git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
    ```
    
- `.zshrc` 파일의 변수 수정
    
    ```bash
    ZSH_THEME="powerlevel10k/powerlevel10k"
    ```
    
- 변경사항 적용
    
    ```bash
    source ~/.zshrc
    ```
    
- 적용을 하면 테마 세팅하는 화면이 나온다. 나오지 않는다면 `p10k configure` 입력.
- pc에 해당 테마가 사용하는 폰트를 설치해야한다.
    - [Meslo 폰트 설치](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf)

## 플러그인 적용

git, git-auto-fetch 등 oh-my-zsh에 기존에 존재하는 플러그인은 `.zshrc`파일에 추가해주는 것만으로 설정이 끝나지만 나머지는 직접 설치해주어야 한다. ([링크](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins)에서 확인 가능)

### 플러그인 설치

- zsh-syntax-highlighting
    
    ![Untitled](/assets/img/project/zsh-install-custom/Untitled%201.png)
    
    - 유효한 명령어인지를 확인할 수 있는 플러그인
    - git clone
        
        ```bash
        git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
        ```
        
    - zshrc 추가
        
        ```bash
        echo "source ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" >> ${HOME}/.zshrc
        ```
        
- zsh-autosuggestions
    
    ![Untitled](/assets/img/project/zsh-install-custom/Untitled%202.png)
    
    - history를 기반으로 명령어를 추천해주는 플러그인
    - git clone
        
        ```bash
        git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
        ```
        

### 플러그인 설정

- `~/.zshrc` 을 수정한다.
- 기본적으로 `plugins=(git)`으로만 설정되어있는 부분을 아래와 같이 변경한다.
    
    ```
    plugins=(
        git
        git-auto-fetch
        zsh-autosuggestions
    )
    ```
    
- 변경사항 적용
    
    ```bash
    source ~/.zshrc
    ```

# 참고

- [ohmyzsh/ohmyzsh](https://github.com/ohmyzsh/ohmyzsh)
- [romkatv/powerlevel10k: A Zsh theme](https://github.com/romkatv/powerlevel10k)
- [command line - How to make ZSH the default shell? - Ask Ubuntu](https://askubuntu.com/questions/131823/how-to-make-zsh-the-default-shell)
- [Customize Oh My Zsh with Syntax Highlighting and Auto-Suggestions \| HackerNoon](https://hackernoon.com/customize-oh-my-zsh-with-syntax-highlighting-and-auto-suggestions-6q1b3w8o)
