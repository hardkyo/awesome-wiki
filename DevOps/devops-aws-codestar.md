---
title: AWS CodeStar로 1인 DevOps 코스프레하기
date: 2017-09-01 11:49:55
categories: devops
---

오랜전 일이지만 IDE에서 빌드한 jar, war 패키지를 직접 수동으로 배포하는 일은 개발에만 집중하고 싶은 이들에게는 괴로운 일이였죠.

여전히 IDE에서 빌드한 결과물을 수동으로 배포하는 조직도 존재하지만 반면 더욱 빠른 배포를 위해서 개발(Dev)과 운영(Ops)을 함께 진행하는 `DevOps` 문화를 정착시키기 위한 노력을 하는 조직도 동시에 존재합니다. 이와 같이 프로젝트를 빌드, 테스트하고 배포하는 과정을 자동화하기 위한 노력은 끊임없이 계속 되어 왔습니다.

지속적으로 일하는 방식을 개선하는 노력은 조직의 역량에 매우 긍정적이 영향을 미친다고 생각하는데, 이 글은 AWS CodeStar를 통해 적은 노력으로 이러한 문제를 해결 할 수 있기를 바라면서 시작합니다.

## 남들보다 빠르게 누구보다 빠르게

빌드, 배포하는 과정을 자동화하는 것만으로도 우리는 퇴사하고 싶은 유혹에서 많이 벗어 날 수 있었습니다. 하지만 빌드, 배포를 자동화 만으로는 모든 것을 해결할 수는 없었죠.

<img src='http://dev2ops.org/wp-content/uploads/2010/02/WallOfConfusion.png' />

```
개발자 - "아주 긴급한 내역입니다! 무려 Hotfixed 라구요 배포 부탁드립니다"
인프라 운영자 - "배포 요청이 너무 많은데요! 기다려주세요"
```

```
개발자 - "죄송합니다 또 수정 내역입니다! Hotfixed-2 라구요.. 배포 부탁드려요..ㅜㅜ"
인프라 운영자 - "흠 방금 배포했는데..?"
```

<img src='http://dev2ops.org/wp-content/uploads/2010/02/WallOfConfusion_Release.png' />

#### `"무언가 불편하다.."`

서비스를 운영하는 방식은 조직마다 차이가 있겠지만 개발자라면 누구나 한번쯤은 겪어 볼만한 일이죠, 하지만 이런 케이스는 아래와 같이 대부분 중요한 이슈일 경우가 많습니다. 

> 변화가 빠른 스타트업 조직에서의 잦은 업데이트
> 배포 후 장애 발생 시에 긴급한 재 배포

<img src='http://dev2ops.org/wp-content/uploads/2010/02/WallOfConfusion_TrainWreck.png' />

> 이미지 출처 - http://dev2ops.org/2010/02/what-is-devops/

#### `"장애 상황을 대처하는 과정을 통해 좋은 팀을 구분할 수 있다"`

개발자나 인프라 담당자나 서비스에 중요한 영향을 미치는 이 시기에 불필요한 행동이 나오면 안되겠죠, 특히나 장애를 대처하기 위해서는 롤백을 통한 빠른 배포가 아주 중요합니다. 최종적으로 배포하기위한 `버튼`에 대한 권한이 누구에게 있느냐에 따라 상황은 달라질 수 있습니다. 

개인적으로는 인프라 운영자가 배포하는 것보다는 직접 서비스를 개발하는 개발자가 능동적으로 배포 할 수 있는 조직이 장애 대처에 효율적이였던 경험이 많습니다.

하지만 그럼에도 불구하고 개발자가 변경내역을 Git Repository에 Push 한 뒤에 배포까지 필요한 과정은 그리 간단하지는 않아보이네요.

## 우리에게 필요한건 대략..

<img src='https://d0.awsstatic.com/feature-illustrations/Feature_CodeStar_Develop-in-minutes.png' />

- 우리는 소중한 소스코드의 관리를 위해 Git Repository를 준비해야 합니다
- Jenkins 서버를 따로 두어서 Git master 브랜치에 변경내역이 생기면 쿨하게 자동으로 테스트와 빌드를 진행 합니다
- 배포 준비가 완료되면 서비스를 담당하는 팀원, 인프라 운영자, 서비스 운영자에게 배포시기, 배포내용을 Notification 하는 것도 중요합니다
- 변경 내역을 공지하고 배포를 시작합니다! 하지만 운영환경에서는 떨리시죠? 빌드 과정에서 발생하는 로그도 자세히 살펴볼 필요가 있겠네요
- 성공적으로 빌드된 결과물은 항상 백업해두고 원격 서버에 전송해야 합니다
- 서버 애플리케이션이라면 여러 대의 서버를 동시에 운영하는 경우가 많겠죠
- 클라이언트 애플리케이션이라면 요구사항에 따라 패키징도 해야 합니다 
- 최종적으로 변경 내역이 정상적으로 작동하는지 모니터링도 필수!

