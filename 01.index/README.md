<img src="https://companieslogo.com/img/orig/MDB_BIG-ad812c6c.png?t=1648915248" width="50%" title="Github_Logo"/> <br>


# MongoDB Atlas Training Optimization

### Create diverse indexes
MongoDB에서 제공하는 다양한 타입의 인덱스를 생성해 빠른 데이터 액세스 조회 및 각 기능을 확인합니다. 
mongosh로 클러스터 내 sample_mflix DB에 접속합니다.
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

<img src="/01.index/images/image05.png" width="100%" height="100%">    
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

<img src="/01.index/images/image06.png" width="100%" height="100%"> 

마찬가지로, compass에서도 결과를 확인해봅니다. 

#### 3. Multikey index
Array 값을 가진 필드에 대한 인덱스인 multikey index를 생성합니다.

```
db.movies.findOne()
db.movies.createIndex({genres: 1})
db.movies.getIndexes()
```
인덱스 생성 후 인덱스를 조회해봅니다.

```
db.movies.explain().find({genres: {$elemMatch: {$eq: 'Drama'}}})
db.movies.explain().find({genres: {$in: ["Comedy", "Biography"]}})
```


Array 필드에 대해서 쿼리를 할 때도, index scan을 통해 더 빠르게 데이터를 조회할 수 있습니다.

#### 4. Geospatial index
이번에는 geospatial 샘플 데이터가 있는 sample_restaurants DB의 restaurants collection을 사용해보겠습니다.



```
use sample_restaurants
db.restaurants.findOne()
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
db.restaurants.explain().find({
  "address.coord": {
    $near: [-73.856077, 40.848447]
  }
})
```
$near operator로 근처에 있는 레스토랑을 쿼리합니다.
인덱스 스캔을 통해 빠르게 데이터 검색이 가능합니다.


#### 5. TTL index
TTL 인덱스를 활용해 일정 시간이 지나면 document가 만료되게 설정할 수 있습니다.
TTL 인덱스를 통해 편리하게 스토리지 사이즈를 효율적으로 관리할 수 있습니다. 

TTL 인덱스 테스트를 위한 샘플 데이터를 insert합니다.
하루치 로그인 로그를 저장하는 document입니다.
```
use test
db.login_events.insertMany([
{user_id: 1, login_timestamp: ISODate("2025-04-21T09:10:30Z"), location: "KR"},
{user_id: 2, login_timestamp: ISODate("2025-04-21T12:10:30Z"), location: "US"},
{user_id: 3, login_timestamp: ISODate("2025-04-21T13:10:30Z"), location: "KR"},
{user_id: 4, login_timestamp: ISODate("2025-04-22T10:10:30Z"), location: "CA"},
{user_id: 5, login_timestamp: ISODate("2025-04-22T11:30:30Z"), location: "KR"},
])
db.login_events.find()
```
24시간이 지난 로그인 로그 데이터는 삭제되도록 TTL index를 생성합니다.

```
db.login_events.createIndex(
{"login_timestamp": 1},
{expireAfterSeconds: 86400}
)
db.login_events.getIndexes()
```

인덱스 생성 후 인덱스를 조회해봅니다.
TTL 스레드는 1분에 한번씩 만료시킬 데이터가 있는지 스캔을 진행합니다. 

db.login_events.find()
약간의 시간을 두고 다시 데이터를 조회해봅니다. 
데이터를 조회해보면 24시간 전 데이터는 삭제된 것을 확인하실 수 있습니다.

#### 6. Unique index 
Unique index를 활용해 특정 필드에 대해 고유한 값을 보장할 수 있습니다. 
샘플 데이터를 insert해보겠습니다.

```
use test
db.users.insertMany([
{user_id: 1, gender: "Female", age: 40, location: "IN"},
{user_id: 2, gender: "Male", age: 21, location: "US"},
{user_id: 3, gender: "Male", age: 39, location: "KR"},
{user_id: 4, gender: "Female", age: 55, location: "CA"},
{user_id: 5, gender: "Female", age: 34, location: "KR"},
])
db.users.find()
```
유저들에 대한 데이터가 insert된 것을 확인합니다.
이제 해당 유저에 대한 고유 아이디인 user_id가 중복되지 않고 user_id에 대한 빠른 쿼리가 가능하도록 인덱스를 생성하겠습니다.

````
db.users.createIndex({user_id: 1}, {unique: true})
db.users.getIndexes()
````
인덱스를 생성 후 인덱스를 조회해봅니다.

