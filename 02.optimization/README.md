<img src="https://companieslogo.com/img/orig/MDB_BIG-ad812c6c.png?t=1648915248" width="50%" title="Github_Logo"/> <br>


# MongoDB Atlas Training Optimization

#### 1. Load Sample Data - sample_airbnb collection
생성한 클러스터의 메뉴에서 "Load Sample Dataset"을 선택한 후, sample_airbnb를 선택합니다.
<img width="928" alt="Screenshot 2025-04-20 at 8 54 25 PM" src="https://github.com/user-attachments/assets/b5675e42-40c9-4f91-b527-a5fa065650f0" />

<img width="608" alt="Screenshot 2025-04-20 at 8 54 53 PM" src="https://github.com/user-attachments/assets/71736c49-2db3-4e0e-8236-3999cadf81c4" />

현재 listingsAndReviews 콜렉션의 다큐먼트 개수를 확인합니다. (클러스터에 접속한 후 mongosh를 통해서 아래 내용을 수행합니다)  
```
> show dbs
> use sample_airbnb
> show collections
> db.listingsAndReviews.countDocuments({})
```
아래와 같이 aggregation을 이용해서 listingsAndReviews_big_collection 이라는 새로운 콜렉션을 생성하고 다큐먼트 개수를 확인합니다.
```
> db.listingsAndReviews.aggregate([ 
{ $addFields: { a: { $range:[0,10] }}},
{ $unwind: "$a" },
{ $project: { _id:0, a:0 } },
{ $out: "listingsAndReviews_big_collection" } 
  ])

> db.listingsAndReviews_big_collection.countDocuments({})
55550
```

#### 2. Run Slow Queries / Inefficient Queries for Analysis

효율적이지 않은 쿼리를 실행하기 위해 아래와 같이 실행합니다.
```
> for (x = 0; x < 100; x++) {
  db.listingsAndReviews_big_collection.findOne({
    property_type: "House"
  })
}
```
```
> for (x = 0; x < 100; x++) {
  db.listingsAndReviews_big_collection.find({ 
    minimum_nights: { $gt: '2' },
    maximum_nights: { $lte: '30' },
    bedrooms: { $gte: 2 }, 
    property_type: 'House', 
    room_type: 'Entire home/apt' 
    }).sort({number_of_reviews:-1})
}
```
```
> for (x = 0; x < 100; x++) {
  db.listingsAndReviews_big_collection.find({
    'address.market': 'Rio De Janeiro',
    price: { $lte: Decimal128('200.00') },
    bedrooms: { $gte: 2},
    minimum_nights: { $lte: '2' },
    amenities: 'Wifi'}).sort({price: -1})
}
```
#### 3. Atlas Real Time 
Atlas의 Real Time 패널로 이동합니다. 
아래 Slow Query를 실행한 후, Real Time Performance Panel에서 어떻게 나타나는지 확인합니다.
```
> for(x=0;x<20;x++) {
  db.listingsAndReviews_big_collection.findOne( {property_type : "Palace"})
}
```
아래 Query를 실행한 후, Real Time Performance Panel에서 어떻게 나타나는지 확인합니다.
```
> for(x=0;x<100000;x++) {
  db.listingsAndReviews_big_collection.find().limit(10).itcount()
}
```

#### 4. Atlas Alert 
아래와 같은 Alert를 새로 생성하고, alert가 정상적으로 동작하는지 확인합니다.
Project Settings > Alerts, [Add New Alert] 
Alert 조건: Query Targeting: Scanned Objects / Returned is... 값이 3초 이상 소요되는 query가 발생했을 때 Project Owner에게 Email 보내기
<img width="937" alt="Screenshot 2025-04-21 at 10 20 45 AM" src="https://github.com/user-attachments/assets/920a9a38-2e31-4ee0-b492-3dc0701a130a" />

아래 쿼리를 실행하여 해당 alert가 정상적으로 동작하는지 확인합니다.
```
> for (x = 0; x < 400; x++) {
  db.listingsAndReviews_big_collection.findOne({ property_type: "Palace" })
}
```


#### 5. Identify Slow/Inefficient Queries with Query Profiler
Query Profiler를 이용해 slow/inefficient query를 식별하고, 해당 query hash를 reject할 수 있도록 설정합니다.
(아래 내용에서 나오는 operation이 가장 긴 query나 queryShapeHash값은 아래에서 예를 든 값과 다를 수 있습니다)
##### 5.1 Query Profiler를 사용해 Operation Execution Time이 가장 높은 operation을 선택합니다.
<img width="1349" alt="Screenshot 2025-04-21 at 10 43 00 AM" src="https://github.com/user-attachments/assets/9746f6e6-6ca7-4928-8309-955f486548d3" />

##### 5.2 Query Details -> Query Log (View Full Parsed Log를 enable)에서 queryShapeHash값 확인
<img width="577" alt="Screenshot 2025-04-21 at 10 44 05 AM" src="https://github.com/user-attachments/assets/bc31fbd1-b885-4ce6-a2a8-5a53d2b8b0e5" />

<img width="578" alt="Screenshot 2025-04-21 at 10 56 40 AM" src="https://github.com/user-attachments/assets/56fedca0-21ea-48bf-a8cf-1974c3cd5729" />

##### 5.3 Operation Rejectio Filters를 사용하여 해당 query shape와 일치하는 query는 실행되지 않도록 명령어 실행
```
db.adminCommand({
   setQuerySettings: <5.2에서 확인한 queryShapeHash값>,
   settings: {
      reject: true
   }
})
```


#### 6. Performance Advisor
Performance Advisor를 이용해서 효율적인 쿼리 실행을 위해 Create Indexes / Drop Indexes 항목을 확인하여 해당 권고사항을 반영합니다.
(Performance Advisor에서 권고하는 항목은 아래 예시와 다를 수 있습니다)
<img width="1065" alt="Screenshot 2025-04-21 at 10 59 14 AM" src="https://github.com/user-attachments/assets/16842346-8368-478d-8087-ebf56554f4b5" />
<img width="1120" alt="Screenshot 2025-04-21 at 10 59 37 AM" src="https://github.com/user-attachments/assets/75e26b11-58a7-46c3-8dd9-b3bba87c5a25" />
<img width="780" alt="Screenshot 2025-04-21 at 10 59 54 AM" src="https://github.com/user-attachments/assets/6ac5dc51-199e-4c99-b030-ebd911bcf9fd" />




