# Openswan을 이용한 AWS site to site VPN 구현

# 아키텍처

---

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/vpn_arc.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/vpn_arc.png)

온프레미스 환경이 필요하기 때문에 고객사를 가정한 VPC, AWS 측의 VPC 총 두 개를 구현하였다. 완전히 분리된 환경을 구현하기 위해 서로 다른 리전에 각각 VPC를 구성하였으며 위의 아키텍처를 성공적으로 구현하면 서로 다른 네트워크에 속해있는 두 EC2 인스턴스는 VPN connection을 통해 서로의 Private IP로 통신이 가능해진다. 이 글에서는 AWS_VPC를 Seoul Region에, On-premise 는 Virginia Region에 구성하였다.

# 작업 순서

**고객사 측(On-premise) 환경 구현**

1. VPC 생성
2. 서브넷 생성
3. 라우팅 테이블 생성
4. 인터넷 게이트웨이 생성
5. 인스턴스 생성
6. SSH 접속
7. openswan 설치 및 설정

**AWS 측 환경 구현**

1. VPC 생성
2. 서브넷 생성
3. 라우팅 테이블 생성
4. 인터넷 게이트웨이 생성
5. 인스턴스 생성

**VPN Connection**

1. 고객 게이트웨이 생성(Customer Gateway)
2. VGW(Virtual private Gateway) 생성
3. Site-to-Site VPN Connection
4. Route Propagation
5. Download Configuration
6. Openswan 설정

---

# 고객사 측(On-premise)

---

## 1. VPC 생성

---

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled.png)

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%201.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%201.png)

**Create VPC**

Name tag : onpremise_vpc

IPv4 CIDR block : 10.0.0.0/16

Tenancy : Dedicated를 선택하면 AWS 데이터 센터에 전용 하드웨어 영역을 할당할 수 있다. 지금은 필요하지 않으므로 Default로 설정.

## 2. 서브넷

---

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%202.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%202.png)

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%203.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%203.png)

**Public Subnet**

Subnet name : onpremise_public_sub

Availability Zone : US East (N. Virginia) / us-east-1a

IPv4 CIDR block : 10.0.0.0/24

**Private Subnet**

Subnet name : onpremise_private_sub

Availability Zone : US East (N. Virginia) / us-east-1a

IPv4 CIDR block : 10.0.1.0/24

## 3. 라우팅 테이블

---

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%204.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%204.png)

public route table을 생성하고 onpremise_vpc에 연결한다.

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%205.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%205.png)

public route table과 public subnet을 연결한다.

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%206.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%206.png)

private route table에 대해서도 동일하게 적용한다.

## 4. 인터넷 게이트웨이

---

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%207.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%207.png)

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%208.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%208.png)

인터넷 게이트웨이(onpremise_igw)를 생성하고 onpremise_vpc에 연결한다.

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%209.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%209.png)

onpremise_public_rt-Edit routes를 선택하고 퍼블릿 서브넷의 외부로 나가는 모든 트래픽이 인터넷 게이트웨이로 향하도록 설정한다.

## 5. 인스턴스

---

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2010.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2010.png)

AMI는 Amazon Linux 2를 선택

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2011.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2011.png)

인스턴스 타입은 t2.micro를 선택

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2012.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2012.png)

인스턴스를 배치할 VPC와 서브넷을 선택하고 Public IP를 자동 할당할 것인지 선택한다.

나머지 옵션은 디폴트 값으로.

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2013.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2013.png)

스토리지는 기본 값으로 선택

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2014.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2014.png)

Key : Name

Value : onpremise_ec2

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2015.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2015.png)

보안 그룹 생성

All TCP : 모든 TCP 트래픽을 허용한다.

All UDP : openswan은 UDP 4500 포트를 이용한다. 편의상 모든 UDP 트래픽을 허용한다.

SSH : 테스트를 위해 My IP에서만 SSH 접근이 가능하도록 한다.

All ICMP : AWS_VPC와 ping을 주고받기 위해 허용한다.

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2016.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2016.png)

키 페어를 생성하고 저장한다.

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2017.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2017.png)

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2018.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2018.png)

onpremise_ec2 → Networking → Change source/destination check → Stop

패킷에 명시된 source 또는 destination에 해당하지 않더라도 패킷을 받아볼 수 있도록 check 작업을 정지시킨다. 예를 들어 NAT 인스턴스는 하위 사설망에 패킷을 전달하기 위해 들어오는 외부 패킷을 열어볼 수 있어야 한다.

## 6. SSH 접속 및 openswan 설치

---

```bash
root@MZC01-JTLEE:~# ssh -i onpremise_key.pem ec2-user@35.174.104.59

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/

[ec2-user@ip-10-0-0-195 ~]$ sudo su -
[root@ip-10-0-0-195 ~]# yum install -y openswan
```

