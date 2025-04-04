<img src="https://companieslogo.com/img/orig/MDB_BIG-ad812c6c.png?t=1648915248" width="50%" title="Github_Logo"/> <br>


# MongoDB Atlas Training Index 

### Create diverse indexes
MongoDB에서 제공하는 다양한 타입의 인덱스를 생성해 빠른 데이터 액세스 조회 및 각 기능을 확인합니다. mongosh로 클러스터 내 sample_mflix DB에 접속합니다.
```
mongosh "mongodb+srv://{host명}.mongodb.net/" --apiVersion 1 --username <db_username>
use sample_mflix
show collections
```

1. Primary index

Primary index는 _id 필드에 대해 기본적으로 생성되어 있는 인덱스입니다.

```
db.comments.getIndexes()
db.movies.getIndexes()
```
getIndexes() 커맨드를 활용해 각 collection에 생성된 인덱스를 확인합니다.
인덱스를 확인해보면 primary index가 생성되어 있는 것을 보실 수 있습니다.

2. Secondary index
Primary index가 아닌 사용자가 직접 만드는 index를 생성하고 쿼리 시간을 비교해보겠습니다. 
```
db.movies.explain().find({type: "series"})
```
explain output의 winning plan 정보에서, collection scan이 발생한 것을 확인합니다.

explain 결과를 확인할 때는 compass로 확인하는 것이 더 정보를 확인하기 쉽습니다.
같은 쿼리를 compass에서도 실행해서 explain 결과를 확인해보겠습니다.

[이미지 추가]

```
db.movies.createIndex({type: 1})
db.movies.getIndexes()
```
인덱스를 생성하고 조회해보면, type_1라는 이름의 인덱스가 생성되어 있습니다.

