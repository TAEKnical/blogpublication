# AWS 네트워크의 이해(AWS Builders 200)

https://kr-resources.awscloud.com/aws-builders-ondemand-level-200/amazon-vpc%EC%99%80-elb-direct-connect-vpn-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0-%EA%B9%80%EC%84%B8%EC%A4%80-aws-%EC%86%94%EB%A3%A8%EC%85%98%EC%A6%88-%EC%95%84%ED%82%A4%ED%85%8D%ED%8A%B8-aws-builders-200-3

---

## VPC

### 개요

Virtual Private Cloud

사용자가 정의한 가상의 AWS 클라우드의 가상 데이터센터이자 네트워크 환경이며 논리적으로 격리됨

사용자별 네트워크 제어 가능

VPN, DirectConnect를 이용하면 온프레미스와 연결 가능

VPC 설정 단계

- IP 범위 지정
- 용도에 따른 Subnet 분리
- Routing Table
- Security Group, NACL
- Gateway : Internet Gateway, NAT Gateway, Virtual Gateway 



### 1. 초기 설정

1. Region, IP 대역 설정

   **Region 선택**

   서울은 3개 가용영역(서울 2개 , 용인1개로 알려짐) 운영중이며 2020년 7월에 AWS에서 4번째 AZ에 대해서 논의

   

   **서브넷 CIDR 설정**

   IP 범위를 설정. RFC 1918 private 대역을 준수하기를 권장.

   예약된 IP 주소(10.0.0.0/24인 경우)

   |대역|사용|

   |---|---|

   |~0 | 네트워크 주소|

   |~1 | VPC 라우터용으로 예약된 주소|

   |~2 | AWS DNS 주소|

   |~3 | AWS에서 향후 사용을 위해 예약|

   |~255 | 브로드캐스트 주소. VPC에서는브로드캐스트를 지원하지 않으나, AWS에서 예약|

2. AZ에 Subnet 설정

3. Routing 설정

   **Routing**

   동일 네트워크 안에서는 통신 가능하나, 다른 네트워크와의 통신에는 라우팅 설정 필요

   VPC 생성시 각 서브넷간 통신이 가능하도록 디폴트 라우팅 테이블이 생성되고, 삭제 불가능

   커스텀 라우팅 테이블을 정의할 수 있는데, 외부 인터넷이나 온프레미스와 통신을 위해 라우팅 룰을 설정 가능.

