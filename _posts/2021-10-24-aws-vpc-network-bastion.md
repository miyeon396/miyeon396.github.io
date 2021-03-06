---
layout: single
categories: Aws
title: "[Aws] VPC, Network, Bastion"
---

---
### VPC

 - 신규 리전 외의 2019년도 생성 기존 리전은 vpc 생성 시 기본적으로 생성이 됨
 - 기본 VPC는 삭제하고 시작. (스크립트 짜놓고 한방에 지우던가 그런식으로 함)
 - 네트워크 인터페이스나 ec2 같은게 있음 vpc 안지워지는데 콘솔에서는 ec2 디폴트면 일단 지워짐
 - vpc 생성 시 기존 대역대와 안겹치게 쓰는것이 좋음

> vpc cidr 영역 나눠서 쓰는 이유? 
> - 계정 자체를 나누기도 하지만 vpc를 나눠서 각 사용 용도를 분리하기 위해

---
### Subnet


##### 서브넷 생성

 - 퍼블릭 서브넷
   필요도 없고 보안적으로 좋지도 않음 
   이 서브넷에서 인터넷게이트웨이 연결되서 접속할 수 있기 때문에 ec2가 자기 ip 달고 나갈 수 있는 설정이 되어있는 서브넷
    ex) 베스천 서버 - 프라이빗에 넣을수도 있지만 퍼블릭하게 여기에 넣어 놓는게 구분하기가 좋음.
- 프라이빗 서브넷 : 3tier web application 사용 중 (WEB + API + DB) 
    1. web
    2. api(app)
    3. db
    4. data
- multi-az  -> az-a에 1세트 , az-c에 한세트 -> 총 2세트 생성

> 서브넷 이름 짓기 tip
> - 아이디를 외우고 다닐게 아니라면 끝에 a존이냐 b존이냐 명시해주면 편리 
> ex) SN-PRD-PRIVATE-WEB-A


##### 서브네팅
- 서버들을 위치시킬 서브넷 만들고 대역들을 나누는 것
- 사이더를 얼마로 나누고 이 서브넷에 얼마의 몇개의 ip를 할당할거냐
- cidr 계산기 참고해서 진행 하면 편리
- 서버나 용도에 맞게 분류해서 서브넷 구성해야함
- 서브넷 하나마다 아마 대략 3개정도의 ip는 AWS의 예약 IP임
- 사이더 블록은 2의 배수로 찍을 수 있음 -> 아이피 64개정도 할당하고 싶다 /26

> 서브넷팅 tip
> - 보통 /24로 두개 받앗다면  A존에 하나 C존에 하나 두개 아예 나눠서 서브넷 구성 여유분이 비슷하게 남도록
> - 퍼블릭은 제일 적게 할당, 서비스 특성을 고려하여 웹서버보다 배치성 백엔드성인 데이터 쪽에 많은 할당이 들어가면 좋음

---
### 라우팅 테이블

##### 라우팅 테이블 생성 
- 서브넷 티어별로 한개씩 만듦
- 프라이빗 서브넷의 경우 az별 2개씩 만드는 이유 -> 장애나 재해 발생 시 서비스가 유지가 되어야 하기 때문에 2개의 Routing Table 놓고 트래픽이 흐르도록 구성
- 어디로 향하게 설정하는가가 중요. 만들고 각 라우팅 테이블 별 생성해놓은 서브넷 연결.
- 예제
	1. RT-PRIVATE-WEB (nat필요)
 인터넷 접속할 필요가 있을까? 없음.
사용자 요청 소스를 프론트엔드에 주거나 백엔드 로드밸런서에게 전달하는 역할 인터널로 구성되어 있어서 프라이빗 아이피로 통신이 되기 때문에 필요가 없다
보통은 필요 없지만 파게이트 사용 시 헬스체크 주고 받기 때문에 얘에서 나갈 수 있는 아이피가 필요함. http 퍼블릭 으로 통신하기 때문에
우분투에서 인터넷 통해서 패키지 설치해야 할때 
사실 위의 특수케이스 제외하고는 웹 서버는 사실상 nat가 필요가 없다 
	2. RT-PRIVATE-APP 인증 통신 등 (nat필요)
	3. RT-PRIVATE-DB (nat필요)
	4. RT-PRIVATE-DATA (nat필요)
	5. RT-PUBLIC

##### 나트 필요 유무에 따라 라우팅 테이블 편집
- 할당 받은 ip 대역은 local 이고, 나머지 애들이 0.0.0.0 nat-gateway 생성한 것으로 향하도록 추가 필요(web, app, data)
- 퍼블릭 같은 경우는 인터넷으로 나갈 수 있는 환경을 만들어 주기 위해 igw 만들어 놓은 것을 라우팅 테이블 연결

---
### NACL (Network Access List)
- 서브넷을 생성하면 생성한 모든 서브넷이 다 연결 대상으로 설정되어있음.
- 따로 지정을 하지 않는 이상 기본적으로 vpc 만들때 적용됫던 nacl이 적용이 되고, 서브넷 별로 무언가를 막아야겟다 할때, 그 서브넷에만 acl 설정해서 진행하면 됨
- 외부 -> vpc -> subnet1 / 2/ 3/ ...5 (nacl이 vpc와 서브넷 사이를 끊어 버리는 거)
- 번호가 낮을 수록 먼저 적용됨
- 전체 서브넷에 공통 규칙 적용하고 싶을 때 사용

---
### NAT Gateway (Network Address Translation Gateway)
> Private의 리소스들이 Interet과 통신하기 위해 나갈 수 있는 통로 -> 외부통신!

