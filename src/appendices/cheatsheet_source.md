# Stages Cheatsheet Source

To test all the aggregation stage examples shown in the [Cheatsheet](cheatsheet.html), run the following JavaScript from the MongoDB Shell connected to a MongoDB database.

## Collections Configuration & Data Population

```javascript
// DB configuration
use cheatsheet;
db.dropDatabase();
db.places.createIndex({loc: "2dsphere"});

// 'shapes' collection
db.shapes.insertMany([
  {_id: "◐", x: "■", y: "▲", val: 11},
  {_id: "◑", x: "■", y: "■", val: 74},
  {_id: "◒", x: "●", y: "■", val: 79},
  {_id: "◓", x: "▲", y: "▲", val: 81},
  {_id: "◔", x: "■", y: "▲", val: 83},
  {_id: "◕", x: "●", y: "■", val: 85},
]);

// 'lists' collection
db.lists.insertMany([
  {_id: "▤", a: "●", b: ["◰", "◱"]},
  {_id: "▥", a: "▲", b: ["◲"]},
  {_id: "▦", a: "▲", b: ["◰", "◳", "◱"]},
  {_id: "▧", a: "●", b: ["◰"]},
  {_id: "▨", a: "■", b: ["◳", "◱"]},
]);

// 'places' collection
db.places.insertMany([
  {_id: "◧", loc: {type: "Point", coordinates: [1,1]}},
  {_id: "◨", loc: {type: "Point", coordinates: [3,3]}},
  {_id: "◩", loc: {type: "Point", coordinates: [5,5]}},
  {_id: "◪", loc: {type: "LineString", coordinates: [[7,7],[8,8]]}},
]);
```


## Aggregation Stage Examples

> _Some of these stages apply to later versions of MongoDB only (e.g. 4.4, 5.0). Each stage is marked with the minimum version of MongoDB (shown in brackets). Therefore if you are running an earlier version, first comment out the appropriate "JavaScript stage sections" before executing the code._

```javascript
// $addFields  (v3.4)
db.shapes.aggregate([
  {"$addFields": {"z": "●"}}
]);


// $bucket  (v3.4)
db.shapes.aggregate([
  {"$bucket": {
    "groupBy": "$val", "boundaries": [0, 25, 50, 75, 100], "default": "Other"
  }}
]);


// $bucketAuto  (v3.4)
db.shapes.aggregate([
  {"$bucketAuto": {"groupBy": "$val", "buckets": 3}}
]);


// $count  (v3.4)
db.shapes.aggregate([
  {"$count": "amount"}
]);


// $facet  (v3.4)
db.shapes.aggregate([
  {"$facet": {
    "X_CIRCLE_FACET": [{"$match": {"x": "●"}}],
    "FIRST_TWO_FACET" : [{"$limit": 2}],
  }}
]);


// $geoNear  (v2.2)
db.places.aggregate([
  {"$geoNear": {
    "near": {"type": "Point", "coordinates": [9,9]}, "distanceField": "distance"
  }}
]);


// $graphLookup  (v3.4)
db.shapes.aggregate([
  {"$graphLookup": {
    "from": "shapes",
    "startWith": "$x",
    "connectFromField": "x",
    "connectToField": "y",
    "depthField": "depth",
    "as": "connections",
  }},
  {$project: {"connections_count": {"$size": "$connections"}}}
]);


// $group  (v2.2)
db.shapes.aggregate([
  {"$group": {"_id": "$x", "ylist": {"$push": "$y"}}}
]);


// $limit  (v2.2)
db.shapes.aggregate([
  {"$limit": 2}
]);


// $lookup  (v3.2)
db.shapes.aggregate([
  {"$lookup": {
    "from": "lists",
    "localField": "y",
    "foreignField": "a",
    "as": "refs",
  }}
]);


// $match  (v2.2)
db.shapes.aggregate([
  {"$match": {"y": "▲"}  
}]);


// $merge  (v4.2)
db.results.drop();
db.shapes.aggregate([
  {"$merge": {"into": "results"}}
]);
db.results.find();


// $out  (v2.6)
db.results.drop();
db.shapes.aggregate([
  {"$out": "results"}
]);
db.results.find();


// $project  (v2.2)
db.shapes.aggregate([
  {"$project": {"x": 1}}
]);


// $redact  (v2.6)
db.places.aggregate([
  {"$redact": {"$cond": {
    "if"  : {"$eq": ["$type", "LineString"]},
    "then": "$$PRUNE",
    "else": "$$DESCEND"
  }}}
]);


// $replaceRoot  (v3.4)
db.lists.aggregate([
  {"$replaceRoot": {"newRoot": {"first": {"$first": "$b"}, "last": {"$last": "$b"}}}}
]);


// $replaceWith  (v4.2)
db.lists.aggregate([
  {"$replaceWith": {"first": {"$first": "$b"}, "last": {"$last": "$b"}}}
]);


// $sample  (v3.2)
db.shapes.aggregate([
  {"$sample": {"size": 2}}
]);


// $set  (v4.2)
db.shapes.aggregate([
  {"$set": {"y": "▲"}}
]);


// $setWindowFields  (v5.0)
db.shapes.aggregate([
  {"$setWindowFields": {
    "partitionBy": "$x",
    "sortBy": {"_id": 1},    
    "output": {
      "cumulativeValShapeX": {
        "$sum": "$val",
        "window": {
          "documents": ["unbounded", "current"]
        }
      }
    }
  }}
]);


// $skip  (v2.2)
db.shapes.aggregate([
  {"$skip": 5}
]);


// $sort  (v2.2)
db.shapes.aggregate([
  {"$sort": {"x": 1, "y": 1}}
]);


// $sortByCount  (v3.4)
db.shapes.aggregate([
  {"$sortByCount": "$x"}
]);


// $unionWith  (v4.4)
db.shapes.aggregate([
  {"$unionWith": {"coll": "lists"}}
]);


// $unset  (v4.2)
db.shapes.aggregate([
  {"$unset": ["x"]}
]);


// $unwind  (v2.2)
db.lists.aggregate([
  {"$unwind": {"path": "$b"}}
]);
```

