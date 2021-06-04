<img src="https://s3.amazonaws.com/ee-assets-prod-us-east-1/modules/gd2015-loadgen/v0.1/media/image1.png" style="width:6.5in;height:7.6125in" />

유니콘을 다루는 설명서!
==========================================

조작법
----------------

<table>
<thead>
<tr class="header">
<th>Revision</th>
<th>Author</th>
<th>Date</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>.1</td>
<td>Mike</td>
<td>3/1/2015</td>
<td>draft!</td>
</tr>
<tr class="even">
<td>.2</td>
<td>Dave</td>
<td>3/14/2015</td>
<td>draft major updates and fixes</td>
</tr>
<tr class="odd">
<td>1.0</td>
<td>Dave</td>
<td>3/21/2015</td>
<td>final draft! So excited!</td>
</tr>
<tr class="even">
<td>1.1</td>
<td>Mike</td>
<td>3/24/2015</td>
<td>major fixes. Added Troubleshooting after fun with disk drives ;-)</td>
</tr>
<tr class="odd">
<td>2.0</td>
<td>Dave</td>
<td>4/1/2015</td>
<td>migrate to AWS. Removed Troubleshooting: no more physical hardware!</td>
</tr>
<tr class="even">
<td>3.0</td>
<td>Mike</td>
<td>4/2/2015</td>
<td>full rewrite of 2.0. Added Troubleshooting again. Dave is an optimist.</td>
</tr>
<tr class="odd">
<td>4.0</td>
<td>Dave</td>
<td>5/13/2015</td>
<td>full rewrite of 3.0. Added Disaster Recovery for obvious reasons, Mike.</td>
</tr>
<tr class="even">
<td>4.1</td>
<td>Mike</td>
<td>5/13/2015</td>
<td>minor updates to DR section based on the Event as mandated by mgmt.</td>
</tr>
<tr class="odd">
<td>5.0</td>
<td>Dave</td>
<td>5/16/2015</td>
<td>4.1 was a tissue of lies. Rewrote Disaster Recovery based on the latest Event. Last commit before the long nap.</td>
</tr>
<tr class="odd">
<td>5.01</td>
<td>Seunguk</td>
<td>11/04/2020</td>
<td>Translate for KOREAN</td>
</tr>
</tbody>
</table>


시작하기에 앞서
------------

Unicorn Rentals 웹 사이트는 암호화 된 강력한 해시를 서비스로 제공합니다.

그만큼 시스템은 클라이언트 애플리케이션과 독립형 서버로 구성됩니다. 시스템은 항상 주어진 키에 대해 동일한 해시 값을 반환합니다. 

시스템은 다음을 기반으로 해시를 빌드합니다. 

독점적이고 계산 비용이 많이 드는 해싱 알고리즘. 서버는 현재 단일 독립형 'go'바이너리로 배포됩니다. 추적 할 세션 상태 나 수행 할 외부 호출이 없으므로 서버 아키텍처는 단순한 수평 확장에 잘 반응합니다. 제한된 클라이언트 키 세트가있을 것으로 예상되므로 시스템은 캐싱 기술에 잘 반응해야합니다. 그게 힌트입니다.

범위  
-----  

이 런북은 Unicorn Rentals 웹 사이트를 구동하는 생산 시스템의 운영 이론과 실제를 설명합니다. 주요 대상은 사이트를 운영하는 DevOps 팀입니다.



DevOps 팀은 코드 배포,로드에 대한 응답으로 사이트 확장, 게시 된 SLA (응답 시간 및 가동 시간 포함) 유지 관리, 재해 복구, 문제 해결 활동 및 이러한 목표를 충족하는 데 필요한 모니터링 및 경고 활동을 담당합니다.

인프라 비용  
-------------------  

 여기 Unicorn Rentals에서는 돈을 벌려면 돈이 필요하다는 것을 알고 있습니다. _(EC2 사용 15 분마다  $ 8가 청구됩니다. 예를 들어 EC2 인스턴스가 하나만 실행중인 경우 3 분 후 종료하면 8달러가 청구됩니다.)_ 13 분 동안 2 개의 인스턴스를 실행하는 경우 $ 16가 청구됩니다. 따라서 필요한 것만 프로비저닝하십시오!

