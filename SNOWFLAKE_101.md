# Snowflake 101

## Storage
* All data is stored in encrypted files (aka micro-partitions)
  * Stores data subset alongside metadata
* Snowflake Warehouses perform the import and hold raw data → stages and send data out
* Snowflake compresses files on its backend

### Staging
* **Schema**: a logical grouping of database objects like views, tables, etc.
* **Stages**: the location of saved data (within a database)
* **Staging**: the process of uploading files into one of these stages is known as staging
* **External stages**: storage locations outside the Snowflake environment in another cloud storage location (like AWS S3)
  * Allows for flexibility when it comes to ETL solutions and data sourcing
  * Ingestion speed may be impacted due to differences in region of Snowflake env and cloud storage
* **User stages**: personal storage locations for each user
  * Unique to the user
  * Other users cannot access the stage
  * Each user has a user stage allocated by default
  * Cannot be altered or dropped
* **Table stages**: storage locations within a table object
  * Great for quickly loading files into one specific table 
  * Files limited to that one table
  * Files cannot be accessed by other tables 
* **Internal named stages**: storage location objects within a Snowflake database/schema
  * Same security permissions apply as other database objects
  * Not created automatically (unlike user & table stages)
  * Lots of flexibility for loading files into multiple tables/letting multiple users access the same stage

### Warehouses
* **Staging** Warehouse: Used by any process that brings information into Snowflake
  * Switches on just long enough to stage info from source
  * Scaling outward to improve performance
  * Start small and scale up for speed
* **Integration** Warehouse: Used when you want to transform and integrate information within Snowflake
  * Avoid setting number of server clusters to maximum available for that T-shirt size; consciously increment max number of servers in the cluster to optimize processing
  * If warehouse is suspending routinely during integration, increase auto-suspend timeout
  * Can combine staging & integration warehouses
    * Avoid when needing to separate resources for data security
* **Consumption** Warehouse: Used by any process that extracts or uses information hosted within Snowflake; for example, Tableau connecting and gathering information for use by a dashboard
  * Split by function or department, depending on business needs
  * Stay on longer than integrations and staging
  * Scale only when resources are in high demand - benchmark performance using BI tools or extraction process in order to see need for scaling up
  * Increase auto-suspend timeout if BI tool or process makes infrequent requests to maximize cache usage
* **Lab** Warehouse: Optional compute reserved exclusively for high-intensity requests from Data Scientists and Data Citizens within your organisation
  * Good idea to establish a resource monitor to keep tabs on consumption & show early indications of excessive usage:
    
    ```
    CREATE RESOURCE MONITOR "RM_LAB_OPERATIONS" 
    WITH CREDIT_QUOTA = 30, frequency = 'MONTHLY'
    TRIGGERS 
    ON 80 PERCENT DO NOTIFY
    ON 120 PERCENT DO SUSPEND;

    ALTER WAREHOUSE “LAB_OPERATIONS SET RESOURCE_MONITOR = "RM_LAB_OPERATIONS";
    ```
    
  * Can notify or restrict usage after hitting quota
* Type of warehouse needed influenced by:
  1. Isolation of Workload: If you want to load a lot of information into Snowflake from various sources in parallel—as might be the case with a nightly load—or maybe you want to reserve or secure compute for specific users, it can be useful to define multiple warehouses of either a staging or consumption type.
  2. Usage Chargeback: If your data analytics delivery model relies on funding from different departments, then creating separate consumption and/or lab warehouses for each consumer group can help you to track and chargeback as applicable.

## Loading Data

* First: 
  * Create database & schema where data will be staged 
  * Create the table into which data will be loaded
* Must consider data that’s being loaded with field-type
* Semi-structured data stored using `VARIANT` field type
* Snowflake can load structured files alongside semi-structured data
  * Just make sure each field has the proper designated type
  
    ```
    COPY INTO <db>.<schema>.<input_table>  
    FROM @<db>.<schema>.<stage>
    FILE_FORMAT = (TYPE = ‘JSON’ STRIP_OUTER_ARRAY = TRUE);
    ```
  
  * Consider different file format options provided by Snowflake

## Semi-structured data

* Define end-data type and type of value being retrieved: `SELECT JSON_DATA:<JSON_key>::<data_type> FROM <input_table>`
* Can select from array using `:` or `[]` notation (latter approach receives each entry as its own column)

### Flattening Arrays

* Can’t dynamically retrieve all objects in array with the aforementioned notations
* Flattening: unpackaging semi-structured data into columnar format by converting arrays into different rows of data:
  
  ```
  SELECT <query phrase> 
  FROM <input_table>, LATERAL FLATTEN (<data>:<array>) AS <alias>
  ```
  
* Must flatten each array returned from our specific semi-structured data:
  
  ```
  SELECT
      JSON_DATA
    , JSON_DATA:Category::string
    , Regions.value:Region::string
    , "Sub-Categories".value:"Sub-Category"::string
    , EmployeeSales.value
  FROM DEMO_INPUT_TABLE
      ,   LATERAL FLATTEN (JSON_DATA:Regions) as Regions
      ,   LATERAL FLATTEN (Regions.value:"Sub-Categories") as "Sub-Categories"
      ,   LATERAL FLATTEN ("Sub-Categories".value:EmployeeSales) as EmployeeSales;
  ```

* Other file formats like Avro, ORC, Parquet, and XML an be handled similarly

### Resources

* [Zero to Snowflake series](https://interworks.com/blog/chastie/2019/10/18/zero-to-snowflake-creating-your-first-database/)
