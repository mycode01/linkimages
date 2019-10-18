## SSH TUNNELING tutorial

현석님이 일전에 이야기하신 ssh터널링을 유용하게 사용할방법을 몇개 알고있는데,    
간단한 튜토리얼을 통해 ssh터널링을 이용해 보겠습니다.

목표는 
#### 보안그룹 수정없이 집의 IP로 회사외에서 접근이 불가하게 보안그룹이 정의된 RDS에 연결하기 
입니다.
Mac os 기준으로 합니다.
준비물은 다음과 같습니다.
* RDS로의 tunnel을 맡아줄 EC2 인스턴스
* EC2 인스턴스에서 RDS로의 연결이 가능하도록 보안그룹을 수정할수있는 IAM 권한
* EC2로 연결이 가능한 key file(PEM)
* ssh명령이 가능한 shell(on local machine)    
아, 그리고 tunnel역을 맡아줄 EC2인스턴스는 편의상 어디서든 연결이 가능하도록 보안그룹 설정에 0.0.0.0/0 : 22를 사용해 줍니다.    

##### 이하의 결과는 보안적으로 헛점이 발생할 여지가 많으니, 키파일은 엄중히 관리하시고 되도록 보안그룹, 서브넷 설정도 신경써주시기 바랍니다.
------


개념적으로는 이렇게 연결이 됩니다.    
Localhost:3306 <>~~remote instance:22~~<> RDS:3306

ssh tunnel역을 맡아줄 aws instance생성 후 RDS로 연결될수있도록 보안그룹설정을 해줍니다.     
환경마다 보안그룹 모양은 다르니 자세한 설명은 넘어갑니다.    
EC2 인스턴스에서 RDS로 연결이 가능한지 테스트를 합니다. (Remote instance <> RDS:3306)    
```> nc -v RDS_Endpoint 3306 ```
로컬머신이 아니라 EC2 인스턴스에서 실행해줍니다.    

명령을 실행하여 깨진 문자라도 반응이 온다면 일단 연결이 된것이며, 타임아웃이 발생할경우 다시 보안그룹 설정을 만져줍니다.

remote에서 RDS로의 연결이 확인되었으니, 로컬머신에서 22번포트를 이용하여 remote(Ec2 인스턴스)로 연결후,     
localhost의 3306포트를 remote를 통해 RDS의 3306으로 연결만 시켜주면 됩니다.

로컬에서 다음명령을 실행합니다.    
![연결시도](https://raw.githubusercontent.com/mycode01/linkimages/master/sshtunneling/01.png "try sshtunneling")    
```> ssh -i "remote연결이가능한pemfile" ec2-user@ec2_publicDns -N -L 3306:RDS_endpoint:3306```    
다만 ssh연결을 위한 user id는 EC2인스턴스 생성시 선택한 AMI에 따라 다를수 있습니다.    
```ec2_publicDns``` 에는 EC2인스턴스로 연결될수있는 주소를 입력해주시면 됩니다.    
```3306:RDS_endpoint:3306``` 에는 RDS 엔드포인트 주소를 적어주면 되며,    
첫번째 3306은 로컬머신에서 연결될 포트번호, 뒤에 있는 3306에는 RDS가 listen중인 포트번호를 적어주시면 됩니다. 예를들어    
##### 나는 로컬머신에서 53306포트를 이용하여 RDS에 3306포트로 연결하고 싶어 > ```53306:RDS_endpoint:3306``` 으로 대치 시키시면 됩니다.     
반대로, 
##### 우리팀의 RDS는 보안상의 문제로 53306포트를 이용하고 있어, 나는 로컬에서 3306을 사용하여 연결하고 싶어 > ```3306:RDS_endpoint:53306``` 으로 대치시켜주시면 됩니다.

```-N``` 플래그로 인하여 쉘에서 입력이 되지 않습니다. 단순한 터널역을 하기위해 사용하는 플래그입니다. 인터렉션이 필요하다면 빼줍니다.
```-L``` 플래그는 listen포트를 연결해주는 역할을 합니다.

다른 쉘을 켜서 로컬에서,    
![ssh연결확인`](https://raw.githubusercontent.com/mycode01/linkimages/master/sshtunneling/02.png "sure it")    
```> nc localhost 3306 ```    
을 실행시켜 remote에서 rds로 nc명령을 실행시켰을때와 같은 반응이 돌아온다면 성공한것입니다.    
RDS로의 우회 연결을 하기위하여 RDS의 endpoint대신 localhost:3306을 이용합니다.    
집에서도 밀린 업무를 할수있게 되었습니다. 만세!
![워크벤치연결확인](https://raw.githubusercontent.com/mycode01/linkimages/master/sshtunneling/03.png "workbench")  