ssh로 onpremise_ec2에 접속해서 openswan을 설치한다.

**/etc/ipsec.conf 수정**

```bash
[root@ip-10-0-0-195 ~]# vim /etc/ipsec.conf
...
# It is best to add your IPsec connections as separate files in /etc/ipsec.d/
include /etc/ipsec.d/*.conf
```

/etc/ipsec.conf 파일을 열어서 **include /etc/ipsec.d/*.conf** 라인의 주석을 해제한다. 이미 해제되어 있으면 그대로.

**/etc/sysctl.conf 수정**

```bash
net.ipv4.ip_forward = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
```

/etc/sysctl.conf 파일을 열어서 위의 내용을 추가한다.

**network 서비스 재시작**

```bash
[root@ip-10-0-0-195 ~]# systemctl restart network
```

# AWS 측

---

## 1. VPC 생성

---

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2019.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2019.png)

vpc 이름을 aws_vpc로, 대역을 172.16.0.0/16으로 설정한다.

## 2. 서브넷 생성

---

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2020.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2020.png)

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2021.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2021.png)

onpremise와 마찬가지로 public / private subnet을 각각 설정한다.

public subnet : 172.16.0.0/24

private subnet : 172.16.1.0/24

## 3. 라우팅 테이블

---

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2022.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2022.png)

aws_private_rt

aws_public_rt

두 서브넷을 onpremise와 동일하게 생성하고 퍼블릭 서브넷은 IGW로 모든 트래픽을 보내도록 설정한다.

## 4. 인터넷 게이트웨이

---

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2023.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2023.png)

IGW 생성 후 VPC에 attach한다.

## 5. 인스턴스

---

온프레미스 ec2와 동일하게 생성한다.

# VPN Connection

---

## 1. 고객 게이트웨이(Customer Gateway)

---

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2024.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2024.png)

이 부분은 AWS측이 세팅된 Seoul 리전에서 작업한다.

Name : onpremise-cgw

Routing : Static

IP Address : onpremise_ec2 의 public ip

Certificate ARN : <static 일때는 기입할 필요 없음>

Device : <static 일때는 기입할 필요 없음>

## 2. Virtual Private Gateway(VGW)

---

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2025.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2025.png)

Name tag : aws-onpremise-vgw

ASN : Amazon default ASN

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2026.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2026.png)

생성한 VGW를 VPC에 attach하기 위해 aws-onpremise-vgw→Attach to VPC에 진입

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2027.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2027.png)

AWS_VPC를 선택한다.

## 3. VPN Connection

---

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2028.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2028.png)

Name tag : aws_onpremise_vpn

Target Gateway Type : Virtual Private Gateway

Virtual Private Gateway : aws-onpremise-vgw

Customer Gateway : Existing(New를 선택하면 새로 생성)

Customer Gateway ID : onpremise-cgw

Routing Options : Static

Static IP Prefixes : 10.0.0.0/16

Tunnel Inside IP Version : IPv4

Local IPv4 Network CIDR : 고객 게이트웨이와 VPN상에서 통신할 대역을 제한한다.

Remote IPv4 Network CIDR : AWS측과 VPN상에서 통신할 대역을 제한한다.

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2029.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2029.png)

그 아래의 Tunnel Options 항목들은 기입하지 않아도 AWS에서 자동으로 생성해준다. 만약 사전에 준비된 대칭키가 있다면 사용할 수 있다.

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2030.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2030.png)

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2031.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2031.png)

VPN Connection을 생성하고 State가 available이 될 때 까지 잠시 기다린 후 하단의 Tunnel Details 항목에서 2개의 VPN Tunnel을 확인할 수 있다. 아직은 두 터널이 모두 DOWN 상태이다.

## 4. Route Propagation

---

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2032.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2032.png)

AWS측의 퍼블릭 라우팅 테이블을 선택하고 하단의 Route Propagation→Edit route propagation 이동

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2033.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2033.png)

앞서 생성한 aws-onpremise-vgw의 Propagate에 체크 후 Save

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2034.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2034.png)

라우팅 테이블이 확장된 것을 확인할 수 있다. on-premise에 해당하는 10.0.0.0/16대역이 목적지인 트래픽은 모두 VGW로 향하게 되었다.

## 5. Download Configuration

---

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2035.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2035.png)

다시 Site-to-Site VPN Connection 메뉴로 돌아와서 aws_onpremise_vpn을 선택하고 상단의 Download Configuration 클릭

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2036.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2036.png)

Openswan에 맞는 형식으로 Configuration 파일이 다운로드 되도록 설정하고 다운로드한다.

## 6. Openswan 설정

---

onpremise-ec2에 SSH로 연결한 상태에서 먼저 root 권한으로 전환한다.