```
db.users.insertOne({user_id: 1, gender: "Female", age: 40, location: "IN"})
db.users.find()
```
Unique index가 있는 상태에서 user_id가 1인 document를 insert하니 duplicate key 에러가 발생하며 데이터가 써지지 않습니다.
이를 통해 인덱싱한 필드 값에 대해서는 고유한 값만 insert가 되는 것을 확인했습니다. 


#### 7. Partial index
Partial index를 활용하면 특정 조건을 만족하는 document에 대해서만 인덱스를 생성합니다.
데이터 양이 많고 모든 데이터를 빈번하게 쿼리하는 게 아니라면 해당 인덱스를 활용하셔서 인덱스 사이즈를 줄이실 수 있습니다.

sample_mflix.comments 데이터를 활용해보겠습니다.
2017년 전에 작성된 영화 리뷰 데이터는 빈번하게 쿼리하지 않아 해당 데이터들에 대해서는 인덱스가 필요하지 않고, 리뷰를 최신순으로 조회하기 위해 date 기준으로 쿼리를 자주 실행한다고 가정합니다.
이에 date 필드에 대해 인덱스를 생성하고 partial index의 조건도 date 기준으로 설정합니다.
```
use sample_mflix
db.comments.createIndex( 
{ date: 1 },             
{ partialFilterExpression: {date: {$gt: ISODate('2017-01-01T00:00:00.000+00:00')}} } )
```

```
db.comments.getIndexes()
```
인덱스 생성 후 인덱스를 조회해보면 date_1이라는 인덱스가 생성되었고 해당 인덱스에 대한 조건도 확인할 수 있습니다.

```
db.comments.explain().find({
  date: {$lt: ISODate('2017-01-01T00:00:00.000+00:00')}
})
```
2017 전에 생성된 리뷰를 찾는 쿼리를 실행하니 collection scan이 발생합니다.

```
db.comments.explain().find({
  date: {$gt: ISODate('2017-01-01T00:00:00.000+00:00')}
})
```
2017 후에 생성된 리뷰를 찾는 쿼리를 실행하니 index scan을 진행합니다.

### Index tip
#### 1. ESR rule
MongoDB에서는 복합 필드 인덱스가 가장 많이 사용됩니다.
복합 필드 인덱스를 설정할 때는, 쿼리 패턴에 맞게 인덱스 필드 순서를 지정하는 것이 중요합니다.


sample_mflix.movies 에서 2000년 이후에 개봉된 영화 중 "Bill Murray"가 출연한 영화 리스트를 검색 하고 제목 순서로 출력 합니다.   

Compass에서 movies 컬렉션을 선택 하고 Explain Plan 에서 실행 합니다.   
````
db.movies.find(
	{
    	"cast":"Bill Murray",
    	"year":{$gte:2000}
	}
).sort(
	{"title":1}
)
````
<img src="/01.index/images/image01.png" width="100%" height="100%">     

No index available for this query 로 인덱스가 사용 되지 않은 것을 확인 할 수 있으며 Dcouments Examined의 갯수가 23530으로 전체 문서가 스캔 된 것을 확인 할 수 있습니다.    
또한 Documnets Returned 가 12인 것으로 전체 문서 중 12개 문서가 리턴된 것으로 12개 문서를 찾기 위해 23530 문서를 검색한 것으로 비효율적인 것을 알 수 있습니다.

쿼리 성능 개선을 위해서 인덱스를 생성해보겠습니다.

테스트를 위해 cast - year - title 순서로 인덱스를 생성 하고 테스트 합니다. 
해당 인덱스는 exact match query - range query - sort query 순서입니다.

<img src="/01.index/images/image02.png" width="50%" height="50%">     


동일한 쿼리를 수행 하여 봅니다.    

<img src="/01.index/images/image03.png" width="90%" height="90%">     

문서 스캔이 Index 스캔으로 변경 되고 기존에 비해 성능이 개선된 것을 확인 합니다.  

첫 번째에서 IXSCAN으로 생성한 인덱스를 이용하여 12개의 문서가 검색된 것을 확인 할 수 있습니다. 이후 정렬 과정을 거친 후 데이터가 반환 되는 것을 확인 할 수 있습니다. 

인덱스를 ESR 순서로 작성합니다. (cast-title-year)   
동일한 쿼리를 실행 하여 플랜을 확인 합니다.    

<img src="/01.index/images/image04.png" width="90%" height="90%">     

Projection 항목에 title만을 출력 하도록 하고 Plan을 확인 합니다.
이번에는 IXSCAN과 더불어 정렬 단계가 없어져, 더 효율적인 쿼리 실행이 가능한 것을 확인합니다.
