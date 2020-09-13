# AsterixdbEval
Evaluation of Asterixdb compared to Omnisci

# Machine
AWS P2.XLARGE GPU Instance

- 1 NVIDIA K80 GPU, 4 vCPU, 61 RAM(GiB)，11.75 ECUs
- NVIDIA K80 GPU
  - 4992 NVIDIA CUDA Cores
  - 24GB memory
  - 480 GB/s aggregate memory bandwidth 
- Price
  - 0.900 $/hour
  - 21.6 $/day
  
# Omnisci Database Architecture

Hybrid compute architecture that utilizes GPU, CPU and storage. When queries are executed, the Core database optimizer utilizes GPU RAM first if its available. The database will cache hot data. If GPU RAM is filled or unavailable, it will utilizes CPU RAM then SSD. The COPY FROM command that used for loading data into Omnisci will loads the data into the database's disk memory. The database only moves the data and columns on which the queries are asking to CPU or GPU.


# Data File

- AsterixDB
  - AsterixDB is an object database. It supports ADM and Json file format
- Omnisci
  - Omnisci is a relational database. It supports CSV and TXT file format

The experiment uses 20 million tweets data (22GB in ADM or Json format). In order to load data to Omnisci, I convert the Json file into CSV format by using python library Json. 

# Queries

AsterixDB provides inverted index based key word search while Omnisci only supports like statement for string matching function. Here we focus on comparing AsterixDB keyword search, ftcontains statement, with Omnisci like statement because AsterixDB like statement is too slow. For the same queries while AsterixDB keyword search and Omnisci like statement runs in million seconds, AsterixDB like statement needs to take more than ten seconds.

## sample queries
- AsterixDB keyword search using ftcontains
  - select t.`id` as `id`,t.`coordinate` as `coordinate`,
t.`place`.`bounding_box` as `place.bounding_box`
from twitter.ds_tweet t
where t.`create_at` >= datetime('2015-11-25T00:00:00.000') 
and t.`create_at` < datetime('2015-11-27T00:00:00.000') and 
ftcontains(t.`text`, ['hurricane'], {'mode':'all'}) and 
t.`geo_tag`.`stateID` in [ 37,51,24,11,10,34,42,9,44,48,
35,4,40,6,20,32,8,49,12,22,28,1,13,45,5,47,21,29,54,17,
18,39,19,55,26,27,31,56,41,46,16,30,53,38,25,36,50,33,23,2 ];
- AsterixDB like statement
  - select t.`id` as `id`,t.`coordinate` as `coordinate`,
t.`place`.`bounding_box` as `place.bounding_box`
from twitter.ds_tweet t
where t.`create_at` >= datetime('2015-11-25T00:00:00.000') 
and t.`create_at` < datetime('2015-11-27T00:00:00.000') and 
t.`text` like '%hurricane%' and 
t.`geo_tag`.`stateID` in [ 37,51,24,11,10,34,42,9,44,48,
35,4,40,6,20,32,8,49,12,22,28,1,13,45,5,47,21,29,54,17,
18,39,19,55,26,27,31,56,41,46,16,30,53,38,25,36,50,33,23,2 ];
- Omnisci like statement
  - Select t.id as id, t.coordinate as coordinate, 
t.place_bounding_box as bounding_box 
from ds_tweets as t 
where t.create_at >= '2015-11-25T00:00:00.000' 
and t.create_at < '2015-11-27T00:00:00.000' 
and t.text_ like '%hurricane%' 
and t.geo_tag_stateID in (37,51,24,11,10,34,42,9,44,48,35,4,40,6,20,32,8,49,
12,22,28,1,13,45,5,47,21,29,54,17,18,39,19,55,26,27,31,
56,41,46,16,30,53,38,25,36,50,33,23,2);

|Query|AsterixDB-keyword search|Omnisci-like statement|AsterixDB-like statement|
| --- | --- | --- | --- |
|Query by keyword wildfire|0.174s|0.317s|11.335s|
|Query by keyword hurricane|0.156s|0.152s|11.459s|
