# [AWS Builders 100] Cloud Pricing

# [AWS Builders 100] 클라우드 비용, 어떻게 줄일 수 있을까?

# 목차

1. Gather data
    - Billing Dashboard
    - Cost and Usage Reports(CUR)
    - Trust Advisor
    - 비용 탐색기(Cost Explorer)
    - Tagging
2. 비용 절감방법
    - Right Sizing
    - Instance Scheduler
    - Pricing

        1) 인스턴스 가격 옵션

        2) 스토리지 비용

        3) 이외 비용

3. Take Action

# 1. Gather data

---

## Billing Dashboard

클라우드 비용 절감의 첫 번째 단계는 비용에 대한 데이터를 수집하고 어디서 비용이 많이 발생하고 있는지 파악하는 것이다.

빌링 대시보드에서는 서비스별, 기간 별로 얼마나 비용이 청구되었는지 파악할 수 있다.

전체적인 사용량을 이해하는 데 있어서는 빌링 대시보드가 도움이 된다.

## Cost and Usage Reports(CUR)

하지만 AWS는 시간당 과금 체계이기 때문에 보다 상세한 사용량 데이터가 필요한 고객이 있을 수 있다.

AWS 요금의 디테일한 리스트는 '비용 및 사용량 보고서 (CUR) '을 통해 파악할 수 있으며, CUR 데이터를 받기 위해서는 빌링 대시보드→ Cost and Usage Reports 에서 생성할 수 있다.

데이터 확인을 마쳤다면 이에 대한 인사이트가 필요하다.

## Trust Advisor

옆에서 중요한 데이터를 짚어주고, 모은 데이터에 대한 인사이트를 쉽게 파악할 수 있도록 도와준다면 편리할 것이다.

전반적인 인프라 환경을 선생님처럼 점검해주는 서비스로 Trust Advisor 가 있다.

리소스를 효율적으로 사용하고 있는지, 추가적으로 점검할 보안 항목은 없는지 점검해주는데,

비용 최적화, 성능, 보안, 내결함성 서비스 한도까지 다섯개 항목으로 점검한다.

![%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled.png](%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled.png)

![%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%201.png](%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%201.png)

초록색은 문제가 없다는 의미, 노란색은 점검이 필요한 항목, 빨간색은 지금 즉시 조치가 필요한 항목

![%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%202.png](%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%202.png)

기본적으로 Trust Advisor는 무료로 제공하는 서비스이며, 기본 서비스 이상을 이용하면 더 상세한 분석 결과를 스프레드시트 파일로 받아볼 수 있다.

## 비용 탐색기(Cost Explorer)

![%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%203.png](%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%203.png)

AWS 비용을 시각화해서 빠르게 비용 문제점을 한 눈에 파악할 수 있다.

## Tagging

아직 한 가지 더 수집해야 할 정보가 남았다.

누가 어떤 리소스의 주인이고, 어떤 목적으로 사용중인지 파악해서 비용을 절감할 필요가 있다.

이를 가능하게 하는 것이 바로 태깅(Tagging).

태깅은 그 중요성을 아무리 강조해도 모자라지 않을 정도인데,

AWS는 거의 모든 리소스에 태깅이 가능하며 태깅은 비용관리에 꼭 필요하기 때문이다.

프로젝트를 시작하기에 앞서 팀원들과 태깅 정책을 세워 체계적인 리소스 관리를 할 수 있다.

# 2. Apply methods(비용 절감 방법)

---

## Right Sizing

![%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%204.png](%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%204.png)

AWS 인스턴스는 옷 사이즈처럼 구분되어 있다.

과거 온프레미스 환경에서는 언제나 peak를 예상하고 리소스를 크게 잡아서 인프라를 구축했지만 클라우드 환경에서는 오토스케일링을 통해 수요와 맞춰 최소한의 비용으로 애플리케이션 needs를 충족할 수 있다.

![%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%205.png](%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%205.png)

AWS에서도 직접 돌려보고 확인한 뒤 Right Sizing 하기를 권장하고 있다.

![%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%206.png](%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%206.png)

고객의 워크로드에 맞춘 다양한 인스턴스 타입 제공

## Instance Scheduler

