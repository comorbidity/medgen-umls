# Medical Genetics + Unified  Medical Language System 

This package greatly simplifies the creation of local mirrors for NLM
National Library of Medicine sources, which currently includes:

-   NCBI Medical Genetics linked sources
-   UMLS Unified Medical Language System
-   PubMed annotated content

*Currently MySQL supported with plans to support generic SQL and Athena/Presto*

------------------------------------------------------------------------

# requirements

Any Unix-like operating system (including OS X) will run medgen-umls.

Medgen-umls downloads are automated via Makefile and run entirely within
bash scripts. Thus the requirements are small:

-   bash
-   wget
-   mysql (SQL presto support on the way)

------------------------------------------------------------------------

# setup

    git clone git@github.com:comorbidity/medgen-umls.git
    cd medgen-umls

# SQL

If you only want to download medgen-umls files, you do NOT need MySQL.
If you to save files to MySQL you need a running server and *mysql user*
<https://dev.mysql.com/doc/refman/8.0/en/create-user.html>

The first time you use this repository, you must run the database
scripts that create a mysql user that will be able to load the medgen
databases:

    make user

(Note that MySQL must be running and you must have the ability to use
the \"root\" superuser.)

# Makefile usage

`make all`

If you want every database downloaded and installed, simply run
`make all`. Done.

Note that due to the size of some of these databases, it could take days
to run everything the first time. Successive runs will take far less
time since only newer files will be downloaded to your local mirror.

`make <dbname>`

The Makefile in the root of the medgen-mysql directory provides ability
to `make <dbname>` for each supported database. (See below for complete
list.)

For each database desired, type `make <dbname>` to complete all of the
tasks associated with downloading, extracting, and inputting to MySQL
these particular sources.

For example, `make clinvar` will complete the following steps:

    ./mirror.sh clinvar/urls
    ./unpack.sh clinvar
    ./create_database.sh clinvar
    ./load_database.sh clinvar
    ./index_database.sh clinvar

All of the above steps can be run individually on the command line, so
if you only want to run the download script, run
`./mirror.sh <dbname>/urls`, which puts downloaded content into
`<dbname>/mirror`.

Note that already-downloaded files will not be re-downloaded, as long as
wget is convinced that the remote and local files are identical. If
these files are not identical, wget will redownload this particular
file.

This conservative updating means that you can schedule regular updates
of your medical genetics databases without overusing your connection.

Note also that datasets vary widely in how much disk space they require.
Some datasets are EXTREMELY LARGE. Average use is usally \~ 50GB.

## all-variants

|dataset|provides|
|---------|-------------|
|PubTator|NCBI Text Mined mutations for all PubMed abstracts|
|clinvar|NCBI Clinical Variants|
|GTR|NCBI Genetic Testing Reference|

## all-genes

|dataset|provides|
|---------|-------------|
|gene|NCBI Entrez Gene database|
|GeneReviews|NCBI Gene Reviews|
|GO|Gene Ontology|
|hugo|Hugo Gene Names|

## all-phenotypes

|dataset|provides|
|---------|-------------|
|medgen|NCBI Medical Genetics|
|disgenet|Disease Gene Network|
|hpo|Human Phenotype Ontology|
|orphanet|Rare diseases|

## umls

|dataset|provides|
|---------|-------------|
|umls|Unified Medical Language System|
|rxnorm|Drug concept synonyms and drug naming|


## pubmed

PubMed PMID linkages to the above sources

------------------------------------------------------------------------

command line usage

* [mirror.sh](#mirror.sh) mirrors a dataset with wget

* [create_database.sh](#create_database.sh) creates a mysql database with common loading procedures and logging

* [unpack.sh](#unpack.sh) unzip and untar mirrored content

* [load_database.sh](#load_database.sh) imports unpacked content into
mysql database

* [\$mysql_dataset](#mysql_dataset) opens mysql client for the current dataset

## mirror.sh

*example*: mirror **PubMed annotations** containing **gene mutations**
with primary sources :

    $./mirror.sh PubTator
    $./mirror.sh gene/urls
    $./mirror.sh pubmed/urls

| 

## create_database.sh

*example*: create mysql database for PubTator :: \$./create_database.sh
PubTator

## unpack.sh

*example*: unzip PubTator mirrored flat files :

    $./unpack.sh PubTator

## load_database.sh

*example*: load PubTator database with mirrored flat files :

    $./load_database.sh PubTator

------------------------------------------------------------------------

# MySQL usage

-   [\$mysql_dataset](#mysql_dataset) opens mysql client for the current
    dataset
-   [processlist](#processlist) show active SQL commands with elapsed
    time (selects, DML, indexes)
-   [info](#info) table schema with load statistics

| 

## \$mysql_dataset

*example*: open a mysql client for the PubTator database :

    source ./PubTator/db.config
    $mysql_dataset

| 

## info

*example*: show PubTator tables and statistics. *Make you have sufficent
MEMORY for the indexes!* \| To check on the status of the load see
[processlist](#processlist) . :

    mysql> call info; 
    +--------------+--------+-------------------+------------+---------+----------+----------+-----------------+
    | table_schema | ENGINE | TABLE_NAME        | TABLE_ROWS | million | data_MB  | index_MB | TABLE_COLLATION |
    +--------------+--------+-------------------+------------+---------+----------+----------+-----------------+
    | PubTator     | InnoDB | chemical2pubtator |   27453916 | 27.45   | 1549.00M | 0.00M    | utf8_unicode_ci |
    | PubTator     | InnoDB | disease2pubtator  |   27825311 | 27.83   | 1870.00M | 0.00M    | utf8_unicode_ci |
    | PubTator     | InnoDB | gene2pubtator     |   10800507 | 10.80   | 657.00M  | 0.00M    | utf8_unicode_ci |
    | PubTator     | InnoDB | log               |         36 | 0.00    | 0.02M    | 0.00M    | utf8_unicode_ci |
    | PubTator     | InnoDB | mutation2pubtator |     537030 | 0.54    | 29.56M   | 23.08M   | utf8_unicode_ci |
    | PubTator     | InnoDB | README            |         11 | 0.00    | 0.02M    | 0.00M    | utf8_general_ci |
    | PubTator     | InnoDB | species2pubtator  |   16563014 | 16.56   | 805.00M  | 0.00M    | utf8_unicode_ci |
    +--------------+--------+-------------------+------------+---------+----------+----------+-----------------+

| 

## processlist

show active SQL commands (processlist) running for this dataset. \|
**NOTE:** some datasets take a very long time to load and index.

    mysql> call ps;
    +-----+----------+-----------+----------+---------+------+-------+-----------+
    | ID  | USER     | HOST      | DB       | COMMAND | TIME | STATE | INFO      |
    +-----+----------+-----------+----------+---------+------+-------+-----------+
    | 115 | pubtator | localhost | PubTator | Query   |   74 | NULL  |           |
    |                                                                            |
    |   load data local infile 'mirror/gene2pubtator'                            |
    |   into table gene2pubtator                                                 |
    |   fields terminated by '\t' ESCAPED BY ''                                  |
    |   lines terminated by '\n' ignore 1 lines                                  |
    |                                                                            |
    +-----+----------+-----------+----------+---------+------+-------+-----------+

------------------------------------------------------------------------
