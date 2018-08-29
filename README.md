# Assignment task BI 

## Storing data from MSSQL into Elasticsearch and visualizing with Kibana
The task is to export data from an existing MSSQL database containing various tables.

### Tools used:
* MSSQL Server 2017
  Self explainatory
  
* Elasticsearch
  Search Engine

* LogStash
Open source, server-side data processing pipeline. We can use it to ingest data from multiple sources, transform it and send to Elastic.

* JDBC SQL Driver
  Used to access the SQL Db and make querys
  
* Kibana
  Visualisation tool


## Overview of workflow:

Prepare SQL Query in config file => Execute LogStash 
