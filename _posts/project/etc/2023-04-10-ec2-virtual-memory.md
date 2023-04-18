---
title: "[AWS EC2] 가상 메모리 설정"
date: 2023-04-10
categories: [Project, etc_p]
tags: [aws, ec2, issue]
---

# 개요

AWS EC2의 t2.micro를 사용하면 인스턴스의 메인 메모리는 1GB가 된다. 부족한 메모리로 작업을 하며 인스턴스가 다운되는 등의 **메모리 문제**를 겪던 중 **스왑 공간(swap space)**을 통해 문제를 해결하였다.

# 스왑 공간 (swap space)

스왑 공간을 이해하기 위해서는 우선 가상 메모리에 대해 알아야 한다.

- 가상 메모리(virtual memory)
    - **메인 메모리의 추상화**를 통해 컴퓨터가 **사용 가능한 메모리를 확장**하는 것
    - RAM 상에 사용중이지 않은 콘텐츠를 **디스크에 저장**하는 방식
    - 추후에 해당 콘텐츠를 사용하게 될 때 **다시 RAM으로** 불러온다.
- 스왑 공간 (또는 스왑 메모리)
    
    ![swap_memory](/assets/img/project/ec2-virtual-memory/Untitled.png)
    
    - **가상 메모리**로 사용되는 하드 디스크 영역
    - 모든 스왑 메모리는 가상 메모리지만, 모든 가상 메모리가 스왑 메모리인 것은 아니다.
        - 스왑 메모리는 하드 디스크의 공간만을 의미
        - 가상 메모리는 메모리 매핑이나 요구 페이징 등의 **기술을 포함**하는 광범위한 개념

# 설정 방법

1. 기존에 인스턴스에 swap 메모리가 설정되어 있는지를 확인한다.
    
    ```bash
    sudo swapon -s
    ```
    
    아무것도 **출력되지 않는다**면 아직 설정되지 않은 것이다.
    
2. 원하는 크기의 **swap 파일**을 생성한다.
    
    ```bash
    sudo fallocate -l [SIZE]G /swapfile
    ```
    
    예시. 2GB로 설정하고 싶다면 다음과 같이 입력한다.
    
    ```bash
    sudo fallocate -l 2G /swapfile
    ```
    
3. **파일 권한**을 600(`-rw-------`)으로 설정한다.
    
    ```bash
    sudo chmod 600 /swapfile
    ```
    
4. 해당 파일을 **swap 공간**으로 설정한다.
    
    ```bash
    sudo mkswap /swapfile
    ```
    
5. swap 공간을 **enable**한다.
    
    ```bash
    sudo swapon /swapfile
    ```
    
6. 설정된 **swap 공간을 확인**한다.
    
    ```bash
    sudo swapon -s
    ```
    
    정상적으로 설정되었다면 다음과 같은 메시지가 출력될 것이다.
    
    ```bash
    Filename             Type         Size            Used            Priority
    /swapfile            file         2097148         0               -2
    ```
    
7. swap 파일이 시스템 재부팅 후에도 **계속 적용**되도록 설정한다.
    
    ```bash
    echo '/swapfile swap swap defaults 0 0' | sudo tee -a /etc/fstab
    ```
    
    부팅 시에 `/etc/fstab` 파일에 새로운 엔트리를 추가한다.
    
    - `/etc/fstab` : Linux 운영체제에서 부팅 시에 자동으로 파일 시스템을 마운트하는데 사용된다.

# 재설정 방법

> 기존에 설정된 swap 공간의 용량을 변경하는 방법

1. 현재 swap **공간을 해제**한다.
    
    ```bash
    sudo swapoff /swapfile
    ```

    - `free -h` 로 확인할 수 있다.

        ```bash
                       total        used        free      shared  buff/cache   available
        Mem:           965Mi       333Mi       166Mi       0.0Ki       465Mi       466Mi
        Swap:             0B          0B          0B
        ```
    
2. 현재 swap 파일을 **삭제**한다.
    
    ```bash
    sudo rm /swapfile
    ```
    
    이후의 과정은 위와 동일하다.

3. 원하는 용량으로 swap 파일을 생성한다.
    
    ```bash
    sudo fallocate -l [SIZE]G /swapfile
    ```
    
4. 파일 권한을 600(`-rw-------`)으로 설정한다.
    
    ```bash
    sudo chmod 600 /swapfile
    ```
    
5. 해당 파일을 swap 공간으로 설정한다.
    
    ```bash
    sudo mkswap /swapfile
    ```
    
6. swap 공간을 enable한다.
    
    ```bash
    sudo swapon /swapfile
    ```
    
7. 새롭게 설정한 swap 공간을 확인한다.
    
    ```bash
    sudo swapon -s
    ```
    
8. swap 파일이 시스템 재부팅 후에도 계속 적용되도록 설정한다.
    
    ```bash
    echo '/swapfile swap swap defaults 0 0' | sudo tee -a /etc/fstab
    ```
    

# 참고
- contents
    - [The Difference Between Virtual Memory and Swap Space \| Baeldung on Computer Science](https://www.baeldung.com/cs/virtual-memory-vs-swap-space)
    - [operating system - What's the difference between "virtual memory" and "swap space"? - Stack Overflow](https://stackoverflow.com/questions/4970421/whats-the-difference-between-virtual-memory-and-swap-space)
    - [Use swap file to allocate memory as swap space in Amazon EC2 instance \| AWS re:Post](https://repost.aws/knowledge-center/ec2-memory-swap-file)
    - [amazon web services - How do you add swap to an EC2 instance? - Stack Overflow](https://stackoverflow.com/questions/17173972/how-do-you-add-swap-to-an-ec2-instance)
    - [How To Add Swap Space on Ubuntu 20.04  \| DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-add-swap-space-on-ubuntu-20-04)
- images
    - [Swap file in Windows 10: optimal size, how to change, move, disable or delete it?](https://recoverhdd.com/blog/swap-file-in-windows.html)