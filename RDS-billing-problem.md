# [AWS] Free-tier로 RDS 사용 중 요금을 지불했어요!

거금 내고 다시 짚어보는 RDS 설정



## 문제 발생

백수생활에 이보다 더 치명적인 실수는 없다. 예상치 못한 지출이 발생했다.

![bill](https://github.com/arara90/images/blob/master/aws/billing-card.png?raw=true)



나는..Free-tier로 분명 다 잘 만든 것 같은데..?? 라고 멍청하게 착각하고 있었다.

AWS에 들어가서 세부 내역을 학인해보자. ( name - My billing Dashboard - Billing - Bills )

![bill](https://github.com/arara90/images/blob/master/aws/billing-detail.png?raw=true)

Relational Database Service(RDS)만 사용했으므로, 당연히 관련 부분에서만 요금이 책정 되었다.

* Amazon Relational Database Service Backup Storage

* Amazon Relational Database Service for PostgreSQL

* Amazon Relational Database Service Provisioned Storage

부분에서 과금이 발생했다. 





## 해결 과정

detail 을 보면서 하나씩 해결해보자.



### Amazon Relational Database Service Backup Storage

- 백업 저장소 사용료 -  *Backup storage* is the storage that is associated with **automated database backups** and any active **database snapshots** that you have taken. 
- DB생성 시에 (혹은 수정시에)  자동 백업의 유지기간을 0으로 설정한다.(비활성화)

![billing-detail-sol1.png](https://github.com/arara90/images/blob/master/aws/billing-detail-sol1.png?raw=true)



### Amazon Relational Database Service for PostgreSQL

* PostgreSQL 사용료 - 아래 이미지에서DB instance class 옵션에 따라 과금되는 부분이다.

  * $0.00 per RDS **db.t2.micro instance** hour (or partial hour) running PostgreSQL under monthly free tier / 750.000 Hrs / $0.00

  * $0.018 per RDS db.t2.micro instance hour (or partial hour) running PostgreSQL / 1,329.742 Hrs / $23.94

![billing-detail-sol2.png](https://github.com/arara90/images/blob/master/aws/billing-detail-sol2.png?raw=true)



### **Amazon Relational Database Service Provisioned Storage**

* 위 이미지에서 Storage type(스토리지유형) 옵션에따라 요금 결정. 

  처음에는 provisioned라는 단어때문에 Storage type 중에서 provisioned IOPS(SSD)를 선택해야 과금되는 것으로 착각했다. 하지만 이는 선택한  storage에 대한 과금. GP냐 provisioned IOPS냐 Magnetic이냐에따라 요금의 종류만 달라질 뿐! 

  ![billing-detail-sol3.png](https://github.com/arara90/images/blob/master/aws/billing-detail-sol3.png?raw=true)

  https://aws.amazon.com/rds/postgresql/pricing/?nc1=h_ls

  

* 그동안 storage의 개념을 instance에 묶어서 생각하고 있었던 것.  하드웨어로 생각해보면 너무나 명쾌한 것을.. 

* 그리고.. 'Provision' 단어의 쓰임  - [프로비저닝(Provisioning) 이란?](https://jins-dev.tistory.com/entry/프로비저닝Provisioning-이란)





## 결론

사실 처음부터 답은 예상했다. 총 3개의  DB(연습용 / 개발중인 앱용 / 개발중인 앱 dev용 )를 사용하고 있던 것. 

대표적으로, 두번째 RDS에 대한 과금 내역을 보면 750시간을 제외한 나머지 시간에 대해 과금된 것을 알 수 있고, 따져보면 3개에 대한 요금이란 것을 추측할 수 있다. 

당연히 활발히 사용중인 개발중인 앱 dev용을 제외하고 나머지 두 개를 후다닥 지웠다.



다행인 점은 세개의 인스턴스 모두 Single-Az, GP로 설정해 두어 과한 요금은 방지할 수 있었다. 한가지 더 알게 된 점은 Backup Storage에 대한 요금인데, 이 또한 큰 요금이 나온 부분이 아니어서 큰 화는 면했다.



마지막으로 나같은 초심자를 위해 간단히 정리하자면, 

### RDS - Free-tier를 사용할 때, 추가 과금을 막기 위해 설정해야 하는 것!

1. **Multi-AZ**(Multi-Availability Zone, 멀티 가용성 존) **deployment** -> **NO** 

   > 특히, Multi-AZ deployment 는 dafault가 Yes로 되어있기때문에 주의해야한다. 

2.  **Storage type(스토리지 타입)** -> **General Purpose(SSD)**

3. **Backup retention period(백업 보존 기간) -> 0days**

4. 그리고... free-tier는 막 퍼주지 않는다. 필요 없는 리소스는 잊지말고 바로바로 삭제하자..ㅋㅋ 





참고 :

https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/User_DBInstanceBilling.html

https://aws.amazon.com/ko/rds/postgresql/pricing/