```bash
[ec2-user@ip-10-0-0-195 ~]$ sudo su -
Last login: Wed Jan  6 05:10:18 UTC 2021 on pts/0
```

이 단계에서부터 필요한 내용은 모두 앞서 다운로드한 Configuration 파일에서 그대로 복사한다.

**/etc/ipsec.d/aws-vpn.conf 설정**

```bash
conn Tunnel1
 authby=secret
 auto=start
 left=%defaultroute
 leftid=<Customer end VPN public IP>
 right=<AWS VPN Tunnel 1 public IP>
 type=tunnel
 ikelifetime=8h
 keylife=1h
 phase2alg=aes128-sha1;modp1024
 ike=aes128-sha1;modp1024
 keyingtries=%forever
 keyexchange=ike
 leftsubnet=<Customer end VPN CIDR>
 rightsubnet=<AWS end VPN CIDR>
 dpddelay=10
 dpdtimeout=30
 dpdaction=restart_by_peer
```

/etc/ipsec.d/aws-vpn.conf 파일을 생성하고 위의 형식에 맞게 내용을 수정한 뒤 저장.

**/etc/ipsec.d/aws-vpn.secrets** 설정

```bash
35.174.104.59 15.165.69.122: PSK "g2d6UT4sPPAjxnJLlBABdqUt__Sd3_4w"
```

파일을 생성하고 다운로드한 Configuration 파일에서 Pre-Shared Key 값을 붙여넣는다.

**openswan(ipsec)** **서비스 재시작**

```bash
[root@ip-10-0-0-195 ~]$ chkconfig ipsec on
[root@ip-10-0-0-195 ~]$ service ipsec start
```

**openswan(ipsec) 서비스 상태 확인**

```bash
[root@ip-10-0-0-195 ~]$ ipsec status
...
000 Total IPsec connections: loaded 1, active 1
```

커넥션이 active 되었음을 확인.

```bash
[root@ip-10-0-0-195 ~]# netstat -nptulz
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      2491/rpcbind
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      3209/sshd
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      2967/master
tcp6       0      0 :::111                  :::*                    LISTEN      2491/rpcbind
tcp6       0      0 :::22                   :::*                    LISTEN      3209/sshd
udp        0      0 0.0.0.0:68              0.0.0.0:*                           3827/dhclient
udp        0      0 0.0.0.0:111             0.0.0.0:*                           2491/rpcbind
udp        0      0 127.0.0.1:323           0.0.0.0:*                           2507/chronyd
udp        0      0 127.0.0.1:4500          0.0.0.0:*                           4290/pluto
udp        0      0 10.0.0.195:4500         0.0.0.0:*                           4290/pluto
udp        0      0 0.0.0.0:963             0.0.0.0:*                           2491/rpcbind
udp        0      0 127.0.0.1:500           0.0.0.0:*                           4290/pluto
udp        0      0 10.0.0.195:500          0.0.0.0:*                           4290/pluto
udp6       0      0 fe80::10c3:c2ff:fe4:546 :::*                                3928/dhclient
udp6       0      0 :::111                  :::*                                2491/rpcbind
udp6       0      0 ::1:323                 :::*                                2507/chronyd
udp6       0      0 :::963                  :::*                                2491/rpcbind
udp6       0      0 ::1:500                 :::*                                4290/pluto
```

4500번 포트에서 pluto 가 실행중임을 확인. pluto는 ipsec 관련 프로세스.

# 결과 확인

---

![Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2037.png](Openswan%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20AWS%20site%20to%20site%20VPN%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20d9790e6513cb4b43824c1749866da528/Untitled%2037.png)

site-to site vpn connection 탭에서 Tunnel Detail을 확인해보면 첫 번째 터널이 UP 상태임을 확인 가능하다. 터널이 두 개인 이유는 기본적으로 AWS에서 이중화를 권장하기 때문이며 실제 고객사 VPN을 세팅할 때는 이중화된 터널을 세팅해야 하나의 터널이 사용 불가능한 상태가 되었을 때 다른 터널로 대체할 수 있다.

```bash
[root@ip-10-0-0-195 ~]# ping 172.16.0.235
PING 172.16.0.235 (172.16.0.235) 56(84) bytes of data.
64 bytes from 172.16.0.235: icmp_seq=1 ttl=254 time=195 ms
64 bytes from 172.16.0.235: icmp_seq=2 ttl=254 time=195 ms
64 bytes from 172.16.0.235: icmp_seq=3 ttl=254 time=195 ms

--- 172.16.0.235 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 195.272/195.298/195.352/0.511 ms
```

Virginia 리전에 있는 onpremise EC2 인스턴스에서 서울 리전의 aws EC2의 private IP를 이용해서 성공적으로 ICMP 트래픽이 오고 감을 확인할 수 있다.