플레이어 대쉬보드
----------------

대시 보드는 https://dashboard.eventengine.run으로 이동하여 액세스 할 수 있습니다. 팀 해시를 입력하라는 메시지가 표시됩니다. 이 팀 해시는 오늘 초에받은 종이에서 찾을 수 있습니다.

대시 보드에 로그인하면 팀 이름을 설정할 차례입니다. 대시 보드에서 "팀 이름 설정" 버튼을 클릭하고 팀 이름을 입력하십시오. 이 팀 이름은 한 번 설정되면 변경할 수 없으며 점수 판에서 팀을 나타냅니다. Unicorn.Rentals의 점수 판 팀은 공정하게 대쉬보드에 공개됩니다.

그러므로 공개된다는 점을 염두에 두고 팀 이름을 만들어야합니다. 부적절한 팀 이름은 Unicorn.Rentals 이사회에서 이름을 변경합니다.  

<img src="https://s3.amazonaws.com/ee-assets-prod-us-east-1/modules/gd2015-loadgen/v0.1/media/dashboard1.png" style="width:6.7in;height:8.5in" />  

대시 보드에는 게임 내내 상호 작용할 몇 가지 주요 구성 요소가 있습니다.

1. 대시 보드의 상단 표시 줄에는 1) 점수 이벤트에 액세스, 2) 스코어 보드에 액세스, 3) 팀 이름 설정 (한 번만 수행 가능), 4) Slack 채널에 초대 요청을 수행 할 수있는 일련의 버튼이 있습니다. 팀 협업 및 채팅, 5) 팀의 AWS 계정에 액세스 (<-중요합니다!)

2. 이 막대 아래에이 게임에 활성화 된 모듈 목록이 표시됩니다. 각 모듈은 출력과 입력을 정의 할 수 있습니다. 위의 예에서 모듈에는 출력이 없지만 하나의 입력이 있습니다. 이 모듈의 경우 주문 처리자에 대한 URL을 제공하여 주문을 전송하고 포인트를 얻을 수 있습니다.

참고 : 팀 이름을 설정하고 UnicornRentals 컬렉 티브에서 팀 이름을 승인하면 대시 보드에 추가 옵션이 표시됩니다. 이러한 옵션에는 팀 점수, 순위 및 추세는 물론 "점수 이벤트"버튼을 통한 점수 판에 대한 액세스가 포함됩니다.


점수 이벤트와 점수 표기판
---------------------------

플레이어 대시 보드에서 "스코어 보드"링크를 클릭하면 스코어 보드 창이 열립니다. 스코어 보드는 룸에있는 모든 플레이어와 팀으로서의 진행 상황을 추적 할 수있는 곳입니다.

<img src="https://s3.amazonaws.com/ee-assets-prod-us-east-1/modules/gd2015-loadgen/v0.1/media/scoreboard1.png" style="width:6.7in;height:2.12569in" />

개별 팀의 성과를 더 자세히 보려면 포인트 별 분석에 액세스하려면 플레이어 대시 보드의 "점수 이벤트"버튼을 클릭하세요.

<img src="https://s3.amazonaws.com/ee-assets-prod-us-east-1/modules/gd2015-loadgen/v0.1/media/scoreevents1.png" style="width:6.7in;height:4.5in" />

이 페이지에는주의해야 할 두 섹션이 있습니다.
1.  각 행에는 팀이 생성 한 모든 점수 이벤트가 나열됩니다. "Source" 열은 포인트 적립 또는 공제 출처를 알려줍니다. "Point" 열은 얼마나 많은 포인트를 얻거나 잃었는지 알려줍니다.
2. "Reason" 열에는 포인트를 얻거나 잃은 이유가 표시됩니다. 무슨 일이 일어나고 있는지, 문제를 해결하는 방법을 이해하기 위해 점수를 잃을 때이 열에 매우주의를 기울이십시오.

참고
----------

1.  서버 애플리케이션은 다음에서 컴파일 된 'go'바이너리로 배포됩니다. 소스가 github 저장소에 저장된다는 소문이 있습니다. 그러나, 이 저장소의 이름은 현재 운영 및 개발 직원에게 알려지지 않았습니다.
  
