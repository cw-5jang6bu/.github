# Cloud Wave 4기 1조 올리브영 쇼핑몰 인프라 구축
![슬라이드1](https://github.com/user-attachments/assets/ba9aa972-4c63-4378-8468-fa11bf6ce0d9)


## 👨‍🏫 프로젝트 소개
올리브영에서 90% 할인 쿠폰을 지급하는 이벤트를 진행합니다. 대규모 트래픽이 몰릴것을 대비해 시스템 및 쿠폰 발급/조회 서비스를 구축했습니다.

## ⏲️ 개발 기간 
- 2025.02.7(금) ~ 2025.02.27(목)

  
## 🧑‍🤝‍🧑 구성원 소개 및 역할
<div align="center">

|  송종인  |  박성훈  |  김경민  |  문병헌  |  이지영  |
| :-----: | :-----: |  :-----: |  :-----: |   :-----: | 
| <img width="116" alt="송종인" src="https://github.com/user-attachments/assets/04ffd101-0699-49bd-8ca4-04a8f9432aa3" /> | <img width="116" alt="박성훈" src="https://github.com/user-attachments/assets/2d95ede3-1867-4ee8-a0a7-44c86f94f9dd" />| <img width="115" alt="김경민" src="https://github.com/user-attachments/assets/9e61b2ae-3385-4cbe-82ce-118c4943000d" />| <img width="116" alt="문병헌" src="https://github.com/user-attachments/assets/7ebcbd47-e931-4da4-a4e1-d30428afab53" />| <img width="116" alt="이지영" src="https://github.com/user-attachments/assets/3ecf1a86-aa0c-4a83-a9cc-494eabf198d0" />
|[@Son-github](https://github.com/Son-github)| [@smile9855](https://github.com/smile9855)| [@gyungmean](https://github.com/gyungmean) | [@MBH992](https://github.com/MBH992) | [@lakedata](https://github.com/lakedata) | 
| Frontend<br>서브넷 구성 | EKS 구축<br>ArgoCD 배포 | 쿠폰 서비스 개발<br>트래픽&데이터 관리<br>CI Jenkins | EKS<br>모니터링&Q/A | 쿠폰 서비스 개발<br>서브도메인 분리<br>CI Jenkins
 
</div>


## 📌 프로젝트 배경
![e69b6b167d4a20702b7dd800171abb18-4](https://github.com/user-attachments/assets/5aa71640-f9b0-4b9c-8add-2ee8674fe86c)
![슬라이드6](https://github.com/user-attachments/assets/6ada9e20-6043-462d-851d-76a0f9186f59)


## ⚙️ 주요 선정 서비스
![Image](https://github.com/user-attachments/assets/6f75661c-977e-4573-8e17-607f78ea6909)

## 📝 Architecture
![Image](https://github.com/user-attachments/assets/28161e59-12c9-40e2-900f-0f8f941de8b3)

저희가 설계한 전반적인 아키텍쳐는 다음과 같습니다.
크게 DEV, PROD 그리고 DR계로 구성되어있습니다.

## 📝 Architecture - DEV
![Image](https://github.com/user-attachments/assets/55a69d09-a754-4ba8-b9d6-6b6ebbe159ce)

개발계는 가상 시나리오 테스트를 진행하여 실제 운영이 되기 전 문제가 없는지 검증했습니다.
개발자가 깃허브에 코드를 올리면 웹 훅을 통해 젠킨스로 코드를 푸쉬하게 됩니다.
알림을 받은 젠킨스는 이를 감지하고 빌드 파이프라인을 테스트합니다. 그리고 테스트된 이미지 도커파일을 빌드하여 ECR로 푸쉬합니다.

## 📝 Architecture - PROD
![Image](https://github.com/user-attachments/assets/a0ffe962-3471-444d-8404-1bc76601d763)

## 📝 Architecture - Event
저희 시스템에서 핵심적인 중추가 되는 이벤트계입니다.
이벤트계는 쿠폰 발급계와 쿠폰 만료계로 구성되어있습니다.

![Image](https://github.com/user-attachments/assets/8766074a-fb6e-45e0-8ef2-452ab5b64c90)

저희 시스템에서 가중 중요한 이벤트계입니다.
이벤트계는 쿠폰 발급계와 쿠폰 만료계로 구성되어있습니다.

- 쿠폰 발급

1.	메인도메인과 서브도메인을 분리하여 쿠폰 발급자의 트래픽이 서브도메인을 통해 가도록 합니다. 쿠폰 발급 사이트가 정적페이지이기 때문에 CloudFront를 통해 데이터를 정적 캐싱하고 글로벌 서비스인 점을 이용하여 접근성을 높였습니다.

2.	그 다음 WAF에 의해 매크로를 차단하여 과발급이나 중복발급을 방지할 수 있도록 했습니다. 그리고 API Gateway 통해 온라인 쿠폰 서비스와 오프라인 쿠폰 서비스 트래픽을 분산했습니다.

3.	메세지가 SQS에 들어오면 순차적으로 처리하여 람다에서 쿠폰 발급 함수를 호출합니다. 

- 쿠폰 만료

1.	REDIS에서 쿠폰 데이터 읽어 SQS를 보냅니다. CloudFront 글로벌 기반 서비스로 인해 모든 자정을 파악하기위해 cron을 통해 1시간마다 주기적으로 업데이트합니다.

2.	이벤트 브릿지를 3개로 하여 각 미국, 일본 그리고 한국으로 설정했습니다. 올리브영이 미국, 일본 진출 계획에 있기 때문에 각 브릿지를 세 나라로 설정했습니다. 자정이 됐을 경우 cron을 통해 람다2를 호출합니다.

3.	SQS에서 저장된 메세지를 가져오고 Redis 데이터가 만료되면 영속성을 보장하기 위해 Aurora에 데이터를 백업합니다.




## 💻 시연 영상





