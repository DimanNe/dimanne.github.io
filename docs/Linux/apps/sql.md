title: SQL

# **SQL**


## **Run SQL against text files**

There are many tools that allow you to run SQL queries against text (csv) files.

### **Q**

```
pip install q
... | q -d, "SELECT min(c2), max(c3) FROM - GROUP BY c1" | ...
```


### **SQLite**


```
sqlite3 mydatabase.db

.mode csv
.import asdf.txt mytable
SELECT min(col2), max(col3) FROM mytable GROUP BY col1;
```


### **CSVkit**

```
pip install csvkit
... | csvsql --query "SELECT min(col2), max(col3) FROM stdin GROUP BY col1"
```


### **TextQL**

```
textql -header -sql "select count() from sample_data" sample_data.csv
```


### **OctoSQL**

Supports: .csv, .json, Excel, Parquet.

```
octosql "SELECT * FROM ./invoice2.csv"
```



### **DSQ**

It supports a wide range of file formats, including .csv, JSON, .tsv, Excel, Parquet, and .ods.

```
dsq testdata.json "SELECT * FROM {} WHERE x > 10"

```