- 엣날엔 나트 인스턴스엿는데 나트 게이트웨이로 이름이 바뀜
- 외부 통신 : 인터넷 -> 퍼블릭으로만 통신 -> 퍼블릭 ip가 있어야 통신 가능
- vpc안에서는 프라이빗 IP만으로도 통신을 할 수 있는데, VPC 외부로 나가려면 퍼블릭 IP를 달고 나가야함
- 우리 망에서 나가는 애들은 하나의 아이피달고 나감 
 ex) 같은 사무실에 근무하는 사람들은 다같이 외부 네이버나 구글 사용 시 같은 아이피를 달고 나간다.
- 프라이빗 서브넷 
	- 구성 자체가 각자가 프라이빗 아이피만 갖고있는 애들만 놓음
	- 퍼블릭에 접속하지 못하는 애들을 모아서 막아 놓음
- 퍼블릭 서브넷 
	-  VPC 내부로 들어가려면 Bastion이라던가, Public 서브넷에 있는 애를 통해서 들어가야 한다
- -> AWS API로 갖고 온다던지, 환율 정보 갖고 온다던지 서브넷들에 있는 통로를 나트에 모아서 하나의 공통된 통로를 이용해 인터넷에 접속

##### NAT Gateway 생성
- 서브넷은 퍼블릭 영역에 만들어주고 연결 유형은 퍼블릭
- eip를 신규로 하나 할당을 해줌 (+ eip 이름 알아볼 수 있도록 변경)
-> NAT Gateway + eip 생성 까지 완료 
- NAT Gateway가 실제로 게이트웨이같지만 내부적으론 인스턴스가 들어가니까 생성 시 약간 시간 소요


> NAT Gateway 용도
> - 나트는 우리가 인터넷에 접속해서 가져오기위해서 사용 (누가 우리 서비스에 접속하기 위해 사용하는게 아님)
	> ex) ubuntu에서 패키지 같은거 설치할 때도 나트 통해서 나감
> - VPC 내의 배스천은 직접 Public IP 갖고 나가지만 그 외의 다른 서버들은 Public Subnet의 NAT를 통해서 IP를 달고 외부로 나가게 된다
> - 참고) 사용자가 서비스에 처음으로 들어와서 첨으로 만나는건 로드밸런서고, 각각의 서버들에게 타겟그룹 통해서 갖다줌 (즉, 들어오는거랑 나가는거랑은 다르며 NAT는 나가는 용도이다)

---
### Internet Gateway
> 인터넷에 나갈 수 있는 환경
-  IGW는 VPC에 붙어있다. -> 통로로 사용하겠다, IGW를 통해서 인터넷으로 나가겠다.
	(=VPC 안에 있는 애들은 IGW 통해서 통신을 한다.)
- IGW가 없으면 IP가 있다하더라도 밖으로 나갈 수 없다.
- IGW 까지 오려고 해도 Public IP를 갖고 있어야한다(Bastion은 자기 EIP, NAT는 NAT Gateway의 EIP)
- 서브넷의 라우팅 테이블내
	- 퍼블릭에 있는 애들은 로컬이 아닌 이상 모든 IP가 인터넷에 연결
	- 프라이빗에 있는 애들은 퍼블릭 아이피를 달아야하는 과정이 필요하므로 NAT를 먼저 들린다. 그럼 NAT에서 퍼블릭 아이피 단 상태에서 IGW 통해서 나갈 수 있는 상태가 된다.
 
##### 인터넷 게이트웨이 생성
- 생성 후 VPC에 연결
	- 인터넷 게이트웨이 연결 유무에 따라 인터넷 통신이 필요한 VPC구나 아니구나 판단 가능
- 퍼블릭으로 만들어 놓은 라우팅 테이블(=인터넷 연결이 필요한 친구들)에 IGW 추가해줌
 - 라우팅 테이블에 0.0.0.0 으로 연결 
 - 라우팅 테이블에 추가할 애들은 인테넷 연결이 필요한 친구들
 - 인터넷 연결이 필요 없는 나머지 프라이빗 친구들은 IGW 생성 후 NAT도 생성 한 후에 라우팅 테이블에 NAT를 연결해준다.

---
### Bastion

##### Bastion용 EC2 인스턴스 생성
- 퍼블릭 서브넷
- Bastion에는 eip 설정 필요. 한개 할당 받아서 Bastion에 붙여줌
- 접속하기 위해서는 보안그룹 필요 만들어줌 
ex) ssh (22) 로 접속할 곳(사무실, 집 등) 인바운드 열어줌
- 접속 후 Private Subnet의 EC2 인터넷 접속 test
  1. Bastion 접속
  2. ssh -i key-ec2-key.pem ubuntu@10.10.10.100 : ssh를 통한 ec2 접속
  3. ping google.co.kr
    - 된다? Public Routing Table에 Nat를 잘 연결했다. Private Routing Table에 IGW를 잘 연결했다.
    - 안된다? 뭔가 Routes를 빼먹었다. 다시 체크


TODO :: 베스천에 otp 거는법
TODO :: SSM, CW Agent 설치 법 정리

---
### 총정리

1. VPC를 생성한다.
2. 할당 받은 IP들을 잘 사용하기 위해 서브넷 구성 및 서브네팅을 한다.
3. Private Subnet 내의 Server들이 외부와 통신을 하기 위해 Routing Table, IGW, NAT Gateway를 생성한다.
	- 프라이빗 서브넷에 있는 애들은 퍼블릭 서브넷의 NAT 통해서 IGW 통해서 나감
4. 외부 통신을 위해 Bastion Server를 하나 구성한다.
	- Bastion은 퍼블릭 서브넷에 있고 본인 IP가 있기 때문에 바로 IGW통해서 바로 나감
