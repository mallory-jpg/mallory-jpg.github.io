# Working with Python 🐍 + Snowflake ❄️

Using the combined power of Python multi-threading and Snowflake clustering to quickly load and query data.

## Multi-threaded Loading

* XS warehouses are faster and cheaper overall as long as data is prepared and load is executed properly
  * Loading compressed files <100MB
* When loading large amounts of files, folks tend to use larger warehouse with support for multi-threaded loading
  * As data volume increases, this will result in queuing regardless of warehouse size if loading in one single session
* Instead, break down loading script into smaller scripts and run them simultaneously in different sessions
  * If both scripts use same warehouse → resource contention, but reduced especially with multi-cluster warehouses that separate sessions into multiple versions of the same warehouse
  * Can still use multiple XS warehouses to load large datasets into multiple tables 

## Multi-threading

* **Multi-threading**: executing multiple sessions simultaneously
* Faster and cheaper due to:
  * greater control over warehouse
  * removal of single large queue of  `COPY INTO` commands
* If one set of files is taking longer load, it keeps running in its own session while the other file sets load elsewhere in their own sessions
### Steps
1. Define COPY INTO statement
2. Automate differences in COPY INTO statement tables and paths using Python
  * Provide a clear list of variables of table paths in Python:
    * Can input list from a spreadsheet or generate it dynamically while uploading the files into their respective stages
    * Instead of using a simple list like:
      
      ```
      variablesSimpleList = [
        ['DATABASE.SCHEMA.myStage/Table_A', 'DATABASE.SCHEMA.TABLE_A'],
        ['DATABASE.SCHEMA.myStage/Table_B', 'DATABASE.SCHEMA.TABLE_B'],
        ...
        ['S3:myS3Bucket/Table_E', 'DATABASE.SCHEMA.TABLE_E'],
        ['S3:myS3Bucket/Table_F', 'DATABASE.SCHEMA.TABLE_F'],
        ['S3:myS3Bucket/Table_G', 'DATABASE.SCHEMA.TABLE_G']
        ...
      ```
      
    * Make the list more structured and readable by including the `sourceLocation` and `destinationTable` for each list member:
      
      ```
      variablesList = [
          {
            'sourceLocation': 'DATABASE.SCHEMA.myStage/Table_A',
            'destinationTable': 'DATABASE.SCHEMA.TABLE_A'
          },
          {
            'sourceLocation': 'DATABASE.SCHEMA.myStage/Table_B',
            'destinationTable': 'DATABASE.SCHEMA.TABLE_B'
          },
          ...
      ```
      
      Using the following Python code:
      
      ```
      # Define an empty list to populate with variables
      variablesList = []

      # Loop through the members of variablesSimpleList and add them to variablesList
      # This script assumes the first entry of each member of variablesSimpleList is the sourceLocation,
      # and that the second entry is the destinationTable.
      for [sourceLocation, destinationTable] in variablesSimpleList:
      variablesList.append(
      {
      'sourceLocation': sourceLocation,
      'destinationTable': destinationTable
      }
      )
      ```
3. Using newly-created variables list, construct `COPY INTO` statements dynamically:
  ```
  # Define an empty list to populate with COPY INTO statements
  copyIntoStatements = []

  # Loop through the members of variablesList and construct the COPY INTO statements
  # Use .format() to replace the {0} and {1} with variables destinationTable and sourceLocation
  for member in variablesList:
    copyIntoStatements.append(
      '''
      COPY INTO {0}
      FROM @{1}
        FILE_FORMAT = (FORMAT_NAME = DATABASE.SCHEMA.MY_CSV_FORMAT)
        ;
      '''.format(member['destinationTable'], member['sourceLocation'])
    )
  ```
  Feel free to verify the statements with a quick print-check:
  
  ```
  for statement in copyIntoStatements:
    print(statement)
  ```
  
4. Execute each script simultaneously, one in each Snowflake session window/tab
  
### Python Threading

* Execute multiple scripts simultanesouly by using the Python to execute particular blocks of code (aka the threads) at the same time
* Runs code in parallel instead of in-sequence (queue)
  
