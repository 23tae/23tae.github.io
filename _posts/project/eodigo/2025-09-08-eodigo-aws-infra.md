---
title: "어디GO AWS 인프라 구축 과정 (feat. Private Subnet)"
date: 2025-09-08T09:00:00.000Z
categories: [Project, 어디GO]
tags: [aws]
---

## 들어가며

'어디GO' 프로젝트에서 나는 백엔드 API 개발과 함께 서비스가 동작할 AWS 인프라를 구축하는 역할을 맡았다. 단순히 EC2 인스턴스 하나를 띄우는 것을 넘어, **보안, 고가용성, 비용 효율성**이라는 세 가지 목표를 가지고 인프라를 설계했다.

이 글에서는 AWS의 초기 설계가 현실적인 제약에 부딪혀 새로운 구조로 변경되기까지의 과정을 자세히 다룬다.

## 초기 인프라 설계 (v1): 보안을 최우선으로

프로젝트 초기, 나는 AWS Well-Architected 등에서 권장하는 Best Practice를 최대한 따르는 것을 목표로 삼았다.

### 1. VPC와 서브넷

![create_vpc.jpg](/assets/img/project/eodigo/aws-infra/create_vpc.jpg)

가장 먼저 VPC 생성 마법사를 이용해 격리된 네트워크를 구축했다.

- **가용 영역(AZ) 2개 구성:** 고가용성을 위해 서울 리전의 두 개의 다른 데이터 센터(AZ)에 걸쳐 네트워크를 구성했다. 하나의 AZ에 문제가 생겨도 다른 AZ에서 서비스를 운영할 수 있는 기반을 마련한 것이다.
- **Public/Private 서브넷 분리:** 각 AZ마다 외부와 통신하는 Public Subnet과 내부망 역할을 하는 Private Subnet을 하나씩 생성했다. 보안이 중요한 EC2와 RDS는 모두 Private Subnet에 배치할 계획이었다.
- **NAT 게이트웨이와 비용 최적화:** Private Subnet의 EC2가 OS 업데이트 등을 위해 외부 인터넷과 통신할 수 있도록 NAT 게이트웨이를 구성했다. 고가용성을 위해서는 AZ마다 하나씩 두는 것이 정석이지만, 개발 단계의 비용을 절감하기 위해 1개의 AZ에만 생성했다.

### 2. IAM과 보안 그룹

네트워크 설계 후, 리소스에 접근하는 주체인 '사람'과 '기계'의 권한을 최소한으로 제한했다.

- **사용자 그룹 분리:** 인프라를 직접 생성하고 변경하는 `Administrators` 그룹과 애플리케이션 배포 및 로그 확인 등 제한된 권한만 필요한 `Developers` 그룹을 나누었다. 이를 통해 사람의 실수로 인한 리스크를 줄이고자 했다.
    
    ![image.png](/assets/img/project/eodigo/aws-infra/image.png)
    
- **EC2를 위한 IAM 역할:** EC2 인스턴스에 Access Key를 하드코딩하는 대신, IAM 역할을 부여했다. 이 역할 덕분에 EC2는 키 정보 없이도 ECR에서 Docker 이미지를 가져오는 등 필요한 권한을 안전하게 위임받을 수 있었다.
    
    ![aws_ec2_role.jpg](/assets/img/project/eodigo/aws-infra/aws_ec2_role.jpg)
    
- **보안 그룹(Security Group) 설정:** 리소스 단위의 방화벽인 보안 그룹은 `dnd-sg-ec2`(EC2용)와 `dnd-sg-rds`(RDS용) 두 개를 만들어 서로 다른 규칙을 적용했다. 특히 RDS 보안 그룹의 인바운드 규칙에는 IP 주소가 아닌, **EC2의 보안 그룹 ID를 소스(Source)로 지정**했다. 이는 오직 내 EC2 인스턴스만이 데이터베이스에 접근할 수 있도록 강제하는 설정이다.
    
    ![rds_security_group.png](/assets/img/project/eodigo/aws-infra/rds_security_group.png)
    

### 3. Private Subnet의 EC2와 RDS

모든 준비를 마친 뒤, `t4g.small` 사양의 EC2 인스턴스와 RDS for MySQL 인스턴스를 모두 **Private Subnet**에 생성했다. 이로써 외부 인터넷에서는 우리의 애플리케이션 서버와 데이터베이스에 직접 접근할 수 없는, 이론적으로는 안전한 구조가 완성되었다.

![aws_architecture_v1.png](/assets/img/project/eodigo/aws-infra/aws_architecture_v1.png)
_아키텍처 (v1)_

## 문제점: 고립된 EC2

하지만 이러한 안전한 구조는 MVP를 빠르게 개발하고 테스트해야 하는 우리 팀에게 문제로 작용했다.

- **주요 문제점:** Private Subnet에 위치한 EC2는 Public IP가 없어, **외부에서 애플리케이션 접근이 어려웠다.**
- **현실적인 영향:**
    - Swagger API 문서를 프론트엔드 개발자들에게 공유하기 어려웠다. 개발자가 Swagger 문서를 보기 위해 SSH 터널링을 거쳐야 하는건 굉장히 번거로운 일이다.
    - Postman 등으로 간단한 API 테스트를 하는 것조차 불가능했다. 모든 확인 작업은 SSM으로 EC2에 접속해 `curl` 명령어를 사용하는 등 매우 비효율적인 방식으로 이루어져야 했다.