[https://aws.amazon.com/ko/solutions/implementations/instance-scheduler/](https://aws.amazon.com/ko/solutions/implementations/instance-scheduler/)

예를 들어, 실 서비스 목적인 인스턴스 A와, 똑같은 테스트환경인 B가 있고 이들은 일주일 24시간 내내 가동된다고 가정하자.

인스턴스의 동작 시간을 조절 하려면

- 서비스 인스턴스 A는 항상 켜져있어야 하므로 어쩔 수 없이 가동해야 한다
- B는 테스트 서버이므로, 출근하지 않는 주말에는 사용하지 않는다
- 마찬가지로 평일 퇴근시간 이후에도 B는 가동할 필요가 없다

이와 같은 기준을 세우고 AWS instance Scheduler를 이용할 수 있다.

인스턴스의 수동적 관리도 가능하지만 그 수가 많아지면 많아질수록 직접 관리에는 어려움이 따른다.

Instance Scheduler도 CloudFormation으로 관리되기 떄문에 실행이 빠르고 간편하다. 실행이 완료되면 2개의 DynamoDB 테이블이 자동으로 생성되는데 ConfigTable에서는 시간과 요일, 태깅 등 기본적인 설정이 가능하다. Instance Scheduler는 EC2와 RDS 모두 사용 가능하다.

여기서 Tagging이 중요한 이유를 알 수 있는데, Instance Scheduler는 인스턴스에 설정된 태깅에 따라  설정값에 따라 스케줄링을 진행한다.

예컨대, 한국시간 월요일에서 금요일 9시~18시까지만 인스턴스를 동작시킬 수 있다.

## Pricing

### 1) 인스턴스 가격 옵션

---

![%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%207.png](%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%207.png)

세 번째 단계는 AWS에서 제공하는 다양한 가격 옵션을 설정하는 것이다.

AWS에서는 온디맨드, 예약 인스턴스, 스팟 인스턴스, 전용 인스턴스까지 총 4가지 유형의 인스턴스 가격 옵션이 존재한다.

**[1] 온디맨드 인스턴스**

- 연 단위 기간 약정으로 온디맨드 대비 75% 저렴한 비용
- 장기적 이용
- 트래픽을 예측할 수 없고 변동성이 심한 경우

**[2] 예약 인스턴스**

- 표준, 컨버터블 두 개의 클래스
    1. 표준(Standard) : 한 번 구입시 변경이 불가능하지만 할인율이 높음
    2. 컨버터블(Convertable) : 구매한 후에도 인스턴스 Familiy, OS, 테넌시 등 유연한 변경이 가능하나 할인율이 떨어짐
- 기간은 1년 또는 3년 중 선택 가능
- 전체 선결제, 부분 선결제, 선결제 없이 약정 할인 세 가지 결제 옵션

![%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%208.png](%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%208.png)

![%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%209.png](%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%209.png)

**[3] 스팟 인스턴스**

AWS는 전 세계에 수 많은 데이터센터를 갖고 있는 만큼, 사용되지 않고 낭비되는 자원들이 있는데,

AWS는 이렇게 이용되지 않는 자원들을 고객에게 스팟 인스턴스로 제공하고 있다.

EC2 온디맨드 대비 최대 90% 저렴하며 일시 중지, 휴면, 재시작 등이 자유롭다는 특징이 있다.

단, 스팟은 AWS의 예비 리소스인만큼  AWS에서 필요시 회수하면 종료될 수 있다는 리스크가 있는데 이는 AWS 내부의 툴이나 Tirth Party 솔루션을 통해 보완할 수 있다.

과거에는 스팟 인스턴스의 가격이 입찰 형태라서 잘못 구매하면 온디맨드보다 더 높은 경우도 있었으나,  이제는 AWS가 가지고 있는 컴퓨팅 리소스의 수요와 공급에 따라 스팟 인스턴스를 제공하게 되면서 스팟 인스턴스의 가격이 안정화 되었다.

스팟 인스턴스의 인스턴스 유형에 따른 중단 빈도를 미리 확인할 수 있다.([https://aws.amazon.com/ko/ec2/spot/instance-advisor/](https://aws.amazon.com/ko/ec2/spot/instance-advisor/))

**[4] 전용 인스턴스**

다른 AWS 계정의 인스턴스로부터 물리적으로 격리된 전용 하드웨어를 사용한다. 즉, 단일 고객 전용의 VPC에서 인스턴스가 실행된다.

과금은 시간당 인스턴스 사용요금과 리전당 전용 요금이 청구되는데, 리전당 전용 요금은 실행중인 전용 인스턴스의 수에 상관 없이 청구된다.

전용 호스트도 같은 의미로 사용되는 용어인데, 이들 간에는 약간의 차이점이 존재한다.

* vs 전용 호스트

전용 호스트를 사용하면 특정 물리 서버의 VPC 에서 인스턴스를 시작할 수 있다는 점은 동일하다. 비유하자면 전용 인스턴스와 전용 호스트는 단독주택이고, 일반 인스턴스는 아파트에서 특정 호 수만 쓰는 개념이다. 전용 인스턴스와 전용 호스트를 사용하면 AWS의 하드웨어 장비 1개를 전용으로 사용하는 것이고, 공유 인스턴스를 사용하면 AWS의 하드웨어 장비를 여럿이 나눠 사용한다고 볼 수 있다. 하지만 전용 인스턴스와 전용 호스트는 차이가 존재한다.

전용 인스턴스는 필요할 때 AWS의 하드웨어 장비들 중 사용중이지 않은 랜덤한 장비를 한 대  점유하는 것인데, 인스턴스 재시작 시 꼭 같은 장비에서 다시 실행된다는 보장이 없다.

그러나 전용 호스트는 전용 호스트를 해제하기 전 까지는 계속해서 같은 장비에 호스팅된다. 따라서 특정 머신에서 상용 소프트웨어를 사용하기 위해 라이센스를 받아두었더라도 장비가 변경되어 라이센스를 새로 적용할 필요 없이 사용하던 라이센스를 그대로 사용할 수 있다. 호스트를 해제하고 다시 할당하기 전 까지는 같은 장비를 사용한다.

이를 통해 기업 규정 준수 및 규제 요구 사항을 충족하는 인스턴스를 배포할 수 있다. 또한 전용 호스트를 사용하면 EC2 자체 소켓당/코어당 소프트웨어 라이센스를 사용함으로써 라이센스 비용을 절감할 수 있다. 

### 2) 스토리지 비용

---

스토리지는 크게 오브젝트 스토리지와 블록 스토리지로 구분된다.

스토리지도 워크로드에 맞는 Right Sizing이 중요하다.

**[1] 오브젝트 스토리지(S3) 클래스 비교**

![%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%2010.png](%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%2010.png)

 

S3 : 가장 기본

S3 Inteligent-Tiering : 파일이 얼마나 자주 쓰이는지 잘 모르는 경우 AWS가 알아서 분석하고 접근 빈도에 따라 알아서 티어를 변경

S3-IA : S3와 동일하지만 자주 접근하지 않는 데이터

S3-OneZone : IA와 동일하지만 하나의 가용영역만 사용하기에 가격이 더 저렴

Glacier : 자주 액세스하지 않는 아카이빙용

**[2] 블록 스토리지(EBS) 클래스 비교**

![%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%2011.png](%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%2011.png)

EBS는 오브젝트 스토리지보다 비용이 더 높다. 따라서 EBS 사용 전에 해당 워크로드가 IOPS가 중요한지, Throuput이 중요한지 확인해야 한다.

이를 잘 모르겠다면 범용 SSD인 gp2(gp3)를 사용하는 것을 권장하며,

프로비저닝된 IOPS는 매우 빠르지만 비용이 매우 비싸므로 범용SSD로 할 수 있는 부분이 없는지 한 번 더 살펴보아야 한다.

S3는 동적으로 스토리지 용량이 확장되지만 EBS는 OS 및 파일시스템 구현에 따라 제한된다.

따라서  맞는 클래스를 선택하는 것이 EBS 비용을 줄일 수 있는 핵심이다.

**[3] 이외 비용**

CDN을 사용한 데이터 Out 비용 절감

Lambda로 서버리스 아키텍처 구성

상용 DB보다는 오픈소스 DB를 사용함으로써 라이센스 비용 절감

# 3. Take Action

---

올바른 액션을 취하기 위해서는 그에 따른 Goal과 Metric이 필요하다.

이에 따라 AWS에서는 고객에게 비용 Target을 챙길 것을 권장하고 있다.

지표는 올 해 IT 예산이 될 수도, 6개월 AWS 사용량이 될 수도 있다.

![%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%2012.png](%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%2012.png)

예를 들어

1. 매일 종료되는 인스턴스(Metric)의  비율을 30% 이상 유지하는 것을 목표(Target)으로 잡았음

7월에는 전체 인스턴스 중 32%가 매일 종료되었으므로 양호

반대로 

5. 태깅이 되어있지 않은 리소스(Metric)를 5% 미만으로 유지하려고 목표(Target)를 설정함

하지만 실제 태깅이 되어있지 않아 무슨 목적으로 사용되고 있는지 모르는 리소스가 18%나 있으므로 개선이 필요

![%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%2013.png](%5BAWS%20Builders%20100%5D%20Cloud%20Pricing%209ca05b1c3fb24a6cb1ba554016285a4b/Untitled%2013.png)

이렇게 타겟을 설정하여 현황을 분석한 후, 최소한의 노력으로 큰 비용 절감 효과를 볼 수 있는 것 부터 차근차근 현재 가능한 조치(Action)를 취하는 것을 권장한다.