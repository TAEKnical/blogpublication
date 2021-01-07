# AWS 핵심 네트워크 서비스 정리

## VPC
사용자 계정에 전용으로 할당되는 독립된 클라우드이자 가상 네트워크 공간. 서브네팅, 라우팅, 연결 방식, 보안 및 제어 등 네트워크망 구축과 관련된 대부분의 서비스를 제공한다.

---
---
### 1. Region, AZ, Edge Location
---
- Region은 AZ(Available Zone)의 집합이며, AWS의 서비스가 위치하고 있는 물리적인 장소. 주로 국가 또는 도시단위이다. 실질적인 IDC는 AZ라고 할 수 있다.
- Edge Location은 CDN을 위한 캐시 서버를 가리킨다. AWS에서는 Cloudfront라는 CDN서비스를 제공한다.


### 2. Subnet
---
AZ에 따라 네트워크 대역을 서브네팅하며, CIDR 표기법을 사용한다.
VPC 내의 서브넷은 Public Subnet과 Private Subnet으로 구성된다.
- Public Subnet : IGW, ELB, EIP를 가진 인스턴스를 배치해서 외부 인터넷과 통신이 가능하도록 한다.
- Private Subnet : 기본적으로 외부망과 차단된 내부망을 위한 서브넷이며 내부의 인스턴스들은 private IP만을 가진다. 원래는 다른 서브넷과의 연결만 가능하지만, NAT Gateway 서비스를 이용하면 일시적으로 인터넷에 연결하여 업데이트 등의 작업을 할 수 있다.

### 3. Route Table
---
앞서 생성된 Public Subnet와 Private Subnet에 각기 다른 라우팅 테이블을 적용하여 아웃바운드 트래픽의 경로를 설정할 수 있다. Public Subnet은 0.0.0.0/0 을 IGW로 향하도록 함으로써 해당 서브넷의 모든 인스턴스의 트래픽이 인터넷 게이트웨이를 통하므로 인터넷이 가능하도록 한다. 이와 달리 Public Subnet은 0.0.0.0/0을 NAT 인스턴스(bastion 호스트) 또는 NAT Gateway로 향하도록 한다.

### 4. Internet Gateway, NAT Gateway
---
- Internet Gateway(IGX) : VPC 내부 인스턴스와 외부 인터넷을 연결하는 관리형 게이트웨이 서비스이며, VPC당 1개만 사용할 수 있다. EIP가 필요함.
- NAT Gateway : 인터넷이 불가능한 Private Subent에 있는 인스턴스들이 업데이트 등의 이유로 일시적인 인터넷 접속이 필요할 때 사용되는 관리형 게이트웨이 서비스. 역시 EIP가 필요하며, NACL을 이용한 트래픽 제어로 특정 서버에서만 다운로드 작업이 가능하도록 할 수 있다.

### 5. Elastic IP(EIP)
---
AWS에서 부여하는 Static IP이며 계정마다 할당되어 변경되지 않는다. 리전당 5개까지 추가 할당이 가능하고, 첫 EIP는 무료로 제공한다. EIP가 인스턴스와 매핑된 상태면 과금되지 않지만 제대로 매핑되지 않거나 인스턴스가 중단된 상태면 과금된다. 네트워크 인터페이스 단위로 할당/회수가 가능하다.

### 6. Security Group vs NACL
---
| NACL | Security Group |
|---|---|
| 서브넷 레벨 | 인스턴스 레벨 |
| Stateless | Stateful |
| Allow & Deny | Allow만 가능 |
| 트래픽이 들어오면 규칙을 순서대로 평가 | 트래픽 허용 전 모든 룰을 평가 |
| 서브넷 당 하나의 NACL | 인스턴스는 여러개의 Security Group 가능 |
| 인바운드 보안의 첫 단계, 아웃바운드 보안의 두 번째 단계 | 인바운드 보안의 두 번째 단계, 아웃바운드 보안의 첫 단계 |


### 7. VPC Flow logs
---
VPC Flow logs 이전에는 EC2에 에이전트를 설치해서 로그 데이터를 수집했기에 각 인스턴스에 부하를 주고, 제한적인 보기만 가능했다.

VPC Flow logs는 네트워크 모니터링 중에서도 로그 수집의 기능 향상을 위한 서비스이다. 특정 VPC에서 VPC Flow logs를 사용하면 VPC 서브넷과 ENI의 네트워크 트래픽이 CloudWatch Logs틀 통해 저장된 후에 다른 분석 프로그램에서 분석할 수 있다.

로그 정보 안에는 NACL과 Security Group에 의해 허용 및 차단된 트래픽의 정보, soruce/destination IP, Port, 프로토콜 번호, 모니터링 시간 간격, 이에 따른 액션(ACCEPT/REJECT) 정보가 포함되어 있다.

### 8. AWS VPN
---
VPN은 가상 사설망으로, 마치 전용선 연결의 효과를 누릴 수 있도록 하는 기법이다. 보안성을 확보해야 하지만 전용선 구축이 어려울 때 자주 쓰인다. 암호화 통신과 비용 절감 효과를 볼 수 있지만 속도가 느리다는 단점이 있다.