이 문제를 제대로 해결하려면 ALB(Application Load Balancer)를 Public Subnet에 두거나, 별도의 점프 서버(Bastion Host)를 구축해야 했다. 하지만 이는 MVP 단계의 프로젝트에서 비용과 관리 복잡성을 크게 증가시키는 해결책이었다.

## 인프라 재구축 (v2): 접근성과 보안의 균형점

고민 끝에 **EC2 인스턴스를 Public Subnet으로 이전**하는 방법을  선택했다. (RDS는 Private Subnet에 유지)

1. **AMI를 이용한 서버 복제:** 가장 먼저 기존 EC2의 환경을 그대로 보존하기 위해 **AMI(Amazon Machine Image)로 스냅샷을 생성**했다. 그리고 이 AMI를 사용해 Public Subnet에 동일한 환경의 새 EC2 인스턴스를 복제했다.
    
    ![create_ami.png](/assets/img/project/eodigo/aws-infra/create_ami.png)
    _AMI 생성_

    ![create new instance](/assets/img/project/eodigo/aws-infra/create_new_ec2_instance.png)
    _AMI를 사용해서 새 인스턴스 생성_
    
2. **네트워크 구성 변경 및 비용 절감:** EC2가 Public Subnet으로 이동하면서 더 이상 **NAT 게이트웨이가 필요 없어졌다.** 시간당 비용이 부과되는 NAT 게이트웨이를 즉시 삭제하여 비용을 절감했다.
    
    ![delete_nat.png](/assets/img/project/eodigo/aws-infra/delete_nat.png)
    
3. **고정 IP 할당 및 보안 강화:** 새로 생성된 Public EC2에는 재부팅 시에도 주소가 바뀌지 않도록 **탄력적 IP(Elastic IP)**를 할당했다. 그리고 보안 그룹 규칙을 수정하여 SSH(22번 포트)의 인바운드 소스를 `0.0.0.0/0`(Anywhere)가 아닌, **`내 IP`**로 지정하여 허가된 장소에서만 인스턴스에 접속할 수 있도록 제한했다.
    
    ![ec2_inbound_rules.jpg](/assets/img/project/eodigo/aws-infra/ec2_inbound_rules.jpg)
    

## 로컬 DB 접속 환경 구축 (SSH 터널링)

EC2의 접근성 문제를 해결한 후, 로컬 PC에서 데이터베이스 내부 데이터를 확인할 일이 생겼다. 하지만 RDS는 여전히 Private Subnet에 있어 직접 접근이 불가능했다. 그 대신 이제 Public Subnet에 외부와 통신할 수 있는 EC2가 생겼다. 이 **Public EC2를 점프 서버로 활용하는 SSH 터널링**을 설정하여, 로컬 PC에서 Private RDS로 안전하게 접속하는 통로를 만들었다.

- **DataGrip 설정**

  나는 주로 사용하는 DB 클라이언트인 **DataGrip**의 내장 SSH/SSL 터널링 기능을 활용했다. Proxy host에 Public EC2의 탄력적 IP를, Key-pair에 `.pem` 키 경로를 지정해주면 간단하게 터널을 생성할 수 있다.

  ![datagrip_config](/assets/img/project/eodigo/aws-infra/datagrip_config_1.png)
  
  ![datagrip_config](/assets/img/project/eodigo/aws-infra/datagrip_config_2.png)
    

## 마치며

최종 아키텍처(v2)는 다음과 같다.

![aws_architecture_v2.png](/assets/img/project/eodigo/aws-infra/aws_architecture_v2.png)
_아키텍처 (v2)_

이번 인프라 구축을 통해 설계는 한 번에 완벽하게 하는 것이 아니라, **프로젝트의 진행 상황과 새로운 요구사항에 맞춰 점진적으로 개선해 나가는 것**임을 배웠다. 애플리케이션의 접근성은 높이면서도 데이터베이스의 보안은 유지하는 현실적인 트레이드오프를 통해, 이론과 현실 사이의 균형점을 찾는 경험을 할 수 있었다.

## 참고 자료

- [AWS Well-Architected 프레임워크 - AWS Well-Architected 프레임워크](https://docs.aws.amazon.com/ko_kr/wellarchitected/latest/framework/welcome.html)
- [Configure SSH and SSL \| DataGrip Documentation](https://www.jetbrains.com/help/datagrip/configuring-ssh-and-ssl.html)
- [[AWS] Private EC2 인터넷 연결하기](https://bosungtea9416.tistory.com/entry/AWS-Private-EC2-%EC%9D%B8%ED%84%B0%EB%84%B7-%EC%97%B0%EA%B2%B0%ED%95%98%EA%B8%B0)
- [[AWS Subnet] 서버 Private Subnet으로 옮기기(1)](https://developerbee.tistory.com/216)
