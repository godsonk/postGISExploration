# Exploration of PostGIS functionalities

Our goal is to create a database having two tables in which we will be importing data from csv files and performing some data exploration and manipulation operations.
We will get to explore some of the interesting functions that postgis provides and how they can be used.

The following instructions are given for Ubuntu 18.04

## Installation

To install postGIS, we run the following commands :

```
$ sudo apt-get update
$ sudo apt-get install postgis
```
We then switch to the user (Check if he has been created already)

```
$ sudo -i -u postgres
```

We will be using the command `psql` which is a terminal based command that can help us interface with postgreSQL, interact with the databases, their metadata and write queries interactively.

## Create a new database

To create a database the command is `createdb` followed by the name of the database

```
$ createdb postgis_godsonk
```

To list existing databases and make sure our database was created :
```
$ psql -l
```
Press `q` to exit the interactive prompt created.

## Connect to the database

To connect to the database, from the terminal, we uyse `psql` followed by the name of the existing database :

```
$ psql postgis_godsonk
```

The next step is to activate the postgis extenstion (without which spatial data cannot be properly understood) using the command :

```
# create extension postgis;
```
The above command should create a table name as **spatial ref sys** in our database that can be referred to understand spatial data.
It contains records for existing Spatial Reference System (SRS) with each row having the identifer (SRID) and description of a system.
To put it simply, a SRS is a coordinate based local, regional or global system that can help locate geographical entities.
Our dataset here uses the EPSG code 4326. EPSG codes identify the coordinate systems used by the European Petroleum Survey Group. They are also widely implemented and used in many GIS systems. The code EPSG:4326 corresponds to the coordinate system named WSG84 which is used by the GPS navigation system.

## Create a table

Here below are the most basic scripts to create tables in a postgis database :

```
CREATE TABLE trees
(
  gid SERIAL PRIMARY KEY NOT NULL,  
  id Numeric,
  geometry geometry,
  epsg Numeric
);
```

```
CREATE TABLE buildings
(
  gid SERIAL PRIMARY KEY NOT NULL,  
  id Numeric,
  geometry geometry,
  epsg Numeric
);
```

Don't forget the ending **;** whithout which your commands will simply not run (C developers know something about this :))

From the ubuntu terminal prompt, you can use `psql -dt` to list existing tables or once you are connected to a database, `\dt`. 
Similarly to describe a table (its schema, column types etc), you can use `psql -d` or `\d`. You can check out more options about what you can do with psql in the [documentation](https://www.postgresql.org/docs/9.3/app-psql.html)



## Data ingestion from csv files

In case, we have the exact same number of columns in the csv and in the table, we do not need to specify the columns in brackets. 

```
COPY trees (id, geometry,epsg) FROM '/path/to/extract_tree1.csv' DELIMITER ',' CSV HEADER;
```

```
COPY trees (id, geometry,epsg) FROM '/path/to/extract_tree2.csv' DELIMITER ',' CSV HEADER;
```

Copies are incremental which means the content of the second file will be added to the content of the table already imported from the first file.

If this command fails to execute, make sure the srid (Spatial Reference Identifier) has not been set to 4326 already.
The command is similar for buildings :
```
COPY buildings (id, geometry,epsg) FROM '/path/to/extract_building.csv' DELIMITER ',' CSV HEADER;
```

Let's try to view the content of the table. We get the following result :

![Alt text](./images/tablecontent.png?raw=true "The content of the table without setting the srid")

Once the data has been ingested, we can set the right SRID:

```
ALTER TABLE trees ALTER COLUMN geometry TYPE geometry(POINT, 4326) USING ST_SetSRID(geometry, 4326)
```

```
ALTER TABLE buildings ALTER COLUMN geometry TYPE geometry(MULTIPOLYGON, 4326) USING ST_SetSRID(geometry, 4326)
```

To ensure, the column is being displayed as its proper geomtric type requires, we can use the **ST_AsText** function as follows :

```
SELECT  ST_AsText(geometry) FROM extract_tree1 LIMIT 10;
```
## Exploratory analysis

To check whether or not, there are redundant geometries (polygons covering the same area or points with similar coordinates), we can leverage the `ST_AsBinary` function in the following manner :

`SELECT COUNT(DISTINCT (ST_AsBinary(geometry)))  FROM trees;`


## Querying Operations
	
###Getting the nearest neighbour
`ST_Distance(geometry1,geometry2)` can be used as the simplest way to find the nearest nei

## Other useful commands
Rename column : ALTER TABLE table_name RENAME column_name TO new_column_name;


## Notes

An SRID (Spatial Reference Identifier) is a unique identifier associated with a coordinate system. Some of them have been defined by Esri the international supplier of geographuc information system softwares or by the European Petroleum Survey Group (EPSG). A spatial reference system is composed of datum and mathematical formulas constituting a map projection that can be used to locate geographical entities.