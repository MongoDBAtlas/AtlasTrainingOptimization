<img src="https://companieslogo.com/img/orig/MDB_BIG-ad812c6c.png?t=1648915248" width="50%" title="Github_Logo"/> <br>


# MongoDB Atlas Training Optimization

### Pre Work


#### Atlas Account
MongoDB Atlas 의 무료 계정을 생성 합니다.     
Atlas는 관리형 데이터 베이스로 간편하게 데이터 베이스를 생성 하고 인터넷을 통한 엑세스로 편리하게 사용 할 수 있습니다.   

계정 생성을 위해 Atlas 사이트에 접속 합니다.   

https://www.mongodb.com/ko-kr/cloud/atlas/register

신용카드 입력 없이 계정을 생성 할 수 있습니다. 기존 계정을 가지고 있는 경우 2개의 Freetier 데이터베이스 클러스터까지 생성 가능 하며 Hands-on 과정도 Free-tier를 이용하게 됩니다.    

#### Database 생성
Atlas에 로그인 후 테스트용 데이터 베이스를 생성 합니다.   
생성을 위해 클러스터 생성 버튼을 클릭 하여 줍니다.   
<img src="/00.pre-work/images/images20.png" width="90%" height="90%">    

간략화된 배포 옵션을 선택 화면에서 배포 할 클러스터의 티어와 Cloud Provider 및 배포 지역을 선택 하여 줍니다. (추가 옵션을 주어 배포를 하기 위해서는 하단에 Go to Advanced Configuration 을 클릭 하여 볼 수 있습니다.)      
<img src="/00.pre-work/images/images21.png" width="90%" height="90%">    

Free tier 클러스터를 AWS 서울 리전에 배포 하도록 선택 합니다.    
Cluster Tier 는 M0 를 선택 하고 Cluster의 이름을 입력 하여 줍니다. 추가로 Quick setup에 Preload sample dataset을 선택 하여 테스트용 데이터를 사전에 로드 합니다.        
<img src="/00.pre-work/images/images22.png" width="90%" height="90%">     

배포가 진행 되며 시간은 10분 이내로 소요 됩니다.   
<img src="/00.pre-work/images/images23.png" width="90%" height="90%">     


#### Database Account 생성
Atlas 데이터베이스 클러스터를 접근하기 위한 계정 생성으로 Security 메뉴에 Database Access를 클릭 하여 계정을 생성 할 수 있습니다.    
Hands-on에서는 Id/password를 이용하는 방식의 데이터베이스 계정을 생성 합니다.   
<img src="/00.pre-work/images/images08.png" width="90%" height="90%">  
계정은 atlas-account로 하여 생성 합니다. Built-in Role 은 편의상 Read and Write to any database 를 선택합니다.


#### Network Access 생성
데이터 베이스 접근 테스트를 위해서 접근 하려는 컴퓨터의 IP 주소를 방화벽에 허용 해 주어야 합니다.    
Security의 Network Access메뉴를 선택 합니다.
<img src="/00.pre-work/images/images05.png" width="80%" height="80%">  
Add IP Address를 클릭하고 Add IP Access List Entry 에서 Add current IP Address를 클릭하하고 Confirm을 선택 합니다.   
방화벽 설정은 1분 가량의 시간이 소요 됩니다.


#### 기타 필요한 소프트웨어
MongoDB에 접속하고 데이터를 조회 하는 GUI Tool (Compass)를 다운로드 합니다.

Compass :   
https://www.mongodb.com/products/compass

Mongosh :
https://www.mongodb.com/docs/mongodb-shell/install/

