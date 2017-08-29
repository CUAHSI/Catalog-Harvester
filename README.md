# Catalog-Harvester

#### The harvesting system for Hiscentral Catalog is composed of 
- #### Curation software
- #### Workflow-control script
- #### Multi-tier solr cores

#### **The hierarchy structure of solr cores is designed to minimize user disruption, and to disjoint harvesting and searching process, making each process more efficient**


## Features
#### Front Core Harvester (fronHarv)
  * Harvest from _data sources_ to _front cores_:  
    ServiceInfo is acquired from ```http://hiscentral.cuahsi.org/webservices/hiscentral.asmx/GetWaterOneFlowServiceInfo```
  * "Timestamp" field added: the time when the data is harvested from data sources, the value is kept the same in production core
  * Keep the data as "raw" as possible
  * Minimize the number of fields that are indexed, to speed the harvesting process
  * Running time spans from a few minutes up to a few days (esp. for NASA data sources)
  
#### Stage Core Harvester (stageHarv)
  * Harvest from _front cores_ to _stage cores_
  * Integrate front cores
  * Use Data Import Handler
  * Run on localhost, quick, ~ 1 hour for the whole database
  
#### Production Core Harvester (prodHarv)
  * Harvest from _stage cores_ to _production core_
  * “vocab” is added to the schema of front and stage cores, but not to production core 
  * unique record ID definitin: 
    ```
    NetworkID|SiteCode|VariableCode|MethodID|SourceID|QCLID
    ```  
  * logging
  * mapping from variablecode to conceptkeyword


## Prerequisites
  - 4G RAM Linux VM
  - Java 7 or above installed

## File Map
/var/solr/data: cores  
/var/solr/scripts:   
/var/solr/input  
/var/solr/logs  
         
         
## How to run

1. **Front Core Harvesters**   
    It contains a set of shell scripts, and SolrJ applications. There are different types of front core harvesters based on the data sources to be harvested.  
    1. Solr Data Import Handler
        1. customized for those services available via hiscentralrest
        ```http://hiscentralrest.azurewebsites.net/wateroneflow/```
        2. curstomized for National Groundwater Monitoring Network (NGWMN)
    2. Customized harvester for WOF 1.0 services (NetworkID=122, 129, 140, 199, 2501)
    3. Customized harvester for NASA services
     
      examples:
    ```
      /var/solr/script/frontHarv.sh active 1 3 >> /var/solr/logs/front-active.log 2>&1 &
      /var/solr/script/frontHarv.sh wof10  140 152 191 199 2501 >> /var/solr/logs/front-wof10.log 2>&1 &
      ```  

2. Stage Core Harvester  

      examples:
      ```/var/solr/script/stageHarv.sh active stage-active 1 3 228
      /var/solr/script/stageHarv.sh wof10 stage-active 122 129 140 199 2501
      /var/solr/script/stageHarv.sh NWS stage-static 187 189
      ```

3. Production Core Harvester  

    examples:
    ```
    java -jar /var/solr/script/stageHarvJar/solrStageHarvester.jar /var/solr/script/stageHarvJar/input.config.txt 5593  >> /var/solr/logs/stage2prod.log 2>&1 &
    ```


### More examples:
To update an existing service or add a new service, e.g., NetworkID=5593, commands to run:
  ```
  /var/solr/script/frontHarv.sh update 5593 >> /var/solr/logs/front-active.log 2>&1 &
  ```
  afterwards, check if front-core is ready
  
  ```
  /var/solr/script/stageHarv.sh update stage-active 5593 >> /var/solr/logs/stage-active.log 2>&1 &
  ```
  
  afterwards, check the data is in stage-active core  
  
  before running the following command, make sure each item in input.config is correct, including: source and destination core names, sql database
  
  ```
  java -jar /var/solr/script/stageHarvJar/solrStageHarvester.jar /var/solr/script/stageHarvJar/input.config 5593  >> /var/solr/logs/stage2prod.log 2>&1 &
  ```

