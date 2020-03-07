# Movies ETL

**Summary**

**Software:**

1. Download 7zip to merge and unzip data sources
   [https://www.7-zip.org/](https://www.7-zip.org/)
2. Jupyter notebook (Anaconda)
   [https://www.anaconda.com/distribution/](https://www.anaconda.com/distribution/)
3. Postgre (SQLDB)
   [https://www.postgresql.org/download/](https://www.postgresql.org/download/)

**Data source**

1. Wikipedia.movies.json after merge and unzip wikipedia.movies.zip files. This is a list of dictionary of movie, video, and tv-show.  Dictionany items are not in uniform well-form. They needs be cleaned up and normalized.
2. Movies_metadata.csv after merging and upzipping movies_metadata.zip files. This is a list of movies which contain extra information that wikipedia.movies do not contain.
3. rating.csv after merge and unzip rating.zip files. This is a largest file in all three files. It is a list of rating movies by users. This is the most well-form data file among 3 three files.

**Relationship**

**Wikipedia.movies** (extract ***imdb_id***)<-inner->(***imdb_id***)**Movies_metadata**(id or (***kaggle_id***)<-left->(**id**) **rating**

![ETL_Process.png](ETL_Process.png)