굴직한 부분만 나열해 보았지만 사이 사이 필요한 요소들이 더 있을 거예요. Git, Jenkins, 운영 서버등 물리적으로 관리해야 하는 공간들이 많아지고 거기다 한 곳에서 모니터링 할 수 있는 구조도 아니죠. 

#### `"적은 인원의 개발자 혹은 개인이 개발과 운영을 동시에 하는 일은 쉽지 않아 보입니다"`

하지만 다행인 사실은 현대의 운영 환경은 더이상 IDC실에 달려가 직접 인프라 환경을 구성할 필요가 없다는 것입니다. Microsoft, AWS, IBM, Google과 같은 글로벌 기업에서는 클라우드 환경을 통해 인프라를 서비스 형태로 제공하고 뿐만 아니라 인프라 구성을 자동화할 수 있도록 다양한 소프트웨어와 API를 제공하고 있습니다. 

#### `"우리가 이렇게 축복받은 세상에 살고 있습니다"`

이 글에서는 지금까지 구구절절(..) 설명한 인프라 운영에 대한 문제를 AWS의 CodeStar라는 서비스를 중심으로 설명합니다.

## AWS CodeStar

개발 초기부터 `서버 구성, 빌드, 배포`하는 과정에서 반복적으로 발생하는 스트레스에서 벗어나고 싶었습니다. 배포 환경은 AWS의 EC2, 빌드 서버는 Jenkins가 중심인 전통적인 프로세스를 준비하다가 위와 같이 배보다 배꼽이 커지는 상황에 고민에 빠지게 되었는데, 때마침 최근에 AWS에서 CodeStar라는 프로젝트의 서버 구성, 빌드, 배포, 모니터링을 위한 통합 서비스를 발표해서 개이득!을 외치며 살펴 볼 수 있었습니다.

CodeStar는 서비스 운영에 필요한 컨테이너 생성부터 소스코드 관리, 빌드, 배포, 모니터링을 위한 다양한 서비스를 한 곳에서 쉽게 관리 할 수 있는 서비스라고 보면 됩니다. 우리가 위에서 고민했던 많은 문제들을 해결 할 수 있겠네요.

<img src='http://image.toast.com/aaaaahq/aws-code-star.png' />

#### 이런 분들이 사용하면 좋다고 하네요

- 배포 환경의 서버 인프라 구성을 자동화하고자 하는 분
- 아직도 IDE에서 빌드한 Jar 파일을 이용하시는 분
- 빌드 결과물을 생성하기 위해 Jenkins 서버를 따로 운영하는게 번거로우신 분
- 빌드를 위한 운영 비용을 획기적으로 줄이고 싶으신 분들
- Travis CI와 같은 호스트 CI서버로 이동하고 싶은데 조직의 승인이 까다로운 분
- 조직에서 외부 서비스를 통해 빌드 프로세스를 진행 하는 것을 꺼린다면, AWS 안에서 빌드하는 것은 어떠세요?

#### 이슈 관리도 가능

뿐만 아니라 CodeStar에서는 프로젝트에 관련된 팀원을 관리하고 이슈 트래킹을 위해 JIRA와 연동하는 기능까지 제공함으로써 개발 프로세스 전반에 영향을 미치고 싶어하는 모습을 강력히 나타내고 있는데요, 아래의 그림을 보시면 쉽게 이해가 가능합니다. 그럼 CodeStar를 통해서 프로젝트를 생성하고 배포하는 과정까지 필요한 요소들을 살펴 볼까요?

<img src='https://media.amazonwebservices.com/blog/2017/codestar_adh_proj_workflow_1.png' />

## 프로젝트를 생성

AWS CodeStar는 효율적으로 DevOps를 실현하기 위해 다양한 AWS의 서비스를 묶어 이들의 대시보드를 제공하는 형태입니다. CodeStar에서 프로젝트를 생성하는 의미는 CodeStar의 하위 서비스를 통해 프로젝트에 필요한 Git Repository와 서버 인프라를 만드는 것과 같습니다.

#### EC2 or Elastic Beanstalk

CodeStar에서는 EC2 뿐만 아니라 Elastic Beanstalk를 통해 `Java, .NET, PHP, Node.js, Python, Ruby, Go, Docker`를 사용하여 Apache, Nginx, Passenger, IIS와 같은 친숙한 서버에서 개발된 웹 애플리케이션 및 서비스를 간편하게 배포하고 확장할 수 있는 서비스를 제공합니다.