4. Traffic 통제(In/Out)

   **Network ACL**

   서브넷 단위로 적용 가능한 Stateless 방화벽

   Inboud & Outbound

   Rule # 낮은 순서부터 적용

   초기에는 기본적으로 전부 허용 상태

   

   **Security Group**

   인스턴스 단위

   Stateful 방화벽

   생성시 기본적으로 전부 차단되어있는 상태

   

   ### 2. VPC 확장

   1. to Internet

      **Internet Gateway**

      VPC 내부 인스턴스와 외부 인터넷을 연결하는 관리형 서비스. VPC당 1개.

      IPv4, IPv6 지원

      인터넷을 사용하려면 EIP가 필요

      **NAT Gateway**

      인터넷이 불가능한 Private Subnet에 있는 인스턴스들이 업데이트 등의 일시적인 인터넷 접속이 필요할 때 사용되는 관리형 서비스

      EIP가 필요함

      TCP, UDP, ICMP 프로토콜 지원

      NACL을 통한 트래픽 통제로, 특정 서버에서만 다운로드 작업 가능

      VPC Flow Logs를 통해서 로그 수집 가능하며 CloudTrail로 세부적인 로그 수집.

      **EIP**

      계정마다 할당되어 변경되지 않음

      리전당 5개 할당 가능

      EIP가 인스턴스와 매핑된 상태면 과금이 되지 않지만, 매핑되지 않은 상태면 과금

   2. to On-Premise

      **VPN Gateway**

      IPSec 프로토콜 기반의 Site-to-Site VPN을 통하여 고객사 네트워크와 암호화 터널 구축

      이 터널은 이중화 되어있고, TLS로 통신하기 떄문에 안전함

      **Direct Connect(DX)**

      AWS와 연결하는 방식이 아니라 전용 회선으로 고객사 네트워크와 직접 연결

      AWS는 이미 DX와 연결되어 있기 때문에 고객사에서 DX 위치까지만 전용 회선을 구축하면 됨

      **VPC Peering**

      완전히 격리된 VPC 네트워크간의 연결하는 옵션

      다른 계정 뿐만 아니라 다른 리전의 VPC와도 연결 가능

      라우팅 테이블로 통제 가능하며, Transit Routing은 제공하지 않음

      **VPC Endpoint**

      인터넷을 경유하지 않고 VPC 내의 자원과 AWS 서비스와의 연결을 위해  VPC Endpoints를 사용

      VPC Endpoint 생성시 Routing Policy 추가

      다양한 접근 제어 정책 사용(Routing Table, Endpoint Policy, S# Bucket Policy, Security group)

   3. VPC 관리

      **VPC Flow Logs**

      VPC로 들어오는 트래픽을 모니터링

      VPC, Subnet, EC2 인스턴스 단위로 적용 가능한 패킷 수집

      생성된 패킷에 대한 정보를 10분~15분 내에 CloudWatch Logs Group에 적용

      DNS 트래픽, 라이센스 인증 트래픽, DHCP 트래픽 등은 수집하지 않음

   4. 

      

      

      

# Direct Connect

## 1. DirectConnect 유형

3가지 VIF(Virtual Interface) 제공

- Public VIF

  온프레미스에서 퍼블릭 서비스(S3, DynamoDB) 등을 Directconnect로 연결

  BGP(Border Gateway Protocol)를 ASN(Autonomous System Number) 및 IP 접두사와 함께 사용

  BGP 구성시 고객의 Public ASN 필요

- Private VIF

  온프레미스와 VPC에 있는 리소스를 연결하기 위해 사용

  프라이빗 ASN을 사용중이면 ASN이 64512-65535 범위에 속해야 함

- Transit VIF

  Transit Gateway와 Directconnect 할 때 사용

  VPN, Direct Connect의 단일 접점 역할을 하므로 복잡한 네트워크 토폴로지를 개선

  ECMP(Equal-cost multi-path routing) 지원

  CloudWatch, VPC Flow Logs와 연계하여 통합 트래픽 모니터링 가능

## 2. DirectConnect 특징

DX Connection당 최대 50개의 가상 인터페이스(Private and Public) 생성 및 연결 가능

대역폭 원하는 고객사를 위해 APN 파트너를 통한 Hosted Connection 사용 가능. 단, 1개의 VIF만 연결 가능

VPN 대비 낮은 latency와 트래픽 요금



# ELB

트래픽을 분배해주는 관리형 로드밸런서 서비스이며, 리전 내에서 로드밸런싱 함

ELB의 IP는 기본적으로 변경될 수 있기 때문에 ELB 접속시 IP가 아닌 DNS Name Record(DNS의 CNAME, Route53의 Alias ) 이용. 단, NLB는 고정 IP 부여 가능.

TLS Termination : 클라이언트의 SSL을 ELB에서 Termination 하게 되면 백엔드의 EC2 인스턴스에는 SSL 부하가 감소한다.

가용성을 높이기 위해 Cross-Zone 로드 밸런싱을 하는 것이 좋음.

Sticky Session이 필요한 경우는 ALB 사용

## 1. ALB

**Listner**

Listening Port, Protocol 지정

ALB당 최소 1개 이상 지정, 최대 50개 생성 가능

Contents 기반 Rule 지정(Host 기반, Path 기반)

마이크로서비스, 컨테이너 기반 애플리케이션, RESTFUL한 처리에 적합

**Target Group**

ALB 밑 단 타겟들의 논리적 그룹

Target Group에 포함 가능한 리소스 : EC2, ECS, EKS, IP Address, Lambda

![image-20201223171501728](C:\Users\MZC01-JTLEE\AppData\Roaming\Typora\typora-user-images\image-20201223171501728.png)

## 2. NLB

고성능 - 초당 수백만 요청 처리, 낮은 Latency

NLB에는 AZ당 하나의 고정 IP 부여 가능

Long-running Connection이 필요한 애플리케이션에 적합(IoT, gaming, messaging)

ALB와 마찬가지로 Listener, Targer Group으로 구성

Rule을 추가하여 포트 기반 트래픽 처리할 타겟 그룹을 구성

TLS Termination 지원, ACM 연동 가능

