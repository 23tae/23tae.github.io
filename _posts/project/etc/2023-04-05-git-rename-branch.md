---
title: "Git 브랜치 이름 변경 방법"
date: 2023-04-05
categories: [Project, etc_Project]
tags: [git]
---

# 개요
git의 브랜치명 변경에는 로컬 브랜치명 변경 뿐 아니라 remote 저장소와 관련한 재설정이 필요하다.

# 변경 방법
![Untitled](/assets/img/project/git-rename-branch/rename_original.png)

이름이 `old_name`인 브랜치를 `new_name` 으로 변경한다고 하자.

1. **로컬** 브랜치 이름 변경 (다른 브랜치로 이동하여 진행해야 한다.)
    
    ```bash
    git branch -m old_name new_name
    ```
    
2. **remote**에 있는 기존 브랜치 삭제
    
    ```bash
    git push origin --delete old_name
    ```
        
3. 변경한 브랜치로 이동
    
    ```bash
    git checkout new_name
    ```

4. **upstream** 브랜치 업데이트
    
    ```bash
    git push -u origin new_name
    ```

    - upstream 브랜치란?
        - 로컬 브랜치가 **추적(tracking)**하는 remote 저장소의 브랜치
        - Git은 로컬 브랜치와 remote 브랜치를 **연결**해둔다.
        - 이로 인해 `git push`나 `git pull` 명령어를 실행할 때, Git은 remote 저장소의 어느 브랜치에 변경사항을 반영할 것인지를 알 수 있다.
    - 참고. **upstream 브랜치를 확인**하는 방법
        
        ```bash
        git branch -vv
        ```

# 유의 사항

- 브랜치 이름을 변경하기 전에 모든 변경사항을 commit하고 **remote 저장소에 push** 해두어야 한다.
- 변경 전 브랜치로 작업하는 사람이 있는 경우, 해당 로컬에서도 아래 과정을 통해 **브랜치명을 업데이트** 해주어야 한다.
    ![Untitled](/assets/img/project/git-rename-branch/rename_other.png)
    1. 로컬 브랜치명 변경 (다른 브랜치에서 실행되어야 한다.)
        
        ```bash
        git branch -m old_name new_name
        ```
        
    2. git fetch
        
        ```bash
        git fetch origin
        ```
        
    3. 변경한 브랜치로 이동
        
        ```bash
        git checkout new_name
        ```
        
    4. upstream 브랜치 재설정
        
        ```bash
        git branch --unset-upstream
        ```
        
        ```bash
        git branch -u origin/new_name
        ```
            

# 참고

- [Git Rename Branch – How to Change a Local Branch Name](https://www.freecodecamp.org/news/git-rename-branch-how-to-change-a-local-branch-name/)
- [repository - How do I rename both a Git local and remote branch name? - Stack Overflow](https://stackoverflow.com/questions/30590083/how-do-i-rename-both-a-git-local-and-remote-branch-name)
- [How to Set or Change Upstream Branch in Git](https://phoenixnap.com/kb/git-set-upstream)