코드를 업로드하기만 하면 Elastic Beanstalk가 용량 프로비저닝, 로드 밸런싱, 자동 크기 조정부터 시작하여 애플리케이션 상태 모니터링에 이르기까지 배포를 자동으로 처리합니다. 이뿐만 아니라 애플리케이션을 실행하는 데 필요한 AWS 리소스를 완벽하게 제어할 수 있으며 언제든지 기본 리소스에 액세스할 수 있습니다.

#### CodeCommit

AWS CodeCommit은 안전하고 확장성이 뛰어난 프라이빗 Git 리포지토리를 쉽게 호스팅할 수 있도록 지원하는 완전 관리형 소스 제어 서비스입니다. CodeCommit를 사용하면 자체 소스 제어 시스템을 운영하거나 인프라 조정을 염려할 필요가 없습니다. CodeCommit을 사용하면 소스 코드에서 바이너리까지 모든 것을 안전하게 저장할 수 있고 기존 Git 도구와 원활하게 연동됩니다.

## 빌드

#### CodeBuild

CodeBuild는 소스 코드를 컴파일하는 단계부터 테스트 실행 후 소프트웨어 패키지를 개발하여 배포하는 단계까지 마칠 수 있는 완전관리형 빌드 서비스입니다. CodeBuild를 사용하면 자체 빌드 서버를 프로비저닝, 관리 및 확장할 필요가 없습니다. CodeBuild는 지속적으로 확장되며 여러 빌드를 동시에 처리하기 때문에 빌드가 대기열에서 대기하지 않습니다. 사전 패키징된 빌드 환경을 사용하면 신속하게 시작할 수 있으며 혹은 자체 빌드 도구를 사용하는 사용자 지정 빌드 환경을 만들 수 있습니다. CodeBuild에서는 컴퓨팅 리소스를 사용하는 시간(분) 단위로 비용이 청구됩니다.

<img src='https://s3.amazonaws.com/apnblog/2016+Blog+Images/Stelligent+Guest+Post/Updates/rsz_figure_3_.png' width='500' />

## 배포

#### CodePipeline

Source 저장소에 변경내역을 감지하는 것을 시작으로 배포하기 까지의 일련의 흐름을 CodePipeline을 통해 관리합니다.

CodePipeline은 빠르고 안정적인 애플리케이션 및 인프라 업데이트를 위한 지속적 통합 및 지속적 전달 서비스입니다. CodePipeline은 사용자가 정의하는 출시 프로세스 모델에 따라 코드 변경이 있을 때마다 코드를 구축, 테스트 및 배포합니다. 따라서 기능과 업데이트를 신속하고 안정적으로 제공할 수 있습니다.

<img src='https://s3.amazonaws.com/apnblog/2016+Blog+Images/Stelligent+Guest+Post/Updates/figure_1' />

## 모니터링

CodeStar의 콘솔에서는 모니터링을 위한 UI를 제공합니다. 

#### CloudWatch

CloudWatch를 사용하여 지표를 수집 및 추적하고, 로그 파일을 수집 및 모니터링하며, 경보를 설정하고, AWS 리소스 변경에 자동으로 대응할 수 있습니다. Amazon EC2 인스턴스, Amazon DynamoDB 테이블, Amazon RDS DB 인스턴스 같은 AWS 리소스뿐만 아니라 애플리케이션과 서비스에서 생성된 사용자 정의 지표 및 애플리케이션에서 생성된 모든 로그 파일을 모니터링할 수 있습니다. Amazon CloudWatch를 사용하여 시스템 전반의 리소스 사용률, 애플리케이션 성능, 운영 상태를 파악할 수 있습니다. 이러한 통찰력을 사용하여 문제에 적절히 대응하고 애플리케이션 실행을 원활하게 유지할 수 있습니다.

## 마치며

조직의 규모를 떠나서 애플리케이션의 변경 내역을 빠르고 안정적으로 배포하는 것은 점점 중요해지고 있습니다. 하지만 이를 위해 시스템을 준비하고 지속적으로 개선하는 것은 쉬운일이 아닙니다. 

AWS의 CodeStar는 기존의 효율적인 시스템을 묶어 소스코드 관리부터 배포, 모니터링까지의 파이프라인을 손쉽게 구축하도록 제공하고 있습니다. 1인 개발자나 작은 조직에서 DevOps에 투자하는 비용이 부담이 된다면 이 글을 통해 조금이라도 도움이 되기를 바랍니다.

## References 

- https://aws.amazon.com/ko/blogs/apn/an-introduction-to-aws-codebuild/
- https://aws.amazon.com/ko/codestar/
- https://aws.amazon.com/ko/blogs/aws/new-aws-codestar/
- https://www.slideshare.net/awskr/ansible-cloudformation
- https://www.slideshare.net/awskorea/aws-code-star-devops