AWS에서는 두 가지 유형의 VPN 서비스를 제공한다.
- AWS Site-to-Site VPN : 온프레미스 네트워크를 AWS VPC에 보안 연결
- AWS Client VPN : 사용자 PC만 AWS에 보안 연결

> **IPSec**
Layer3 네트워크계층에서 IP패킷 단위로 인증, 암호화, 키관리 하는 프로토콜.
경유 구간에 일종의 보안 통로인 터널링을 형성하고, VPN에서 많이 사용된다.
Site-to-Site, Client-to-Server, Client-to-Gateway 등 다양하게 사용 가능.


VPN을 연결하기 위해서는 AWS 콘솔에서 3가지 항목을 생성해야 한다.
- 고객 게이트웨이 : 고객 측 PC 또는 사무실 네트워크의 VPN 엔드포인트이며 BGP를 사용하기 위해서는 고객측 게이트웨이 퍼블릭 IP와 ASN이 필요하다.
- 가상 프라이빗 게이트웨이(Virtual Private Gateway) : AWS VPC측의 VPN 엔드포인트
- VPN 연결 : 고객 측 엔드포인트와 AWS VPC간의 가상 사설망 연결

### 9. VPC Endpoints
---
가상의 장치로서 VPC와 AWS 서비스간의 전용 연결을 만든다. 예를 들어 EC2 인스턴스에서 S3에 접근하려면 EIP가 할당된 인터넷 게이트웨이를 거쳐 접근하는 방법이 있겠지만 이렇게 되면 AWS 서비스끼리 통신하는데 외부 트래픽을 생성하므로 불필요한 트래픽 처리 비용이 발생한다.

VPC Endpoint를 생성하면 private subnet에 있는 인스턴스에서 리전 내에 있는 다른 서비스를 Route table에 등록하여 private ip만으로 서비스간 전용 통신이 가능해진다. 

### 10. VPC Peering, Transit Gateway
---
- VPC Peering
private IP를 사용하여 두 VPC간 트래픽을 라우팅할 수 있도록 하기 위한 VPC 사이의 연결이다. 동일 네트워크에 있는 것과 같은 효과로, 두 VPC간의 인스턴스가 통신 가능해진다. 사용자가 생성한 VPC간, 다른 계정의 VPC간, 다른 리전의 VPC간 Peering이 가능하다. 

- Transit Gateway
VPC와 온프레미스 네트워크를 하나의 게이트웨이에 연결할 있는 서비스. 서로 다른 리전의 VPC, 다른 계정의 VPC까지도 연결할 수 있다는 점에서 기능 자체는 VPC Peering과 Transit gateway 모두 동일하다. 단, VPC Peering은 연결을 중앙에서 관리하는 기능이 없어서 피어링이 늘어날 수록 관리와 운영 비용이 많이 들게 된다. 또한 VPC와 온프레미스와 연결하려면 VPN 연결도 필요하므로 VPC가 늘어날 때 마다 VPN을 각각 연결해주어야 하는 번거로움도 있다.

#### 10.1 VPC Peering vs Transit Gateway
|-| VPC Peering | Transit Gateway |
|---|---|---|
| 대역폭 | 무제한 | 50Gbps |
| 최대 개수 | VPC당 125개 | Transit Gateway당 5000개 |
| 연결 범위 | 다른 리전, 다른 계정의 VPC | 동일 리전, 다른 계정의 VPC |


상황에 따라 비용이 달라질 수 있으므로 예상 금액을 정산하여 비교 필요함.

### 11. Direct Connect Gateway
---
Direct Connect Gateway(이하 DxG)는 VPC간의 연결을 위해 사용한다. DxG와 transit gateway, DxG와 VGW를 연결할 수 있다.
- DxG - transit gateway : 동일 리전에 있는 다수의 VPC들을 연결할 때 사용
- DxG - VGW : 동일 리전 뿐 아니라 다른 리전에 있는 계정의 VPC들을 연결할 때 사용

>**애니캐스트**
들어오는 요청을 다양한 노드로 라우팅 할 수 있는 네트워크 주소를 지정하는 기술이다. 예를 들어 사용자는 특정 서버에 콘텐츠를 요청했지만 이를 실질적으로 처리하는 CDN에서는 트래픽을 처리할 수 있는 가장 가까운 데이터 센터로 라우팅시킨다. 모든 노드들은 동일한 IP주소를 공유하고 있다. 애니캐스트의 장점은 대기시간을 줄이고 가용성을 향상시킬 수 있다는 점이다.<br>
애니캐스트는 BGP 프로토콜(독립된 대규모 네트워크 간 연결을 위해 사용하는 라우팅 프로토콜)로 연결되고, 해당 라우터의 모든 이웃 라우터가 해당 라우터를 통해 도달할 수 있는 경로를 인지할 수 있게 한다. 그러므로 라우터는 자신의 이웃 라우터중 어떤 라우터가 최단 경로를 제공하는지 파악할  있게 된다.



# 참고 문서
**AWS — Difference between Security Groups and Network Access Control List (NACL)**
(https://medium.com/awesome-cloud/aws-difference-between-security-groups-and-network-acls-adc632ea29ae)
**Amazon Web Services 한국 블로그**
https://aws.amazon.com/ko/blogs/korea/vpc-flow-logs-log-and-view-network-traffic-flows/