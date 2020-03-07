# Movies ETL

**Summary**
---

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

Data strategy to load to Pandas dataframe:

- Wikipedia.movies.json: (This will merge with Movies_Metadata)
   
   1. Load the file to object with JSON module function. Assumption JSON return a list of dictionary movie object.
   2. Filter only movies from the list 
      a. Assumption every item in the list is a dictionary. To make sure only extract list item is dictionary item, we can add the type condition
      
      ```
      def filter_movies
         ...
         type(movie) == dict
      ...
      ```
      
      b. Assumption every dictionary item has the similar structure. To make sure to not add the item is not movie dictionary structure, we can skip all bad item 
      
      ```
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
  
Pandas dataframe transform:  

   1.wiki_movies_df: 
      a. Extract a new column ***imdb_link*** from **imdb_id** for a merge key later. It is like a key so dataframe needs to remove duplicated rows
      b. Remove all columns that have more than 90% values NaN
      c. Extract a new column ***box_office*** from **Box office** column and ***budget*** from **Budget** column. Then drop two columns **Box office** and **Budget**. This process converts the text to number by using custom functions. 
      
      ```
      def parse_dollars(s):
         #Parse a string in patterns to a number
      def parse_to_number(dataframe, oldcolumn_name, newcolumn_name):
         #Parse a column string in patterns to a new column number
         #Then drop the column string
      ```
      
      The custom functions are heavy to use regular expressions for two patterns
      
      ```
      form_one = r'\$\s*\d+\.?\d*\s*[mb]illi?on'
      form_two = r'\$\s*\d{1,3}(?:[,\.]\d{3})+(?!\s[mb]illion)'
      ```
      
      d. Extract a new column ***release_date*** from **Release date** column. The procces is similar like above step (c), but the patterns are
      
      ```
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
      
      e. Ex

