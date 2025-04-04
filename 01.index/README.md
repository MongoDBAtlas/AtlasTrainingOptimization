<img src="https://companieslogo.com/img/orig/MDB_BIG-ad812c6c.png?t=1648915248" width="50%" title="Github_Logo"/> <br>


# MongoDB Atlas Training Index 

### Create diverse indexes
MongoDB에서 제공하는 다양한 타입의 인덱스를 생성해 빠른 데이터 액세스 조회 및 각 기능을 확인합니다. mongosh로 클러스터 내 sample_mflix DB에 접속합니다.
```
mongosh "mongodb+srv://{host명}.mongodb.net/" --apiVersion 1 --username <db_username>
use sample_mflix
show collections
```

#### 1. Primary index

Primary index는 _id 필드에 대해 기본적으로 생성되어 있는 인덱스입니다.

```
db.comments.getIndexes()
db.movies.getIndexes()
```
getIndexes() 커맨드를 활용해 각 collection에 생성된 인덱스를 확인합니다.
인덱스를 확인해보면 primary index가 생성되어 있는 것을 보실 수 있습니다.

#### 2. Secondary index

Primary index가 아닌 사용자가 직접 만드는 index를 생성하고 쿼리 시간을 비교해보겠습니다. 
```
db.movies.explain().find({type: "series"})
```
explain output의 winning plan 정보에서, collection scan이 발생한 것을 확인합니다.

explain 결과를 확인할 때는 compass로 확인하는 것이 더 정보를 확인하기 쉽습니다.
같은 쿼리를 compass에서도 실행해서 explain 결과를 확인해보겠습니다.

<img src="/01.index/images/image01.png" width="100%" height="100%">    
조건을 만족하는 document를 반환하기 위해 스캔한 document, 최종적으로 반환한 document 및 실행 시간 등을 쉽게 조회할 수 있습니다.

```
db.movies.createIndex({type: 1})
db.movies.getIndexes()
```
다시 mongoshell로 돌아가 인덱스를 생성합니다.
인덱스를 조회해보면 type_1라는 이름의 인덱스가 생성되어 있습니다.


```
db.movies.explain().find({type: "series"})
```
다시 explain 결과를 실행해보면 이번에는 index scan과 fetch 단계가 실행된 것을 확인할 수 있습니다.

<img src="/01.index/images/image02.png" width="100%" height="100%"> 

마찬가지로, compass에서도 결과를 확인해봅니다. 

#### 3. Multikey index
Array 값을 가진 필드에 대한 인덱스인 multikey index를 생성합니다.

```
db.movies.createIndex({genres: 1})
db.movies.getIndexes()
```
인덱스 생성 후 인덱스를 조회해봅니다.

```
db.movies.explain().find({genres: {$elemMatch: {$eq: 'Drama'}}})
db.movies.explain().find({genres: {$in: ["Comedy", "Biography"]}})
```
<img src="/01.index/images/image03.png" width="100%" height="100%"> 

Array 필드에 대해서 쿼리를 할 때도, index scan을 통해 더 빠르게 데이터를 조회할 수 있습니다.

#### 4. Geospatial index
이번에는 geospatial 샘플 데이터가 있는 sample_restaurants DB의 restaurants collection을 사용해보겠습니다.

인덱스 없이 데이터를 조회해보겠습니다.
full scan한 explain compass 화면 추가
<img src="/01.index/images/image04.png" width="100%" height="100%">

<img src="/01.index/images/image05.png" width="100%" height="100%">

```
use sample_restaurants
db.restaurants.find().limit(1)
```
데이터를 조회해보면 address 필드의 nested field인 coord 필드 내 geospatial 데이터가 포함되어 있습니다.

```
db.restaurants.createIndex({"address.coord": "2d"})
```
Nested field를 명시하려면 쌍따옴표로 필드명을 묶어줘야 합니다. 
필드명과 geospatial 데이터 타입을 명시해 인덱스를 생성합니다.

```
db.restaurants.find({
  "address.coord": {
    $near: [-73.856077, 40.848447]
  }
})
```
<img src="/01.index/images/image06.png" width="100%" height="100%"> 
$near operator로 근처에 있는 레스토랑을 쿼리합니다.
인덱스 스캔을 통해 빠르게 데이터 검색이 가능합니다.

```
db.restaurants.find({
  "address.coord": {
    $near: {
      $geometry: {
        type: "Point",
        coordinates: [-73.856077, 40.848447]
      },
      $maxDistance: 1000 // distance in meters
    }
  }
})
```
<img src="/01.index/images/image07.png" width="100%" height="100%"> 
특정 데이터 조건을 통해 
