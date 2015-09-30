# prediction.learn
prediction.learn

## install prediction.io

```
# set 
bash -c "$(curl -s https://install.prediction.io/install.sh)"

Welcome to PredictionIO 0.9.4!
Mac OS detected!
Using user: md
Where would you like to install PredictionIO?
Installation path (/Users/md/PredictionIO): /Users/md/workspace/prediction.io
Vendor path (/Users/md/workspace/prediction.io/vendors):
Please choose between the following sources (1, 2 or 3):
1) PostgreSQL
2) MySQL
3) Elasticsearch + HBase
#? 3

...

# add prediction.io/bin to $PATH

pio-start-all
```

## install recommendation template

```
cd $WORKSPACE/prediction.io
mkdir recommends
cd recommends

pio template get PredictionIO/template-scala-parallel-recommendation Recommendation

pio app new PHApp
2015-09-30 17:15:54.143 java[12453:154318] Unable to load realm info from SCDynamicStore
[INFO] [HBLEvents] The table pio_event:events_1 doesn't exist yet. Creating now...
[INFO] [App$] Initialized Event Store for this app ID: 1.
[INFO] [App$] Created new app:
[INFO] [App$]       Name: PHApp
[INFO] [App$]         ID: 1
[INFO] [App$] Access Key: NIKa6JRIj2Ga0YCf1TBKTRthT0PXe9QuahbfMWxl61pFxcbsvmp4RSb1pmx8l909

pio app list
[INFO] [App$]                 Name |   ID |                                                       Access Key | Allowed Event(s)
[INFO] [App$]                PHApp |    1 | NIKa6JRIj2Ga0YCf1TBKTRthT0PXe9QuahbfMWxl61pFxcbsvmp4RSb1pmx8l909 | (all)
[INFO] [App$] Finished listing 1 app(s).

```

## install python sdk
```
pip install predictionio
```

## import sample data
```
cd /Users/md/workspace/prediction.io/recommends/Recommendation
curl https://raw.githubusercontent.com/apache/spark/master/data/mllib/sample_movielens_data.txt --create-dirs -o data/sample_movielens_data.txt
python data/import_eventserver.py --access_key $PIO_ACCESS_KEY
```

## deploy as a service

- engine.json
```
{
  "id": "default",
  "description": "Default settings",
  "engineFactory": "public_health.RecommendationEngine",
  "datasource": {
    "params" : {
      "appName": "PHApp"
    }
  },
  "algorithms": [
    {
      "name": "als",
      "params": {
        "rank": 10,
        "numIterations": 20,
        "lambda": 0.01,
        "seed": 3
      }
    }
  ]
}
```

## building
```
# sbt 빌드 시간이 꽤 걸린다..를 넘어 심하다 ;;
pio build --verbose
...
[INFO] [Console$] Your engine is ready for training.
```

## Training the Predictive Model
```
pio train
...
[INFO] [CoreWorkflow$] Training completed successfully.
```

## Deploying the Engine
```
pio deploy
```

## deploy engine bind
- http://localhost:8000


## query results
```
curl -H "Content-Type: application/json" \
-d '{ "user": "1", "num": 4 }' http://localhost:8000/queries.json

{
    "itemScores": [
        {
            "item": "83",
            "score": 12.123393065423839
        },
        {
            "item": "46",
            "score": 10.08474603110993
        },
        {
            "item": "38",
            "score": 9.559770500200752
        },
        {
            "item": "71",
            "score": 8.11239036898116
        }
    ]
}
```

## servers

- Elasticsearch
  - http://localhost:9200

- eventserver
  - http://localhost:7070

- environment
```
# set access key to the shell variable
PIO_ACCESS_KEY=NIKa6JRIj2Ga0YCf1TBKTRthT0PXe9QuahbfMWxl61pFxcbsvmp4RSb1pmx8l909
```

## api
- example

```
curl -i -X POST http://localhost:7070/events.json?accessKey=$PIO_ACCESS_KEY \
-H "Content-Type: application/json" \
-d '{
  "event" : "rate",
  "entityType" : "user"
  "entityId" : "u0",
  "targetEntityType" : "item",
  "targetEntityId" : "i0",
  "properties" : {
    "rating" : 5
  }
  "eventTime" : "2014-11-02T09:39:45.618-08:00"
}'
```

- get 
```
curl -i -X GET "http://localhost:7070/events.json?accessKey=$PIO_ACCESS_KEY"

[
  {
    "eventId": "illrLcpg1dDE2bvZZ1NpggAAAUlxl11So2YuqaXhHrg",
    "event": "rate",
    "entityType": "user",
    "entityId": "u0",
    "targetEntityType": "item",
    "targetEntityId": "i0",
    "properties": {
    "rating": 5
  },
  "eventTime": "2014-11-02T09:39:45.618-08:00",
  "creationTime": "2015-09-30T08:32:27.601Z"
  }
]
```