2.  서버 애플리케이션은 이전에 약 5 개의 연결을 처리 할 수 있습니다. 정말 느려지기 시작합니다. 과부하에주의하고 대기열이 채워질 때 503 초를 확인하십시오. 백업이 시작되면 앱 또는 서버를 다시 시작할 수 있습니다.
  
3.  서버 애플리케이션은 x86 정적으로 링크되고 스트리핑되지 않은 ELF입니다. 여기에서 찾을 수있는 실행 파일 : https://s3.amazonaws.com/ee-assets-prod-us-east-1/modules/gd2015-loadgen/v0.1/server
  
4.  우리가 선택한 기본 OS는 [Amazon Linux] (https://aws.amazon.com/amazon-linux-ami/). 이 배포는 광범위한 업계 지원, 안정성, 지원 가용성 및 AWS와의 탁월한 통합. 이 배포판은 플랫폼 강화에 대한 요구 사항에 따라 SecOps에서 선택했습니다.
  
5.  서비스 출시 계획의 일환으로 아키텍처가 AWS로 이전되었습니다.
    AWS CLI 작동 메뉴얼이 도움이 될 것입니다.
    (<http://docs.aws.amazon.com/cli/latest/userguide/installing.html>)

<img src="https://s3.amazonaws.com/ee-assets-prod-us-east-1/modules/gd2015-loadgen/v0.1/media/image5.png" />

Unicorn\_High\_Level\_AWS

1.  AWS로 작업 할 때 다음 역할 만 허용됩니다. SecOps ... 및 재무 :
-   ec2
  
-   s3
  
-   ebs
  
-   ecs
  
-   eks
  
-   cfn
  
-   elasticache
  
-   cloudwatch
  
-   sns
  
    -   systems manager
    
-   cloudtrail
  
-   config
  
-   vpc
  
-   cloudfront
  
-   lambda
2.  이 어리석은 일을 쓰는 것이 지루해집니다. 누가 필요합니까?

서비스 시스템 구조
-------------------------------

Client &lt;-&gt; Server, 여기에 또 무엇이 필요할까요?

<img src="https://s3.amazonaws.com/ee-assets-prod-us-east-1/modules/gd2015-loadgen/v0.1/media/image6.png" alt="Unicorn_Data_Flow" style="width:5.95in;height:3.94091in" />

Unicorn\_Data\_Flow

온-라인 운영
------------------

### 유니콘 코드 업데이트

-   서버 코드는 AWS에서 '사용자 데이터'로 제공하는 스크립트를 통해 처음 부팅 할 때 다운로드됩니다.
  
-   사용자 데이터는 Auto Scaling 그룹 'Launch Configuration ':

<!-- -->

-   $ aws autoscaling describe-launch-configurations --query
    LaunchConfigurations\[0\].UserData | base64 --decode

> #!/bin/bash
>
> wget **\*binary location (reference 3)\***    
> chmod +x server  
> ./server 
> \# Reboot if the server crashes  
> shutdown -h now
>
> 다음은 AutoScaling 그룹 시작 구성의 개요입니다.  
> <img src="https://s3.amazonaws.com/ee-assets-prod-us-east-1/modules/gd2015-loadgen/v0.1/media/image7.png" alt="Unicorn_AutoScaling_Flow" style="width:2.16538in;height:5.81947in" />

1.  이전 구성을 기반으로 새 시작 구성 만들기
    (http://docs.aws.amazon.com/AutoScaling/latest/DeveloperGuide/WorkingWithLaunchConfig.html)
2.  'UserData'섹션에서 S3에서 다운로드 한 서버 코드의 위치를 업데이트합니다.
  
3.  이 새로운 시작 구성을 Auto Scaling 그룹과 연결합니다.
4.  Scale-up new group / scale-in old group
5.  인스턴스가 시작시 S3 버킷을 참조할 수 있는지 확인(default
    route via IGW in subnet).

### Add ssh-key to instance

1.  ssh-keypair 만들기
    (http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)

2.  Launch Configuration 업데이트하기 ('유니콘 코드 업에데이트'와 동일한 절차입니다)

3.  예, 이는 인스턴스를 다시 시작해야 함을 의미합니다.

### Update Auto Scaling Policy

> 그리고 이것을 살표보세요!
> http://docs.aws.amazon.com/AutoScaling/latest/DeveloperGuide/scaling_typesof.html

Troubleshooting Procedures
--------------------------

### 네트워킹 동작

> 애플리케이션이 다음을 수행하려면 AWS VPC / ELB 환경이 정상이어야합니다.
> 작업. 모든 프로덕션 트래픽은 수신 및 수신 모두에서 ELB를 통해 이동합니다.
> 동작아키텍쳐 :

<img src="https://s3.amazonaws.com/ee-assets-prod-us-east-1/modules/gd2015-loadgen/v0.1/media/image8.png" alt="Unicorn_Detail_Network" style="width:5.95in;height:4.38829in" />

유니콘 렌탈 네트워크 디테일 체크방법 :

1.  인스턴스에 대한 보안 그룹 설정 확인

    1.  필요한 모든 포트가 허용되는지 확인

2.  서브넷의 라우팅 테이블 확인

    1.  라우팅 테이블이 각 서브넷에 적용되었는지 확인
    2.  '기본' 라우팅 테이블은 명시없이 모든 서브넷에 적용됩니다.
    3.  라우팅 테이블에 적절한 규칙이 있는지 확인
    
3.  마지막 6 - 12개월 동안 Ops가 중단 한 VPC에서 확인할 사항-
    
    1.  인스턴스가 작동합니까? 자세한 내용은 "http://\<\<Your-IP-or-DNS\>\>/healthcheck"를 시도하십시오.
    
    <!-- -->
    
    2.  Auto Scaling 그룹에서 인스턴스 '업/다운' ?
    
    <!-- -->
    
    3.  Subnets?
    
        1.  서브넷 'cider???' 이라는것인가 뭔가의 충분한지 체크??라고 전달받음
        2.  Elastic Load Balancer에 서브넷이 추가 되었습니까?
    3.  서브넷이 Auto Scaling 그룹에 추가됩니까?
    4.  라우팅 테이블이 정확하거나 손상되지 않았습니까? 다이어그램을 참조하십시오.
    5.  ACLs: Subnet에 적용??? 그게 제한을하거나 허용을 하나요?
    6.  Security groups?
    7.  IGW?를 통해 트래픽을 흐르게하는 경로가 있습니까?
        참고로 S3에서 서버 어플리케이션 코드를 가져옵니다.
    8.  Route53 records? 올바른 타겟을 향하고 있나요?

-   새로운 어플리케이션 테스트 유틸리티!

> https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/gd2015-loadgen/v0.1/server-bang.linux  
> https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/gd2015-loadgen/v0.1/server-bang.osx  
> https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/gd2015-loadgen/v0.1/server-bang.windows  
>   
> ./server-bang.$arch &lt;server URL&gt;

-   인스턴스에 ssh로 접근하여 어플리케이션을 확인 할 수 있습니다. --
    > 먼저 포트 80에서 동작하는 것만 알고, 키페어를 활용해야한다는 것만 알고 있습니다.
    
-   서버 프로세스 핸들링이 많아지는경우, 서버가 느려질 수 있습니다.
  
    > 많은 연결이 몰리는 경우 재시작할 필요가 있습니다. 

Change Management
-----------------

무야호!

System Monitoring
-----------------

-   모니터링하는 링크: <https://scoreboard.eventengine.run>

-   ELB의 메트릭체크하는 방법 :  
    http://docs.aws.amazon.com/AutoScaling/latest/DeveloperGuide/policy_creating.html
    
    http://docs.aws.amazon.com/ElasticLoadBalancing/latest/DeveloperGuide/elb-cloudwatch-metrics.html

To Do
-----

-   Elasticache, cloudfront 살펴보기 : 이것이 속도를 높일 수 있지 않을까요?!
  
-   고가용성 테스트는? DR은 구성하셨나요?

-   컨테이너 아키텍처를 사용하여 인프라 리팩터링
