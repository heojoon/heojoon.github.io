## ECS Quick HandsBook



1. Create IAM User

   ~~~
   - username : test.ecruser
   - 엑세스 유형 : 프로그래밍 방식 엑세스
   - 권한 : 기존 정책 직접 연결
    : (ECR 전체 접근 권한) AmazonEC2ContainerRegistryFullAccess
    : (ECS 컨트롤 권한) AmazonEC2ContainerServ 로 검색후 나오는 권한 모두 선택
   - 사용자 생성 완료 
   - 엑세스키 ID , 비밀 엑세스키 별도 메모(저장)
   ~~~

   

2. AWS CLI 환경 설정 

   - Windwos WCL 환경 사전 구성 필요. 

   ~~~
   1. 파워쉘 실행
   $ bash
   $ sudo apt-get update -y
   $ sudo apt-get install -y python3-virtualenv
   $ mkdir hello_app
   $ cd hello_app/
   $ virtualenv -p python3 venv
   $ source venv/bin/activate
   (venv) hjoon@DESKJOON-6347:~/hello_app$ pip install awscli --upgrade
   (venv) hjoon@DESKJOON-6347:~/hello_app$ aws configure
   AWS Access Key ID [****************ELH5]: AKIAX7HF56RBUVH3E5PC
   AWS Secret Access Key [****************HliD]: 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
   Default region name [ap-northeast-2]:
   Default output format [None]: json
   
   # ECR 에 Repository 생성 
   aws ecr create-repository --repository-name fargatedemo/helloapp
   
   
   # ECR 로그인
   aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin 548090541123.dkr.ecr.ap-northeast-2.amazonaws.com
   
   # 샘플 app 
   docker pull rrequero/flask-app
   
   
   
   ~~~

3. Docker Install

   ~~~bash
   $ sudo apt-get update
   
   $ sudo apt-get install \
       apt-transport-https \
       ca-certificates \
       curl \
       gnupg-agent \
       software-properties-common
       
   $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   
   $ sudo apt-key fingerprint 0EBFCD88
   ​~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
   pub   rsa4096 2017-02-22 [SCEA]
         9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
   uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
   sub   rsa4096 2017-02-22 [S]
   ​~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
   $ sudo add-apt-repository \
      "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) \
      stable"
   
   $ sudo apt-get update
   
   $ sudo apt-get install docker-ce docker-ce-cli containerd.io
   
   ## WSL 환경으로... (docker run이 안되네..-_-)
   $ sudo service docker start
   $ sudo docker pull brumbrum/python3:production-brumbrum
   
   
   ~~~

   

