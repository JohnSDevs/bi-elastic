# Assignment task BI 

## Storing data from MSSQL into Elasticsearch and visualizing with Kibana
The task is to export data from an existing MSSQL database containing various tables.

### Tools used:
* MSSQL Server 2017
  Self explanatory
  
* Elasticsearch
  Search Engine

* LogStash
Open source, server-side data processing pipeline. We can use it to ingest data from multiple sources, transform it and send to a consumer, in this case Elastic.

* JDBC SQL Driver
  Used to access the SQL Db and make query's
  
* Kibana
  Visualization tool


## Workflow:

### * 1 Prepare SQL Query in config file.
```
input {
  jdbc {
    jdbc_connection_string => "jdbc:sqlserver://localhost;databaseName=ElevatorExam;user=elastic;password=yeahright"
    jdbc_driver_class => "com.microsoft.sqlserver.jdbc.SQLServerDriver"
    jdbc_user => ""

    statement => "SELECT * FROM ElevatorService"
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "elevatorservice"
  }
}
```
### * 2 Execute LogStash
```
logstash -f sql.config
```
The above config, when executed with logstash, will access the ElevatorExam database on localhost and execute an SQL query.
The output object states where the elasticsearch server is running. The output of the SQL Query will be automatically transformed into JSON and finally inserted into an elasticsearch "index" called "elevatorservice"
** NOTE The index in Elastic has to be created forehand

## * 3 Transformation. (SSIS)
Because the nature of Elasticsearchs powerful search engine it not always necessary to make transforms, the above example will insert all the columns inside of the SQL database into Elastic. 
```
"SELECT * FROM ElevatorService"
```
Depending on your SQL database this may cause conflict errors when importing multiple tables into the same index.
So to not have to handle those problems right now, I made a custom query that will extract data into a specific Elastic index instead:
````
"SELECT Elevator.Modeltype , Building.City, ElevatorService.ServiceStatus, Building.BuildningOwner,  ElevatorService.EmployeeId
FROM Elevator 
INNER JOIN Building ON Building.Id = Elevator.BuildingId 
INNER JOIN ElevatorService ON ElevatorService.ElevatorId = Elevator.Id; "
````
Here I want to get information from different tables (JOIN) so I can analyze it with Kiabana.

## * 4 ElasticSearch
When the data is inserted in an "Index" it is basically a catalog with multiple JSON objects.
Example of a JSON object, this is a single row on an SQL db from a Table, or transformed data like method 2 above.
````
{
  "_index": "service-metrics",
  "_type": "doc",
  "_id": "Q4uUhWUBoi6_otznYG6j",
  "_score": 1,
  "_source": {
    "employeeid": "6CB8241C-F35A-43F0-84AA-ECABE5D03931",
    "city": "Stockholm",
    "@timestamp": "2018-08-29T12:06:58.313Z",
    "servicestatus": "Avslutad",
    "modeltype": "Personalhiss",
    "@version": "1"
  },
  "fields": {
    "@timestamp": [
      "2018-08-29T12:06:58.313Z"
    ]
  }
}
````
Elasticsearch is accessible through a REST interface accepting GET, POST, DELETE, PUT commands to insert/modify/view data.
In order to make query's one can user curl and make a request from the command line:
```
curl -o nul -H 'Content-Type: application/x-ndjson' -XPOST localhost:9200/elevators/doc/_bulk --data-binary Elevators.json
```
Here posting JSON objects from a file into an index called "Elevators"

Another way to search, view and analyze the data is to use a visualization tool like Kibana or Grafana for example.

## * 5 Visualzation with Kibana
I created different views so I could visualize specific metrics. 
![alt text](https://raw.githubusercontent.com/JOhnSDevs/bi-elastic/master/Kibana_dashboard.png)
