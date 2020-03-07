# Movies ETL

**Summary**
---

To create an automated pipeline that takes in new data, performs the appropriate transformations, and loads the data into existing tables. The requirement needs to handle data errors or unexpected data more robust

**Software:**
---
1. Download 7zip to merge and unzip data sources
   [https://www.7-zip.org/](https://www.7-zip.org/)
2. Jupyter notebook (Anaconda)
   [https://www.anaconda.com/distribution/](https://www.anaconda.com/distribution/)
3. Postgre (SQLDB)
   [https://www.postgresql.org/download/](https://www.postgresql.org/download/)

**Data source**
---
1. **Wikipedia.movies.json** after merge and unzip wikipedia.movies.zip files. This is a list of dictionary of movie, video, and tv-show.  Dictionany items are not in uniform well-form. They needs be cleaned up and normalized.
2. **Movies_metadata.csv** after merging and upzipping movies_metadata.zip files. This is a list of movies which contain extra information that wikipedia.movies do not contain.
3. **rating.csv** after merge and unzip rating.zip files. This is a largest file in all three files. It is a list of rating movies by users. This is the most well-form data file among 3 three files.

**ETL Process Overview**
---
**Wikipedia.movies** (extract ***imdb_id***)<-inner->(***imdb_id***)**Movies_metadata**(id or (***kaggle_id***)<-left->(**id**) **rating**

![ETL_Process.png](ETL_Process.png)

**ETL Process**
---
Assumption is Wikipedia.movies.json, Movies_metadata.csv, and rating.csv good files and good data for file extensions.

####Data strategy to load to Pandas dataframe:####

- Wikipedia.movies.json: (This will merge with Movies_Metadata)
   
   1. Load the file to object with JSON module function. Assumption JSON return a list of dictionary movie object.
   2. Filter only movies from the list 
      a. Assumption every item in the list is a dictionary. To make sure only extract list item is dictionary item, we can add the type condition
      
      ```python
      def filter_movies
         ...
         type(movie) == dict
      ...
      ```
      
      b. Assumption every dictionary item has the similar structure. To make sure to not add the item is not movie dictionary structure, we can skip all bad item 
      
      ```python
      def clean_movie(movie)
         try:
         ...
         except:
      ...
      ```
      
   3. Load to the list of movies to Panda dataframe (***wiki_movies_df***)
   
- Movies_metadata.csv: (This will merge with  Wikipedia.movies)
   1. Load to Panda dataframe with build-in function (***kaggle_metadata_df***)

- rating.csv: (Load to Postgre database) 
   1. Load to Panda dataframe with build-in function
  
####Pandas dataframe transform:#### 

   1. wiki_movies_df: 
      a. Extract a new column ***imdb_link*** from **imdb_id** for a merge key later. It is like a key so dataframe needs to remove duplicated rows
      b. Remove all columns that have more than 90% values NaN
      c. Extract a new column ***box_office*** from **Box office** column and ***budget*** from **Budget** column. Then drop two columns **Box office** and **Budget**. This process converts the text to number by using custom functions. The custom functions are heavy to use regular expressions for two patterns
      ```python
      def parse_dollars(s):
         #Parse a string in patterns to a number
      def parse_to_number(dataframe, oldcolumn_name, newcolumn_name):
         #Parse a column string in patterns to a new column number
         #Then drop the column string
      ```
      ```python
         form_one = r'\$\s*\d+\.?\d*\s*[mb]illi?on'
         form_two = r'\$\s*\d{1,3}(?:[,\.]\d{3})+(?!\s[mb]illion)'
      ```
      d. Extract a new column ***release_date*** from **Release date** column. The procces is similar like above step (c), but the patterns are
      ```python
        #date patterns
        # May 17, 1990
        date_form_one = r'(?:January|February|March|April|May|June|July|August|September|October|November|December)\s[123]\d,\s\d{4}'
        # 1990.03.24
        date_form_two = r'\d{4}.[01]\d.[123]\d'
        # July 1998
        date_form_three = r'(?:January|February|March|April|May|June|July|August|September|October|November|December)\s\d{4}'
        # 2018
        date_form_four = r'\d{4}'
      ```
      e. Extract a new column ***running_time*** from **Running time** column. String patterns:
      ```python
         # Capture hours and minutes or minutes
         r'(\d+)\s*ho?u?r?s?\s*(\d*)|(\d+)\s*m'
      ```
   2. kaggle_metadata_df:
      a. Convert the **Adult** column to a boolean column
      b. Filter the dataframe with **Adult** is false, drop the **Adult** column
      c. Convert the **budget**, **id**, **popularity** columns to number with pandas built-in function **to_numeric** with *errors* option **coerce** (invalid parsing will be set as NaN). If one value is not a number, the process will raise error and the ETL process will be interrupt or halted.
      d. Convert the **release_date** column to datetime with pandas built-in function **to_datetime** with *errors* option **coerce**
      
####Pandas merge dataframes:####

   1. **wiki_movies_df** and **kaggle_metadata_df** merge with pandas built-in function **merge** with *inner* join option on the key ***imdb_id*** and suffixes _wiki and _kaggle (If columns are the same names, the suffixes are used to identify a merged source). Let call the merged dataframe **movies_df**
   2. Drop redundant columns in the merged dataframe
   3. Fill and drop redundant columns
   4. Project the final set columns
   5. Rename the final set columns
   
####Save Data to PostgreSQL####

   1. Save the **movies_df** dataframe to SQL with pandas built-in function **to_sql** with **sqlalchemy** connection engine
   2. Read the data from ratings.csv to dataframe by chunk, then load data to PostgreSQL, and repeat again until read to end of file

####Let Preform Some Data Queries in PostgreSQL####
   
