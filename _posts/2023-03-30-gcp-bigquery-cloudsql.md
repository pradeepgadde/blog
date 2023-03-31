---

layout: single
title:  "Introduction to SQL for BigQuery and Cloud SQL"
date:   2023-03-30 16:59:04 +0530
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner.png
  og_image: /assets/images/gcp-banner.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Introduction to SQL for BigQuery and Cloud SQL

## 1. The basics of SQL

SQL allows you to get information from "structured datasets".  Structured datasets have clear rules and formatting and often times are  organized into tables, or data that's formatted in rows and columns.

An example of *unstructured data* would be an image file.  Unstructured data is inoperable with SQL and cannot be stored in  BigQuery datasets or tables (at least natively.) To work with image data (for instance), you would use a service like [Cloud Vision](https://google.qwiklabs.com/catalog_lab/1112), perhaps through its [API](https://google.qwiklabs.com/catalog_lab/1241) directly.



A Database is essentially a *collection of one or more tables*. SQL is a structured database management tool.

### SELECT and FROM

SQL has predefined *keywords* which you use to translate your  question into the pseudo-english SQL syntax so you can get the database  engine to return the answer you want.

The most essential keywords are `SELECT` and `FROM`:

- Use `SELECT` to specify what fields you want to pull from your dataset.
- Use `FROM` to specify what table or tables we want to pull our data from.

### WHERE

The `WHERE` keyword is another SQL command that filters tables for specific column values.

### GROUP BY

The `GROUP BY` keyword will aggregate result-set rows that share common criteria (e.g. a column value) and will return all of the  unique entries found for such criteria. `GROUP BY` will output the unique  column values found in the table.

### COUNT

The `COUNT()` function will return the number of rows that share the same criteria (e.g. column value). This can be very useful in tandem with a `GROUP BY`.

### AS

SQL also has an `AS` keyword, which creates an *alias* of a table or column. An alias is a new name that's given to the returned column or table—whatever `AS` specifies.



### ORDER BY

The `ORDER BY` keyword sorts the returned data from a query in ascending or descending order based on a specified criteria or column value. 

## 2. Exploring the BigQuery console

[BigQuery](https://cloud.google.com/bigquery/) is a fully-managed petabyte-scale data warehouse that runs on the  Google Cloud. Data analysts and data scientists can quickly query and  filter large datasets, aggregate results, and perform complex operations without having to worry about setting up and managing servers. It comes in the form of a command line tool (pre installed in cloudshell) or a  web console—both ready for managing and querying data housed in Google  Cloud projects.

> In BigQuery, *projects contain datasets, and datasets contain tables*.

### Uploading queryable data

In this section you pull in some public data into your project so you can practice running SQL commands in BigQuery.

1. Click on **+ ADD**.
2. Choose **Star a project by name**.
3. Enter project name as **bigquery-public-data**.
4. Click **STAR**.

It's important to note that you are still working out of your lab project in this new tab. All you did was pull a publicly accessible project that contains datasets and tables into BigQuery for analysis — you didn't *switch over* to that project.





## 3. Working with Cloud SQL

[Cloud SQL](https://cloud.google.com/sql/) is a fully-managed database service that makes it easy to set up,  maintain, manage, and administer your relational PostgreSQL and MySQL  databases in the cloud. There are two formats of data accepted by Cloud  SQL: dump files (.sql) or CSV files (.csv). 



In the Query Results section click **SAVE RESULTS** > **CSV(local file)**. This initiates a download, which saves this query as a CSV file.

### Upload CSV files to Cloud Storage

1. Go to the Cloud Console where you'll create a storage bucket where you can upload the files you just created.
2. Select **Navigation menu** > **Cloud Storage** > **Buckets**, and then click **CREATE BUCKET**.



## 4. Create a Cloud SQL instance

select **Navigation menu** > **SQL**.

1. Click **CREATE INSTANCE > Choose MySQL** .
2. Enter instance id as **qwiklabs-demo**.
3. Enter a secure password in the **Password** field (remember it!).
4. Select the database version as **MySQL 5.7**.
5. Set the **Multi zones (Highly available)** field as 
6. Click **CREATE INSTANCE**.





## 5. New queries in Cloud SQL

### CREATE keyword (databases and tables)

Now that you have a Cloud SQL instance up and running, create a database inside of it using the Cloud Shell Command Line. Copy the Cloud Shell link below and paste in a new browser incognito tab.





### Create a database in Cloud Shell



```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-92c7778c7284.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_03_2f91aaab2f20@cloudshell:~ (qwiklabs-gcp-03-92c7778c7284)$ gcloud auth login --no-launch-browser

You are already authenticated with gcloud when running
inside the Cloud Shell and so do not need to run this
command. Do you wish to proceed anyway?

Do you want to continue (Y/n)?  y

Go to the following link in your browser:

    https://accounts.google.com/o/oauth2/auth?response_type=code&client_id=32555940559.apps.googleusercontent.com&redirect_uri=https%3A%2F%2Fsdk.cloud.google.com%2Fauthcode.html&scope=openid+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fuserinfo.email+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fcloud-platform+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fappengine.admin+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fsqlservice.login+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fcompute+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Faccounts.reauth&state=bwci1CKqxnlMyEnH2MKCrnbeCJ8u1R&prompt=consent&access_type=offline&code_challenge=bd_B8zvY2FRgzQj55wBhwAdjHBN-1qQFmQDINw9XmKY&code_challenge_method=S256

Enter authorization code: 4/0AVHEtk4ZfJ-GigCHq5Jac2qxeP4TR5MD2ZDLchbZUQ6fl2KouBfArYad2tVfUCtaey0_ow

You are now logged in as [student-03-2f91aaab2f20@qwiklabs.net].
Your current project is [qwiklabs-gcp-03-92c7778c7284].  You can change this setting by running:
  $ gcloud config set project PROJECT_ID
student_03_2f91aaab2f20@cloudshell:~ (qwiklabs-gcp-03-92c7778c7284)$ gcloud sql connect qwiklabs-demo --user=root --quiet
Allowlisting your IP for incoming connection for 5 minutes...done.     
Connecting to database with SQL user [root].Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 102
Server version: 5.7.40-google-log (Google)

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>

```



```sql
mysql> CREATE DATABASE bike;
Query OK, 1 row affected (0.16 sec)

mysql>
```



```sql
mysql> USE bike;
Database changed
mysql> CREATE TABLE london1 (start_station_name VARCHAR(255), num INT);
Query OK, 0 rows affected (0.18 sec)

mysql>
```

```sql
mysql> USE bike;
Database changed
mysql> CREATE TABLE london2 (end_station_name VARCHAR(255), num INT);
Query OK, 0 rows affected (0.18 sec)

mysql>
```

```sql
mysql> SELECT * FROM london1;
Empty set (0.16 sec)

mysql> SELECT * FROM london2;
Empty set (0.15 sec)

mysql>
```

You see "empty set" because you haven't yet loaded the data.



### Upload CSV files to tables

Return to the Cloud SQL console. You will now upload the `start_station_name` and `end_station_name` CSV files into your newly created london1 and london2 tables.

1. In your Cloud SQL instance page, click **IMPORT**.
2. In the Cloud Storage file field, click **Browse**, and then click the arrow next to your bucket name, and then click `start_station_data.csv`. Click **Select**.
3. Select **CSV** as File format.
4. Select the `bike` database and type in "`london1`" as your table.
5. Click **Import**.

Do the same for the other CSV file.



```sql
mysql> SELECT * FROM london1;
+----------------------------------------------------------+--------+
| start_station_name                                       | num    |
+----------------------------------------------------------+--------+
| start_station_name                                       |      0 |
| Belgrove Street , King's Cross                           | 234458 |
| Hyde Park Corner, Hyde Park                              | 215629 |
| Waterloo Station 3, Waterloo                             | 201630 |
| Black Lion Gate, Kensington Gardens                      | 161952 |
| Albert Gate, Hyde Park                                   | 155647 |
| Waterloo Station 1, Waterloo                             | 145910 |
| Wormwood Street, Liverpool Street                        | 119447 |
| Hop Exchange, The Borough                                | 115135 |
| Wellington Arch, Hyde Park                               | 110260 |
| Triangle Car Park, Hyde Park                             | 108347 |
| Finsbury Circus, Liverpool Street                        | 105810 |
| Brushfield Street, Liverpool Street                      | 103114 |
| Bethnal Green Road, Shoreditch                           | 100005 |
| Regent's Row , Haggerston                                |  99706 |
| Craven Street, Strand                                    |  96609 |
| Duke Street Hill, London Bridge                          |  96557 |
| Palace Gate, Kensington Gardens                          |  91917 |
| Old Street Station, St. Luke's                           |  89112 |
| Queen Street 1, Bank                                     |  87321 |
| Newgate Street , St. Paul's                              |  86980 |
| Speakers' Corner 1, Hyde Park                            |  85835 |
| Cheapside, Bank                                          |  85285 |
| Shoreditch High Street, Shoreditch                       |  84286 |
| Crosswall, Tower                                         |  81622 |
| Waterloo Station 2, Waterloo                             |  80840 |
| Soho Square , Soho                                       |  79684 |
| Whitehall Place, Strand                                  |  76669 |
| Exhibition Road, Knightsbridge                           |  76581 |
| Storey's Gate, Westminster                               |  75946 |
| Moorfields, Moorgate                                     |  75453 |
| Serpentine Car Park, Hyde Park                           |  74903 |
| Holborn Circus, Holborn                                  |  74748 |
| William IV Street, Strand                                |  74550 |
| South Kensington Station, South Kensington               |  74467 |
| Bankside Mix, Bankside                                   |  73589 |
| Sun Street, Liverpool Street                             |  73579 |
| Green Park Station, Mayfair                              |  72126 |
| Drury Lane, Covent Garden                                |  71187 |
| Malet Street, Bloomsbury                                 |  70015 |
| Eagle Wharf Road, Hoxton                                 |  69753 |
| Tooley Street, Bermondsey                                |  69470 |
| Islington Green, Angel                                   |  69460 |
| Kennington Lane Rail Bridge, Vauxhall                    |  69407 |
| Tower Gardens , Tower                                    |  68561 |
| Great Tower Street, Monument                             |  68376 |
| Little Argyll Street, West End                           |  67660 |
| Queen's Gate, Kensington Gardens                         |  67596 |
| Aldersgate Street, Barbican                              |  67244 |
| Speakers' Corner 2, Hyde Park                            |  67010 |
| Notting Hill Gate Station, Notting Hill                  |  67006 |
| Vauxhall Cross, Vauxhall                                 |  66349 |
| St. James's Square, St. James's                          |  65477 |
| Leonard Circus , Shoreditch                              |  65014 |
| Baylis Road, Waterloo                                    |  64830 |
| Red Lion Street, Holborn                                 |  64692 |
| Moor Street, Soho                                        |  64093 |
| Nesham Street, Wapping                                   |  64011 |
| Berry Street, Clerkenwell                                |  63480 |
| Leman Street, Aldgate                                    |  62747 |
| Eccleston Place, Victoria                                |  62436 |
| Ashley Place, Victoria                                   |  62210 |
| Fulham Broadway, Walham Green                            |  61702 |
| Hatton Wall, Holborn                                     |  61427 |
| Park Lane , Hyde Park                                    |  61415 |
| Great Marlborough Street, Soho                           |  61110 |
| Finsbury Leisure Centre, St. Luke's                      |  60917 |
| Dunston Road , Haggerston                                |  60541 |
| Wardour Street, Soho                                     |  60530 |
| Winsland Street, Paddington                              |  60249 |
| British Museum, Bloomsbury                               |  59185 |
| Bermondsey Street, Bermondsey                            |  58513 |
| Gloucester Road Station, South Kensington                |  58124 |
| Derry Street, Kensington                                 |  57083 |
| Godliman Street, St. Paul's                              |  56843 |
| West Smithfield Rotunda, Farringdon                      |  56780 |
| Bank of England Museum, Bank                             |  56299 |
| Tavistock Street, Covent Garden                          |  56188 |
| Jubilee Gardens, South Bank                              |  56071 |
| Warren Street Station, Euston                            |  55857 |
| Bath Street, St. Luke's                                  |  55162 |
| Clerkenwell Green, Clerkenwell                           |  55092 |
| Endsleigh Gardens, Euston                                |  54843 |
| Aquatic Centre, Queen Elizabeth Olympic Park             |  54824 |
| Queen's Circus, Battersea Park                           |  53236 |
| Queen Street 2, Bank                                     |  51658 |
| Flood Street, Chelsea                                    |  51631 |
| Graham Street, Angel                                     |  51586 |
| Liverpool Road (N1 Centre), Angel                        |  51256 |
| Pall Mall East, West End                                 |  51201 |
| Bayswater Road, Hyde Park                                |  51127 |
| Wright's Lane, Kensington                                |  51118 |
| Curlew Street, Shad Thames                               |  50967 |
| Braham Street, Aldgate                                   |  50803 |
| Bunhill Row, Moorgate                                    |  50094 |
| Howick Place, Westminster                                |  49663 |
| Southwark Station 2, Southwark                           |  49303 |
| Museum of London, Barbican                               |  49256 |
| Doric Way , Somers Town                                  |  49190 |
| Wren Street, Holborn                                     |  49144 |
| Tysoe Street, Clerkenwell                                |  48977 |
| Hampstead Road, Euston                                   |  48974 |
| Belgrave Road, Victoria                                  |  48788 |
| Southampton Street, Strand                               |  48604 |
| Limerston Street, West Chelsea                           |  48290 |
| South Wharf Road, Paddington                             |  48245 |
| Russell Square Station, Bloomsbury                       |  48007 |
| Queen Victoria Street, St. Paul's                        |  47904 |
| Hatton Garden, Holborn                                   |  47629 |
| Imperial College, Knightsbridge                          |  47473 |
| Ethelburga Estate, Battersea Park                        |  47336 |
| Elizabeth Bridge, Victoria                               |  47226 |
| St. Bride Street, Holborn                                |  47129 |
| Poured Lines, Bankside                                   |  46919 |
| Wells Street, Fitzrovia                                  |  46795 |
| Cumberland Gate, Hyde Park                               |  46785 |
| Belvedere Road, South Bank                               |  46712 |
| Northumberland Avenue, Strand                            |  46700 |
| Sedding Street, Sloane Square                            |  46680 |
| Bouverie Street, Temple                                  |  46222 |
| Somerset House, Strand                                   |  46149 |
| Kennington Cross, Kennington                             |  46016 |
| Bayley Street , Bloomsbury                               |  45844 |
| Old Quebec Street, Marylebone                            |  45578 |
| Boston Place, Marylebone                                 |  45237 |
| New Fetter Lane, Holborn                                 |  45236 |
| Wellington Street , Strand                               |  45158 |
| Pritchard's Road, Bethnal Green                          |  45137 |
| Milroy Walk, South Bank                                  |  44967 |
| Granby Street, Shoreditch                                |  44716 |
| Westbridge Road, Battersea                               |  44653 |
| Bolsover Street, Fitzrovia                               |  44379 |
| Hinde Street, Marylebone                                 |  44003 |
| Guilford Street , Bloomsbury                             |  43991 |
| Sadlers Sports Centre, Finsbury                          |  43960 |
| Shoreditch Park, Hoxton                                  |  43849 |
| Falcon Road, Clapham Junction                            |  43791 |
| Castlehaven Road, Camden Town                            |  43786 |
| Waterloo Place, St. James's                              |  43673 |
| Exhibition Road Museums, South Kensington                |  43377 |
| Queen Mother Sports Centre, Victoria                     |  43296 |
| Drummond Street , Euston                                 |  43276 |
| Montpelier Street, Knightsbridge                         |  43237 |
| Stonecutter Street, Holborn                              |  43231 |
| Bonny Street, Camden Town                                |  42978 |
| Fire Brigade Pier, Vauxhall                              |  42892 |
| Fournier Street, Whitechapel                             |  42766 |
| New Inn Yard, Shoreditch                                 |  42666 |
| Great Russell Street, Bloomsbury                         |  42606 |
| Natural History Museum, South Kensington                 |  42575 |
| Greenland Road, Camden Town                              |  42398 |
| Watney Street, Shadwell                                  |  42245 |
| Tanner Street, Bermondsey                                |  42007 |
| Binfield Road, Stockwell                                 |  41984 |
| Flamborough Street, Limehouse                            |  41964 |
| Parsons Green Station, Parsons Green                     |  41847 |
| Chadwell Street, Angel                                   |  41758 |
| New Globe Walk, Bankside                                 |  41692 |
| Pott Street, Bethnal Green                               |  41584 |
| Howland Street, Fitzrovia                                |  41447 |
| Abingdon Green, Westminster                              |  41428 |
| Lancaster Gate , Bayswater                               |  41352 |
| Strand, Strand                                           |  41321 |
| Marylebone Lane, Marylebone                              |  41235 |
| Snow Hill, Farringdon                                    |  41197 |
| Park Road (Baker Street), The Regent's Park              |  41131 |
| Devonshire Terrace, Bayswater                            |  40838 |
| Brick Lane Market, Shoreditch                            |  40700 |
| Kensington Church Street, Kensington                     |  40556 |
| Warwick Avenue Station, Maida Vale                       |  40527 |
| Wenlock Road , Hoxton                                    |  40463 |
| Victoria Park Road, Hackney Central                      |  40412 |
| Theobald's Road , Holborn                                |  40301 |
| Westferry DLR, Limehouse                                 |  40072 |
| Alfred Place, Bloomsbury                                 |  39952 |
| Golden Square, Soho                                      |  39862 |
| Abbey Orchard Street, Westminster                        |  39749 |
| Northington Street , Holborn                             |  39681 |
| All Saints Church, Portobello                            |  39557 |
| Frith Street, Soho                                       |  39374 |
| Rampayne Street, Pimlico                                 |  39358 |
| York Hall, Bethnal Green                                 |  39313 |
| Stamford Street, South Bank                              |  39206 |
| Bramham Gardens, Earl's Court                            |  39121 |
| Royal London Hospital, Whitechapel                       |  38957 |
| Palace Gardens Terrace, Notting Hill                     |  38948 |
| Great Titchfield Street, Fitzrovia                       |  38824 |
| Panton Street, West End                                  |  38813 |
| Tallis Street, Temple                                    |  38759 |
| Cartwright Gardens , Bloomsbury                          |  38618 |
| Woodstock Grove, Shepherd's Bush                         |  38467 |
| Seville Street, Knightsbridge                            |  38386 |
| Drayton Gardens, West Chelsea                            |  38299 |
| Sloane Avenue, Knightsbridge                             |  38250 |
| Golden Lane, Barbican                                    |  38021 |
| Devonshire Square, Liverpool Street                      |  37929 |
| Pitfield Street North,Hoxton                             |  37685 |
| St. Mary Axe, Aldgate                                    |  37605 |
| Porchester Place, Paddington                             |  37543 |
| Broadcasting House, Marylebone                           |  37437 |
| Southampton Place, Holborn                               |  37429 |
| Shadwell Station, Shadwell                               |  37235 |
| Finsbury Square , Moorgate                               |  37202 |
| Bricklayers Arms, Borough                                |  37144 |
| Erin Close, Walham Green                                 |  36850 |
| St. George's Square, Pimlico                             |  36773 |
| Gloucester Street, Pimlico                               |  36770 |
| Buckingham Gate, Westminster                             |  36727 |
| Whiteley's, Bayswater                                    |  36278 |
| Chapel Place, Marylebone                                 |  36226 |
| Norton Folgate, Liverpool Street                         |  36095 |
| Stephendale Road, Sands End                              |  36035 |
| Haggerston Road, Haggerston                              |  36022 |
| Dorset Square, Marylebone                                |  36011 |
| Torrens Street, Angel                                    |  35961 |
| Monument Street, Monument                                |  35907 |
| Parson's Green , Parson's Green                          |  35856 |
| Brunswick Square, Bloomsbury                             |  35748 |
| Portland Place, Marylebone                               |  35633 |
| Old Montague Street, Whitechapel                         |  35541 |
| Hertford Road, De Beauvoir Town                          |  35536 |
| Jubilee Plaza, Canary Wharf                              |  35422 |
| Christian Street, Whitechapel                            |  35403 |
| Cardinal Place, Victoria                                 |  35237 |
| Eastbourne Mews, Paddington                              |  35134 |
| The Green Bridge, Mile End                               |  35127 |
| LMU Commercial Road, Whitechapel                         |  35055 |
| Horseferry Road, Westminster                             |  34934 |
| Taviton Street, Bloomsbury                               |  34876 |
| Knightsbridge, Hyde Park                                 |  34848 |
| Broadwick Street, Soho                                   |  34835 |
| Old Brompton Road, South Kensington                      |  34777 |
| Turquoise Island, Notting Hill                           |  34593 |
| Gower Place , Euston                                     |  34560 |
| Tavistock Place, Bloomsbury                              |  34560 |
| Gloucester Road (North), Kensington                      |  34523 |
| Cadogan Gardens, Chelsea                                 |  34476 |
| Columbia Road, Shoreditch                                |  34333 |
| Sackville Street, Mayfair                                |  34306 |
| Phillimore Gardens, Kensington                           |  34201 |
| Earnshaw Street , Covent Garden                          |  34165 |
| Barons Court Station, West Kensington                    |  33988 |
| Cadogan Close, Victoria Park                             |  33986 |
| Twig Folly Bridge, Mile End                              |  33826 |
| George Street, Marylebone                                |  33811 |
| Stepney Green Station, Stepney                           |  33560 |
| Millbank Tower, Pimlico                                  |  33461 |
| Ilchester Gardens, Bayswater                             |  33458 |
| Ada Street, Hackney Central                              |  33354 |
| Beaumont Street, Marylebone                              |  33288 |
| Albert Bridge Road, Battersea Park                       |  33162 |
| Vicarage Crescent, Battersea                             |  33159 |
| Harriet Street, Knightsbridge                            |  33049 |
| Baldwin Street, St. Luke's                               |  32931 |
| Wapping Lane, Wapping                                    |  32847 |
| Christopher Street, Liverpool Street                     |  32821 |
| City Road, Angel                                         |  32807 |
| Kingsway Southbound, Strand                              |  32612 |
| Green Street, Mayfair                                    |  32606 |
| Roscoe Street, St. Luke's                                |  32518 |
| Walworth Road, Elephant & Castle                         |  32488 |
| Tachbrook Street, Victoria                               |  32479 |
| Page Street, Westminster                                 |  32477 |
| Sardinia Street, Holborn                                 |  32462 |
| Great Percy Street, Clerkenwell                          |  32441 |
| Charlotte Street, Fitzrovia                              |  32402 |
| Hammersmith Road, Hammersmith                            |  32385 |
| Crawford Street, Marylebone                              |  32331 |
| Waterloo Road, South Bank                                |  32327 |
| Ebury Bridge, Pimlico                                    |  32265 |
| Empire Square, The Borough                               |  32228 |
| St. Chad's Street, King's Cross                          |  32050 |
| Smith Square, Westminster                                |  31994 |
| Euston Road, Euston                                      |  31896 |
| Penywern Road, Earl's Court                              |  31862 |
| Queen's Gate (South), South Kensington                   |  31672 |
| Windsor Terrace, Hoxton                                  |  31654 |
| Holy Trinity Brompton, Knightsbridge                     |  31554 |
| Bell Lane, Liverpool Street                              |  31511 |
| Alderney Street, Pimlico                                 |  31480 |
| Commercial Street, Shoreditch                            |  31438 |
| Clifton Road, Maida Vale                                 |  31246 |
| Farringdon Lane, Clerkenwell                             |  31198 |
| Olympia Way, Olympia                                     |  31191 |
| Barbican Centre, Barbican                                |  31172 |
| East Road, Hoxton                                        |  31157 |
| Appold Street, Liverpool Street                          |  31144 |
| Edgware Road Station, Marylebone                         |  30878 |
| Kennington Oval, Oval                                    |  30818 |
| Embankment (Savoy), Strand                               |  30759 |
| Gloucester Avenue, Camden Town                           |  30661 |
| Crisp Road, Hammersmith                                  |  30610 |
| Caldwell Street, Stockwell                               |  30596 |
| Jewry Street, Aldgate                                    |  30452 |
| Bishop's Bridge Road West, Bayswater                     |  30374 |
| Lower Marsh, Waterloo                                    |  30367 |
| Normand Park, West Kensington                            |  30262 |
| Grosvenor Square, Mayfair                                |  30239 |
| Southwick Street, Paddington                             |  30233 |
| Fisherman's Walk West, Canary Wharf                      |  30208 |
| North Audley Street, Mayfair                             |  30177 |
| New Cavendish Street, Marylebone                         |  30144 |
| Reardon Street, Wapping                                  |  30048 |
| Ontario Street, Elephant & Castle                        |  30031 |
| Chelsea Bridge, Pimlico                                  |  29993 |
| St. Peter's Terrace, Fulham                              |  29980 |
| Prince Albert Road, The Regent's Park                    |  29975 |
| Bell Street , Marylebone                                 |  29924 |
| Old Ford Road, Bethnal Green                             |  29849 |
| De Vere Gardens, Kensington                              |  29795 |
| Cephas Street, Bethnal Green                             |  29718 |
| Red Lion Square, Holborn                                 |  29691 |
| Percival Street, Finsbury                                |  29675 |
| Cloudesley Road, Angel                                   |  29654 |
| Bury Place, Holborn                                      |  29607 |
| Macclesfield Rd, St Lukes                                |  29593 |
| The Tennis Courts, The Regent's Park                     |  29574 |
| Snowsfields, London Bridge                               |  29520 |
| Goldsmith's Row, Haggerston                              |  29476 |
| Prince Consort Road, Knightsbridge                       |  29305 |
| Rodney Road , Walworth                                   |  29255 |
| Swan Street, The Borough                                 |  29206 |
| Wellington Row, Bethnal Green                            |  29172 |
| Fore Street, Guildhall                                   |  29116 |
| Union Street, The Borough                                |  28996 |
| Durant Street, Bethnal Green                             |  28772 |
| Northdown Street, King's Cross                           |  28678 |
| St. Katharine's Way, Tower                               |  28657 |
| Queen's Gate (North), Kensington                         |  28609 |
| Royal College Street, Camden Town                        |  28584 |
| Portman Square, Marylebone                               |  28506 |
| Harrington Square 2, Camden Town                         |  28485 |
| Chesilton Road, Fulham                                   |  28481 |
| Phene Street, Chelsea                                    |  28412 |
| Kennington Road  , Vauxhall                              |  28382 |
| Blenheim Crescent, Ladbroke Grove                        |  28268 |
| Albert Gardens, Stepney                                  |  28264 |
| Manresa Road, Chelsea                                    |  28253 |
| Sutton Street, Shadwell                                  |  28172 |
| Crinan Street, King's Cross                              |  28143 |
| Borough High Street, The Borough                         |  28107 |
| Coomer Place, West Kensington                            |  28043 |
| St. Martin's Street, West End                            |  27892 |
| Hereford Road, Bayswater                                 |  27717 |
| Westbourne Grove, Bayswater                              |  27713 |
| Ampton Street , Clerkenwell                              |  27706 |
| Concert Hall Approach 1, South Bank                      |  27604 |
| Lansdowne Road, Ladbroke Grove                           |  27522 |
| Nutford Place, Marylebone                                |  27386 |
| Finsbury Library , Finsbury                              |  27373 |
| Jubilee Street, Stepney                                  |  27341 |
| Maclise Road, Olympia                                    |  27300 |
| Chepstow Villas, Notting Hill                            |  27243 |
| Queen's Gate (Central), South Kensington                 |  27207 |
| Warwick Square, Pimlico                                  |  27169 |
| Baker Street, Marylebone                                 |  27154 |
| North Wharf Road, Paddington                             |  27149 |
| Wandsworth Town Station, Wandsworth                      |  27079 |
| Fashion Street, Whitechapel                              |  27062 |
| Westfield Library Corner, Shepherd's Bush                |  27040 |
| Concert Hall Approach 2, South Bank                      |  26995 |
| Bedford Way, Bloomsbury                                  |  26939 |
| New Road 1 , Whitechapel                                 |  26927 |
| Grove End Road, St. John's Wood                          |  26922 |
| Long Lane , Bermondsey                                   |  26917 |
| Driffield Road, Old Ford                                 |  26905 |
| Philpot Street, Whitechapel                              |  26771 |
| Parkway, Camden Town                                     |  26736 |
| Royal Avenue 2, Chelsea                                  |  26706 |
| Newton Street, Covent Garden                             |  26697 |
| Danvers Street, West Chelsea                             |  26692 |
| Podium, Queen Elizabeth Olympic Park                     |  26656 |
| Hardwick Street, Clerkenwell                             |  26550 |
| Charles II Street, West End                              |  26525 |
| Great Suffolk Street, The Borough                        |  26487 |
| Hoxton Station, Hoxton                                   |  26426 |
| Great Dover Street, The Borough                          |  26374 |
| Margery Street, Clerkenwell                              |  26360 |
| Globe Town Market, Bethnal Green                         |  26290 |
| Scala Street, Fitzrovia                                  |  26164 |
| St. George Street, Mayfair                               |  26130 |
| Pont Street, Knightsbridge                               |  26108 |
| Saunders Ness Road, Cubitt Town                          |  26092 |
| Wood Street, Guildhall                                   |  26075 |
| Garnet Street, Shadwell                                  |  26007 |
| Gunmakers Lane, Old Ford                                 |  25897 |
| Seymour Place, Marylebone                                |  25858 |
| Fanshaw Street, Hoxton                                   |  25833 |
| London Zoo Car Park, The Regent's Park                   |  25791 |
| Albert Embankment, Vauxhall                              |  25754 |
| Arlington Road, Camden Town                              |  25753 |
| Park Street, Bankside                                    |  25650 |
| St. Luke's Church, Chelsea                               |  25588 |
| Harcourt Terrace, West Brompton                          |  25585 |
| West Cromwell Road, Earl's Court                         |  25578 |
| Westminster University, Marylebone                       |  25545 |
| Clinton Road, Mile End                                   |  25456 |
| Archbishop's Park, Waterloo                              |  25456 |
| Albany Street, The Regent's Park                         |  25270 |
| Rainville Road, Hammersmith                              |  25187 |
| Butler Place, Westminster                                |  25156 |
| Bethnal Green Garden, Bethnal Green                      |  25129 |
| Salmon Lane, Limehouse                                   |  25111 |
| Lexham Gardens, Kensington                               |  25090 |
| Bow Road Station, Bow                                    |  25038 |
| Putney Bridge Station, Fulham                            |  25001 |
| Black Prince Road, Vauxhall                              |  24971 |
| Greyhound Road, Hammersmith                              |  24929 |
| Grant Road East, Clapham Junction                        |  24923 |
| Cheshire Street, Bethnal Green                           |  24900 |
| Watney Market, Stepney                                   |  24888 |
| Westbourne Park Road, Portobello                         |  24849 |
| Eaton Square, Belgravia                                  |  24742 |
| Paddington Street, Marylebone                            |  24716 |
| Pancras Road, King's Cross                               |  24640 |
| Furze Green, Bow                                         |  24636 |
| Knaresborough Place, Earl's Court                        |  24610 |
| St. John Street, Finsbury                                |  24592 |
| Foley Street, Fitzrovia                                  |  24563 |
| Finlay Street, Fulham                                    |  24534 |
| Riverlight North, Nine Elms                              |  24518 |
| Regency Street, Westminster                              |  24437 |
| Shouldham Street, Marylebone                             |  24433 |
| Doddington Grove, Kennington                             |  24359 |
| Kensington Gore, Knightsbridge                           |  24342 |
| Park Lane, Mayfair                                       |  24279 |
| Lollard Street, Vauxhall                                 |  24227 |
| Tyers Gate, Bermondsey                                   |  24164 |
| Surrey Lane, Battersea                                   |  24117 |
| Prince of Wales Drive, Battersea Park                    |  24111 |
| World's End Place, West Chelsea                          |  24035 |
| Hartington Road, Stockwell                               |  24006 |
| Lambeth North Station, Waterloo                          |  23995 |
| Emperor's Gate, South Kensington                         |  23877 |
| High Holborn , Covent Garden                             |  23865 |
| Bridge Avenue, Hammersmith                               |  23852 |
| Gloucester Road (Central), South Kensington              |  23836 |
| Falkirk Street, Hoxton                                   |  23804 |
| Simpson Street, Clapham Junction                         |  23800 |
| Sandilands Road, Walham Green                            |  23786 |
| New Kent Road, The Borough                               |  23661 |
| Battersea Church Road, Battersea                         |  23656 |
| Michael Road, Walham Green                               |  23589 |
| Hampton Street, Walworth                                 |  23576 |
| Shoreditch Court, Haggerston                             |  23558 |
| The Guildhall, Guildhall                                 |  23532 |
| Rathbone Street, Fitzrovia                               |  23530 |
| St. John's Wood Road, St. John's Wood                    |  23503 |
| Claverton Street, Pimlico                                |  23255 |
| Claremont Square, Angel                                  |  23247 |
| St. Mark's Road, North Kensington                        |  23246 |
| Southwark Station 1, Southwark                           |  23216 |
| Queen Mary's, Mile End                                   |  23164 |
| Upper Bank Street, Canary Wharf                          |  23159 |
| Goswell Road (City Uni), Finsbury                        |  23133 |
| King Edward Street, St Pauls                             |  22956 |
| Gloucester Terrace, Bayswater                            |  22843 |
| Carnegie Street, King's Cross                            |  22782 |
| The Vale, West Chelsea                                   |  22743 |
| Ormonde Gate, Chelsea                                    |  22705 |
| Bancroft Road, Bethnal Green                             |  22661 |
| Murray Grove , Hoxton                                    |  22653 |
| Strata, Elephant & Castle                                |  22646 |
| Austin Road, Battersea Park                              |  22631 |
| Cleveland Way, Stepney                                   |  22589 |
| George Place Mews, Marylebone                            |  22558 |
| Woodstock Street, Mayfair                                |  22436 |
| Fulham Park Road, Fulham                                 |  22356 |
| Union Grove, Wandsworth Road                             |  22271 |
| Pembridge Villas, Notting Hill                           |  22192 |
| Mile End Stadium, Mile End                               |  22175 |
| Vincent Street, Pimlico                                  |  22120 |
| Warwick Road, Olympia                                    |  22111 |
| Trebovir Road, Earl's Court                              |  22086 |
| Chancery Lane, Holborn                                   |  22022 |
| Aintree Street, Fulham                                   |  21915 |
| Waterloo Bridge, South Bank                              |  21903 |
| Charlotte Terrace, Angel                                 |  21883 |
| Walnut Tree Walk, Vauxhall                               |  21846 |
| Holland Park, Kensington                                 |  21801 |
| Mile End Park Leisure Centre, Mile End                   |  21799 |
| South Park, Sands End                                    |  21764 |
| Royal Avenue 1, Chelsea                                  |  21761 |
| Cleveland Gardens, Bayswater                             |  21754 |
| William Morris Way, Sands End                            |  21747 |
| Bishop's Avenue, Fulham                                  |  21647 |
| Wandsworth Rd, Isley Court, Wandsworth Road              |  21610 |
| London Zoo,  The Regent's Park                           |  21583 |
| Hoxton Street, Hoxton                                    |  21503 |
| Nevern Place, Earl's Court                               |  21477 |
| Imperial Road, Sands End                                 |  21473 |
| Hortensia Road, West Brompton                            |  21440 |
| Thorndike Close, West Chelsea                            |  21389 |
| Curzon Street, Mayfair                                   |  21263 |
| Clifford Street, Mayfair                                 |  21246 |
| Brook Green South, Brook Green                           |  21139 |
| Finnis Street, Bethnal Green                             |  21116 |
| Risinghill Street, Angel                                 |  21041 |
| Humbolt Road, Fulham                                     |  21025 |
| Pitfield Street Central, Hoxton                          |  20969 |
| Osiers Road, Wandsworth                                  |  20940 |
| Lower Thames Street, Monument                            |  20919 |
| Vereker Road North, West Kensington                      |  20916 |
| Vauxhall Bridge , Pimlico                                |  20903 |
| Richmond Way, Shepherd's Bush                            |  20799 |
| Antill Road, Mile End                                    |  20791 |
| River Street , Clerkenwell                               |  20708 |
| Colombo Street, Southwark                                |  20702 |
| Queensbridge Road, Haggerston                            |  20616 |
| Silverthorne Road, Battersea                             |  20502 |
| Ashley Crescent, Battersea                               |  20463 |
| Collingham Gardens, Earl's Court                         |  20441 |
| Palissy Street, Shoreditch                               |  20424 |
| Wapping High Street, Wapping                             |  20386 |
| Tate Modern, Bankside                                    |  20356 |
| Battersea Park Road, Nine Elms                           |  20339 |
| South Lambeth Road, Vauxhall                             |  20294 |
| Millennium Hotel, Mayfair                                |  20275 |
| St. John's Road, Clapham Junction                        |  20244 |
| Eversholt Street , Camden Town                           |  20126 |
| West Kensington Station, West Kensington                 |  20039 |
| Chelsea Green, Chelsea                                   |  20037 |
| Whiston Road, Haggerston                                 |  19997 |
| Houndsditch, Aldgate                                     |  19933 |
| Rochester Row, Westminster                               |  19900 |
| Argyll Road, Kensington                                  |  19860 |
| Longford Street, The Regent's Park                       |  19816 |
| Good's Way, King's Cross                                 |  19787 |
| Lots Road, West Chelsea                                  |  19755 |
| Clifton Street, Shoreditch                               |  19752 |
| Everington Street, Fulham                                |  19613 |
| Beryl Road, Hammersmith                                  |  19496 |
| All Saints' Road, Portobello                             |  19441 |
| Lambeth Road, Vauxhall                                   |  19433 |
| Webber Street , Southwark                                |  19411 |
| Aberdeen Place, St. John's Wood                          |  19391 |
| Bruton Street, Mayfair                                   |  19359 |
| Abingdon Villas, Kensington                              |  19328 |
| Mostyn Grove, Bow                                        |  19320 |
| Eel Brook Common, Walham Green                           |  19312 |
| Disraeli Road, Putney                                    |  19294 |
| Belgrave Square, Belgravia                               |  19270 |
| St. John's Wood Church, The Regent's Park                |  19215 |
| Harford Street, Mile End                                 |  19213 |
| Crabtree Lane, Fulham                                    |  19178 |
| Alfreda Street, Battersea Park                           |  19176 |
| Borough Road, Elephant & Castle                          |  19074 |
| Charing Cross Hospital, Hammersmith                      |  19059 |
| Ram Street, Wandsworth                                   |  19037 |
| Wellington Road, St. John's Wood                         |  19014 |
| Stanhope Gate, Mayfair                                   |  19000 |
| Bourne Street, Belgravia                                 |  18992 |
| Bow Church Station, Bow                                  |  18935 |
| Upcerne Road, West Chelsea                               |  18887 |
| Harrowby Street, Marylebone                              |  18843 |
| Cadogan Place, Knightsbridge                             |  18836 |
| Bishop's Bridge Road East, Bayswater                     |  18815 |
| Ravenscourt Park Station, Hammersmith                    |  18786 |
| Central House, Aldgate                                   |  18766 |
| Breams Buildings, Holborn                                |  18695 |
| Guildhouse Street, Victoria                              |  18674 |
| Star Road, West Kensington                               |  18654 |
| Grosvenor Road, Pimlico                                  |  18625 |
| Sumner Place, South Kensington                           |  18531 |
| Belford House, Haggerston                                |  18531 |
| Irene Road, Parsons Green                                |  18416 |
| Montgomery Square, Canary Wharf                          |  18399 |
| Kennington Lane Tesco, Vauxhall                          |  18349 |
| Rectory Square, Stepney                                  |  18338 |
| LSBU (Borough Road), Elephant & Castle                   |  18314 |
| Lambeth Palace Road, Waterloo                            |  18298 |
| Hollybush Gardens, Bethnal Green                         |  18297 |
| Kennington Station, Kennington                           |  18210 |
| Shepherd's Bush Road North, Shepherd's Bush              |  18121 |
| Ansell House, Stepney                                    |  18115 |
| Vauxhall Walk, Vauxhall                                  |  18022 |
| Kennington Road Post Office, Oval                        |  18020 |
| Melton Street, Euston                                    |  18016 |
| Buxton Street 1, Shoreditch                              |  17931 |
| Princedale Road , Holland Park                           |  17914 |
| Charlbert Street, St. John's Wood                        |  17888 |
| Westfield Southern Terrace ,Shepherd's Bush              |  17866 |
| Felsham Road, Putney                                     |  17782 |
| Sancroft Street, Vauxhall                                |  17701 |
| Lavington Street, Bankside                               |  17636 |
| Aylward Street, Stepney                                  |  17636 |
| Ashmole Estate, Oval                                     |  17608 |
| Ford Road, Old Ford                                      |  17595 |
| Hermitage Court, Wapping                                 |  17499 |
| Rossmore Road, Marylebone                                |  17487 |
| Gaywood  Street, Elephant & Castle                       |  17466 |
| Imperial Wharf Station, Sands End                        |  17350 |
| Lee Valley VeloPark, Queen Elizabeth Olympic Park        |  17282 |
| Dorothy Road, Clapham Junction                           |  17253 |
| BBC White City, White City                               |  17151 |
| Princes Square, Bayswater                                |  17008 |
| Eaton Square (South), Belgravia                          |  16894 |
| Lindfield Street, Poplar                                 |  16865 |
| Blackfriars Road, Southwark                              |  16746 |
| The Vale, Chelsea                                        |  16634 |
| Lisson Grove, St. John's Wood                            |  16557 |
| Burdett Road, Mile End                                   |  16524 |
| Grafton Street, Mayfair                                  |  16447 |
| Carey Street, Holborn                                    |  16442 |
| Birkenhead Street, King's Cross                          |  16320 |
| Wendon Street, Old Ford                                  |  16266 |
| Hewison Street, Old Ford                                 |  16251 |
| New North Road 2, Hoxton                                 |  16189 |
| Vincent Square, Westminster                              |  16140 |
| Frampton Street, Paddington                              |  16124 |
| Houghton Street, Strand                                  |  15986 |
| Gwendwr Road, West Kensington                            |  15935 |
| Kensington Olympia Station, Olympia                      |  15876 |
| Sail Street, Vauxhall                                    |  15854 |
| Courland Grove, Wandsworth Road                          |  15844 |
| Cotton Garden Estate, Kennington                         |  15795 |
| Sopwith Way, Battersea Park                              |  15746 |
| Hammersmith Town Hall, Hammersmith                       |  15726 |
| Hansard Mews, Holland Park                               |  15647 |
| Greycoat Street , Westminster                            |  15634 |
| South Parade, Chelsea                                    |  15630 |
| East Village, Queen Elizabeth Olympic Park               |  15602 |
| Sheepcote Lane, Battersea                                |  15340 |
| Calshot Street , King's Cross                            |  15328 |
| Portugal Street, Holborn                                 |  15298 |
| Orbel Street, Battersea                                  |  15292 |
| Kensington Town Hall, Kensington                         |  15042 |
| Clarence Walk, Stockwell                                 |  14996 |
| South Audley Street, Mayfair                             |  14877 |
| Devons Road, Bow                                         |  14826 |
| Paddington Green Police Station, Paddington              |  14822 |
| Southerton Road, Hammersmith                             |  14810 |
| Manbre Road, Hammersmith                                 |  14761 |
| Vaughan Way, Wapping                                     |  14739 |
| Monier Road, Hackney Wick                                |  14708 |
| Vicarage Gate, Kensington                                |  14669 |
| Southern Grove, Bow                                      |  14660 |
| Victoria & Albert Museum, South Kensington               |  14599 |
| Chrisp Street Market, Poplar                             |  14579 |
| One Tower Bridge, Bermondsey                             |  14396 |
| Timber Lodge, Queen Elizabeth Olympic Park               |  14347 |
| Stainsby Road , Poplar                                   |  14339 |
| Blythe Road West, Shepherd's Bush                        |  14301 |
| Grosvenor Crescent, Belgravia                            |  14288 |
| Smugglers Way, Wandsworth                                |  14235 |
| Usk Road, Clapham Junction                               |  14235 |
| Farm Street, Mayfair                                     |  14153 |
| Clarkson Street, Bethnal Green                           |  14117 |
| Thurtle Road, Haggerston                                 |  14093 |
| Denyer Street, Knightsbridge                             |  13946 |
| Campden Hill Road, Notting Hill                          |  13876 |
| Sugden Road, Clapham                                     |  13823 |
| Marloes Road, Kensington                                 |  13817 |
| Neville Gill Close, Wandsworth                           |  13781 |
| East India DLR, Blackwall                                |  13668 |
| Little Brook Green, Brook Green                          |  13617 |
| Lightermans Road, Millwall                               |  13459 |
| Harper Road, The Borough                                 |  13456 |
| Cleaver Street, Kennington                               |  13433 |
| Hampstead Road (Cartmel), Euston                         |  13271 |
| Hawley Crescent, Camden Town                             |  13034 |
| Ranelagh Gardens, Fulham                                 |  13032 |
| Clapham Common North side, Clapham Common                |  12594 |
| Pennington Street, Wapping                               |  12524 |
| Penfold Street, Marylebone                               |  12489 |
| Broadley Terrace, Marylebone                             |  12477 |
| Dock Street, Wapping                                     |  12291 |
| Clarendon Road, Avondale                                 |  12262 |
| Stratford Station, Stratford                             |  12155 |
| New North Road 1, Hoxton                                 |  12115 |
| Elysium Place, Fulham                                    |  12112 |
| Napier Avenue, Millwall                                  |  12076 |
| Copper Box Arena, Queen Elizabeth Olympic Park           |  11941 |
| Putney Pier, Wandsworth                                  |  11935 |
| Russell Gardens, Olympia                                 |  11850 |
| Dickens Square, Borough                                  |  11683 |
| Aston Street, Stepney                                    |  11658 |
| Spindrift Avenue, Millwall                               |  11621 |
| Heron Quays DLR, Canary Wharf                            |  11597 |
| Selby Street, Whitechapel                                |  11409 |
| Stanley Grove, Battersea                                 |  11354 |
| Queen Street, Bank                                       |  11328 |
| Canton Street, Poplar                                    |  11279 |
| Northfields, Wandsworth                                  |  11244 |
| Abyssinia Close, Clapham Junction                        |  11184 |
| Halford Road, West Kensington                            |  11166 |
| Addison Road, Holland Park                               |  11116 |
| Plough Terrace, Clapham Junction                         |  10995 |
| Rodney Street, Angel                                     |  10958 |
| Walmer Road, Avondale                                    |  10850 |
| Greenberry Street, St.John's Wood                        |  10711 |
| Killick Street, King's Cross                             |  10704 |
| Culvert Road, Battersea                                  |  10668 |
| Upper Grosvenor Street, Mayfair                          |  10657 |
| Bradmead, Battersea Park                                 |  10643 |
| Westfield Ariel Way, White City                          |  10587 |
| Putney Rail Station, Putney                              |  10560 |
| St Martins Close, Camden Town                            |  10558 |
| Alma Road, Wandsworth                                    |  10533 |
| Imperial Wharf Station                                   |  10495 |
| Ackroyd Drive, Bow                                       |  10442 |
| Hibbert Street, Battersea                                |  10441 |
| Teversham Lane, Stockwell                                |  10408 |
| Montserrat Road , Putney                                 |  10391 |
| Cromer Street, Bloomsbury                                |  10372 |
| Clapham Common North Side, Clapham Common                |  10327 |
| King Edward Walk, Waterloo                               |  10252 |
| Clarges Street, Mayfair                                  |  10191 |
| Mallory Street, Marylebone                               |  10085 |
| Peterborough Road, Sands End                             |  10066 |
| Grenfell Road, Avondale                                  |   9961 |
| Paddington Green, Paddington                             |   9955 |
| Vauxhall Street, Vauxhall                                |   9929 |
| Heath Road, Battersea                                    |   9918 |
| Lincoln's Inn Fields, Holborn                            |   9916 |
| Geraldine Street, Elephant & Castle                      |   9905 |
| Abbotsbury Road, Holland Park                            |   9834 |
| Ilchester Place, Kensington                              |   9754 |
| Belvedere Road 1, South Bank                             |   9734 |
| Southwark Street, Bankside                               |   9720 |
| Westferry Circus, Canary Wharf                           |   9707 |
| Westminster Bridge Road, Elephant & Castle               |   9565 |
| Ossulston Street, Somers Town                            |   9416 |
| Alpha Grove, Millwall                                    |   9407 |
| Orsett Terrace, Bayswater                                |   9280 |
| Coram Street, Bloomsbury                                 |   9205 |
| Putney Bridge Road, East Putney                          |   9120 |
| Jubilee Crescent, Cubitt Town                            |   9117 |
| Rifle Place, Avondale                                    |   9112 |
| Hurlingham Park, Parsons Green                           |   9046 |
| Green Park Station, West End                             |   8850 |
| Lord's, St. John's Wood                                  |   8824 |
| Kings Gate House, Westminster                            |   8622 |
| Holden Street, Battersea                                 |   8579 |
| Riverlight South, Nine Elms                              |   8556 |
| Albert Square, Stockwell                                 |   8549 |
| Thessaly Road North, Wandsworth Road                     |   8467 |
| Harrington Square 1, Camden Town                         |   8326 |
| East Ferry Road, Cubitt Town                             |   8295 |
| Bevington Road West, North Kensington                    |   8247 |
| Ladbroke Grove Central, Ladbroke Grove                   |   8182 |
| Broomhouse Lane, Parsons Green                           |   8075 |
| Cantrell Road, Bow                                       |   8057 |
| Lancaster Drive, Blackwall                               |   7948 |
| Naval Row, Blackwall                                     |   7947 |
| Upper Richmond Road, Putney                              |   7729 |
| Bromley High Street, Bow                                 |   7720 |
| Newby Place, Poplar                                      |   7564 |
| St. Mary & St. Michael Church, Stepney                   |   7454 |
| Vereker Road, West Kensington                            |   7412 |
| Clapham Road, Lingham Street, Stockwell                  |   7326 |
| Freston Road, Avondale                                   |   7240 |
| Handyside Street, King's Cross                           |   7234 |
| Evesham Street, Avondale                                 |   7158 |
| Hibbert Street, Clapham Junction                         |   7145 |
| Oval Way, Vauxhall                                       |   6994 |
| Queensdale Road, Shepherd's Bush                         |   6994 |
| Mudchute DLR, Cubitt Town                                |   6974 |
| Belvedere Road 2, South Bank                             |   6809 |
| Sirdar Road, Avondale                                    |   6723 |
| Lansdowne Walk, Ladbroke Grove                           |   6697 |
| Ingrave Street, Clapham Junction                         |   6604 |
| Embankment (Horse Guards), Westminster                   |   6564 |
| Preston's Road, Cubitt Town                              |   6485 |
| Stewart's Road, Wandsworth Road                          |   6478 |
| Langdon Park, Poplar                                     |   6400 |
| Exhibition Road Museums, Knightsbridge                   |   6353 |
| Spencer Park, Wandsworth Common                          |   6324 |
| Mexfield Road, East Putney                               |   6302 |
| Stebondale Street, Cubitt Town                           |   5641 |
| Malmesbury Road, Bow                                     |   5537 |
| Oxford Road, Putney                                      |   5505 |
| Theobalds Road , Holborn                                 |   5491 |
| Limerston Street, Chelsea                                |   5488 |
| Santos Road, Wandsworth                                  |   5458 |
| Lodge Road, St. John's Wood                              |   5416 |
| Morie Street, Wandsworth                                 |   5388 |
| Spanish Road, Clapham Junction                           |   5329 |
| Park Road (Baker Street), Regent's Park                  |   5304 |
| Pitfield Street (North),Hoxton                           |   5156 |
| Columbia Road, Weavers                                   |   5027 |
| Esmond Street, Putney                                    |   5009 |
| Millharbour, Millwall                                    |   4873 |
| Drayton Gardens, Chelsea                                 |   4808 |
| Limburg Road, Clapham Junction                           |   4748 |
| Cadogan Gardens, Sloane Square                           |   4726 |
| Abingdon Green, Great College Street                     |   4669 |
| Nantes Close, Clapham Junction                           |   4600 |
| New Spring Gardens Walk, Vauxhall                        |   4277 |
| Merchant Street, Bow                                     |   4253 |
| Queen Marys, Mile End                                    |   4171 |
| St. John's Park, Cubitt Town                             |   4036 |
| Manfred Road, East Putney                                |   3997 |
| Great Dover Street, Borough                              |   3928 |
| Edgware Road Station, Paddington                         |   3887 |
| Teviot Street, Poplar                                    |   3885 |
| Fishermans Walk West, Canary Wharf                       |   3809 |
| Millbank House, Pimlico                                  |   3799 |
| Cleveland Way, Bethnal Green                             |   3709 |
| Nantes Close, Wandsworth                                 |   3655 |
| Stockwell Roundabout, Stockwell                          |   3609 |
| Sidney Street, Stepney                                   |   3575 |
| Goldsmiths Row, Haggerston                               |   3560 |
| Fawcett Close, Clapham Junction                          |   3512 |
| Prince Albert Road, Regent's Park                        |   3465 |
| Walworth Road, Southwark                                 |   3452 |
| Hoxton Station, Haggerston                               |   3444 |
| Normand Park, Fulham                                     |   3430 |
| Here East North, Queen Elizabeth Olympic Park            |   3362 |
| New Road  2, Whitechapel                                 |   3342 |
| Aberfeldy Street, Poplar                                 |   3332 |
| Bethnal Green Gardens, Bethnal Green                     |   3323 |
| St Katharines Way, Tower                                 |   3323 |
| The Tennis Courts, Regent's Park                         |   3303 |
| Grant Road West, Clapham Junction                        |   3299 |
| Albany Street, Regent's Park                             |   3242 |
| Austin Road, Battersea                                   |   3184 |
| Fawcett Close, Battersea                                 |   3175 |
| London Zoo Car Park, Regent's Park                       |   3122 |
| Gun Makers Lane, Old Ford                                |   3035 |
| Colet Gardens, Hammersmith                               |   3030 |
| Victory place, Walworth                                  |   2834 |
| Longford Street, Regent's Park                           |   2776 |
| Castalia Square, Cubitt Town                             |   2751 |
| Thornfield House, Poplar                                 |   2747 |
| Simpson Street, Battersea                                |   2585 |
| Altab Ali Park, Whitechapel                              |   2570 |
| London Zoo, Regents Park                                 |   2391 |
| Kingsway, Covent Garden                                  |   2351 |
| New Kent Road, Borough                                   |   2236 |
| Lansdowne Way Bus Garage, Stockwell                      |   2232 |
| Strata, Southwark                                        |   2223 |
| Churchill Place, Canary Wharf                            |   2208 |
| St. John's Wood Church, Regent's Park                    |   2034 |
| Bevington Road, North Kensington                         |   1990 |
| Clapham Common Northside, Clapham Common                 |   1954 |
| Hansard Mews, Shepherds Bush                             |   1898 |
| South Quay East, Canary Wharf                            |   1894 |
| Killick Street, Kings Cross                              |   1881 |
| Bradmead, Nine Elms                                      |   1792 |
| Battersea Power Station, Battersea Park                  |   1743 |
| Grant Road Central, Clapham Junction                     |   1716 |
| Waterloo Roundabout, Waterloo                            |   1678 |
| Harper Road, Borough                                     |   1660 |
| Sugden Road, Battersea                                   |   1554 |
| St Martin's Close, Camden Town                           |   1475 |
| Russell Gardens, Holland Park                            |   1463 |
| Victoria and Albert Museum, Cromwell Road                |   1451 |
| Walmer Road, Notting Hill                                |   1422 |
| Clarges Street, West End                                 |   1416 |
| Usk Road, Battersea                                      |   1222 |
| Thessaly Road North, Nine Elms                           |   1069 |
| Bromley High Street, Bromley                             |    985 |
| Upper Richmond Road, East Putney                         |    978 |
| Harrington Square, Camden Town                           |    911 |
| Ladbroke Grove Central, Notting Hill                     |    896 |
| Oval Way, Lambeth                                        |    870 |
| Lansdowne Walk, Notting Hill                             |    827 |
| St. Mary and St. Michael Church, Stepney                 |    820 |
| Central House, Whitechapel                               |    817 |
| Halford Road, Fulham                                     |    785 |
| Westfield Eastern Access Road, Shepherd's Bush           |    778 |
| Victory Place, Walworth                                  |    727 |
| South Quay West, Canary Wharf                            |    685 |
| Spanish Road, Wandsworth                                 |    666 |
| Stewart's Road, Nine Elms                                |    663 |
| Ingrave Street, Battersea                                |    635 |
| Limburg Road, Clapham Common                             |    466 |
| St John's Park, Cubitt Town                              |    449 |
| Mechanical Workshop Penton                               |    381 |
| Coborn Street, Mile End                                  |    340 |
| Here East South, Queen Elizabeth Olympic Park            |    267 |
| Pop Up Dock 1                                            |     67 |
| Contact Centre, Southbury House                          |     60 |
| Monier Road, Newham                                      |     34 |
| Pop Up Dock 2                                            |     31 |
| Monier Road                                              |     11 |
| Blackfriars road, Southwark                              |      5 |
| LSP2                                                     |      4 |
| PENTON STREET COMMS TEST TERMINAL _ CONTACT MATT McNULTY |      1 |
| LSP1                                                     |      1 |
+----------------------------------------------------------+--------+
881 rows in set (0.31 sec)

mysql>
```



```sql
mysql> SELECT * FROM london2;
+----------------------------------------------------------+--------+
| end_station_name                                         | num    |
+----------------------------------------------------------+--------+
| end_station_name                                         |      0 |
| Belgrove Street , King's Cross                           | 231802 |
| Hyde Park Corner, Hyde Park                              | 215038 |
| Waterloo Station 3, Waterloo                             | 193200 |
| Albert Gate, Hyde Park                                   | 157943 |
| Hop Exchange, The Borough                                | 156964 |
| Black Lion Gate, Kensington Gardens                      | 156020 |
| Waterloo Station 1, Waterloo                             | 141733 |
| Wormwood Street, Liverpool Street                        | 129376 |
| Brushfield Street, Liverpool Street                      | 120659 |
| Finsbury Circus, Liverpool Street                        | 116011 |
| Newgate Street , St. Paul's                              | 108223 |
| Triangle Car Park, Hyde Park                             | 107372 |
| Wellington Arch, Hyde Park                               | 105729 |
| Craven Street, Strand                                    | 104457 |
| Bethnal Green Road, Shoreditch                           | 100590 |
| Duke Street Hill, London Bridge                          |  99851 |
| Regent's Row , Haggerston                                |  98284 |
| Holborn Circus, Holborn                                  |  95902 |
| William IV Street, Strand                                |  94806 |
| Soho Square , Soho                                       |  93536 |
| Old Street Station, St. Luke's                           |  93010 |
| Palace Gate, Kensington Gardens                          |  92638 |
| Queen Street 1, Bank                                     |  90407 |
| Crosswall, Tower                                         |  90080 |
| Cheapside, Bank                                          |  89211 |
| St. James's Square, St. James's                          |  87991 |
| Bankside Mix, Bankside                                   |  86974 |
| Speakers' Corner 1, Hyde Park                            |  85884 |
| Exhibition Road, Knightsbridge                           |  85844 |
| Moorfields, Moorgate                                     |  85500 |
| Storey's Gate, Westminster                               |  85278 |
| Tooley Street, Bermondsey                                |  84807 |
| Whitehall Place, Strand                                  |  84038 |
| Shoreditch High Street, Shoreditch                       |  83212 |
| Sun Street, Liverpool Street                             |  80180 |
| South Kensington Station, South Kensington               |  78592 |
| Green Park Station, Mayfair                              |  77291 |
| Serpentine Car Park, Hyde Park                           |  77053 |
| Tower Gardens , Tower                                    |  76688 |
| Little Argyll Street, West End                           |  76558 |
| Great Tower Street, Monument                             |  75749 |
| Drury Lane, Covent Garden                                |  74456 |
| Malet Street, Bloomsbury                                 |  72148 |
| Kennington Lane Rail Bridge, Vauxhall                    |  71822 |
| Aldersgate Street, Barbican                              |  71770 |
| Stonecutter Street, Holborn                              |  70980 |
| Tavistock Street, Covent Garden                          |  69698 |
| Moor Street, Soho                                        |  69640 |
| Queen's Gate, Kensington Gardens                         |  68563 |
| Leonard Circus , Shoreditch                              |  67862 |
| Baylis Road, Waterloo                                    |  67744 |
| Wardour Street, Soho                                     |  67002 |
| Berry Street, Clerkenwell                                |  66448 |
| Vauxhall Cross, Vauxhall                                 |  66398 |
| Speakers' Corner 2, Hyde Park                            |  66371 |
| Great Marlborough Street, Soho                           |  65752 |
| Godliman Street, St. Paul's                              |  65433 |
| Jubilee Gardens, South Bank                              |  64873 |
| Nesham Street, Wapping                                   |  64824 |
| Islington Green, Angel                                   |  64656 |
| Queen Street 2, Bank                                     |  63302 |
| Ashley Place, Victoria                                   |  63188 |
| Fulham Broadway, Walham Green                            |  62746 |
| Hatton Wall, Holborn                                     |  62386 |
| Eccleston Place, Victoria                                |  62203 |
| Red Lion Street, Holborn                                 |  61865 |
| Derry Street, Kensington                                 |  61284 |
| West Smithfield Rotunda, Farringdon                      |  61141 |
| Bank of England Museum, Bank                             |  61073 |
| Leman Street, Aldgate                                    |  61014 |
| Notting Hill Gate Station, Notting Hill                  |  60767 |
| Park Lane , Hyde Park                                    |  60638 |
| Pall Mall East, West End                                 |  60032 |
| Finsbury Leisure Centre, St. Luke's                      |  59876 |
| British Museum, Bloomsbury                               |  59597 |
| Winsland Street, Paddington                              |  59455 |
| Gloucester Road Station, South Kensington                |  58950 |
| Waterloo Station 2, Waterloo                             |  58696 |
| Bunhill Row, Moorgate                                    |  58491 |
| Clerkenwell Green, Clerkenwell                           |  58401 |
| Howick Place, Westminster                                |  57623 |
| Dunston Road , Haggerston                                |  56633 |
| Bath Street, St. Luke's                                  |  56352 |
| Aquatic Centre, Queen Elizabeth Olympic Park             |  55979 |
| Bermondsey Street, Bermondsey                            |  55973 |
| Northumberland Avenue, Strand                            |  55425 |
| Wright's Lane, Kensington                                |  54378 |
| St. Bride Street, Holborn                                |  53954 |
| Southampton Street, Strand                               |  53947 |
| Endsleigh Gardens, Euston                                |  53598 |
| Flood Street, Chelsea                                    |  53365 |
| Warren Street Station, Euston                            |  53032 |
| Queen Victoria Street, St. Paul's                        |  52844 |
| Queen's Circus, Battersea Park                           |  52208 |
| Braham Street, Aldgate                                   |  51354 |
| Somerset House, Strand                                   |  51341 |
| Eagle Wharf Road, Hoxton                                 |  51235 |
| Imperial College, Knightsbridge                          |  50828 |
| Curlew Street, Shad Thames                               |  50670 |
| Belvedere Road, South Bank                               |  50352 |
| Sedding Street, Sloane Square                            |  50214 |
| Elizabeth Bridge, Victoria                               |  49883 |
| Museum of London, Barbican                               |  49852 |
| Poured Lines, Bankside                                   |  49748 |
| Abingdon Green, Westminster                              |  49631 |
| Limerston Street, West Chelsea                           |  49413 |
| Tysoe Street, Clerkenwell                                |  49317 |
| Hinde Street, Marylebone                                 |  49258 |
| Bouverie Street, Temple                                  |  49216 |
| Bayswater Road, Hyde Park                                |  48387 |
| Wellington Street , Strand                               |  48190 |
| Abbey Orchard Street, Westminster                        |  47838 |
| Stamford Street, South Bank                              |  47759 |
| Bayley Street , Bloomsbury                               |  47706 |
| Wells Street, Fitzrovia                                  |  47225 |
| Russell Square Station, Bloomsbury                       |  47176 |
| Belgrave Road, Victoria                                  |  46821 |
| Waterloo Place, St. James's                              |  46760 |
| Liverpool Road (N1 Centre), Angel                        |  46624 |
| Cumberland Gate, Hyde Park                               |  46551 |
| Old Quebec Street, Marylebone                            |  46473 |
| Queen Mother Sports Centre, Victoria                     |  46465 |
| Montpelier Street, Knightsbridge                         |  46169 |
| Southwark Station 2, Southwark                           |  45872 |
| South Wharf Road, Paddington                             |  45696 |
| Ethelburga Estate, Battersea Park                        |  45563 |
| Strand, Strand                                           |  45471 |
| Wren Street, Holborn                                     |  45149 |
| Falcon Road, Clapham Junction                            |  44767 |
| Milroy Walk, South Bank                                  |  44729 |
| Exhibition Road Museums, South Kensington                |  44661 |
| New Inn Yard, Shoreditch                                 |  44597 |
| Fournier Street, Whitechapel                             |  44333 |
| Panton Street, West End                                  |  43898 |
| Hatton Garden, Holborn                                   |  43790 |
| Sadlers Sports Centre, Finsbury                          |  43733 |
| Great Russell Street, Bloomsbury                         |  43700 |
| Frith Street, Soho                                       |  43527 |
| Westbridge Road, Battersea                               |  43503 |
| Tanner Street, Bermondsey                                |  43503 |
| Greenland Road, Camden Town                              |  43472 |
| Guilford Street , Bloomsbury                             |  43432 |
| Marylebone Lane, Marylebone                              |  43159 |
| Binfield Road, Stockwell                                 |  42991 |
| Bolsover Street, Fitzrovia                               |  42986 |
| Tallis Street, Temple                                    |  42770 |
| Natural History Museum, South Kensington                 |  42602 |
| Westferry DLR, Limehouse                                 |  42541 |
| Golden Square, Soho                                      |  42540 |
| Doric Way , Somers Town                                  |  42521 |
| Howland Street, Fitzrovia                                |  42497 |
| Castlehaven Road, Camden Town                            |  42378 |
| Finsbury Square , Moorgate                               |  42261 |
| Buckingham Gate, Westminster                             |  42156 |
| Pritchard's Road, Bethnal Green                          |  41977 |
| New Globe Walk, Bankside                                 |  41725 |
| Graham Street, Angel                                     |  41716 |
| Flamborough Street, Limehouse                            |  41704 |
| Snow Hill, Farringdon                                    |  41658 |
| Kennington Cross, Kennington                             |  41531 |
| Pott Street, Bethnal Green                               |  41117 |
| Kensington Church Street, Kensington                     |  40817 |
| Bonny Street, Camden Town                                |  40535 |
| Cardinal Place, Victoria                                 |  40499 |
| Park Road (Baker Street), The Regent's Park              |  40468 |
| Sloane Avenue, Knightsbridge                             |  40162 |
| Northington Street , Holborn                             |  40114 |
| Alfred Place, Bloomsbury                                 |  39949 |
| Great Titchfield Street, Fitzrovia                       |  39911 |
| Golden Lane, Barbican                                    |  39696 |
| Seville Street, Knightsbridge                            |  39565 |
| Brick Lane Market, Shoreditch                            |  39429 |
| Norton Folgate, Liverpool Street                         |  39415 |
| Parsons Green Station, Parsons Green                     |  39378 |
| Theobald's Road , Holborn                                |  39322 |
| Broadcasting House, Marylebone                           |  39286 |
| Devonshire Square, Liverpool Street                      |  39165 |
| Drayton Gardens, West Chelsea                            |  39139 |
| Granby Street, Shoreditch                                |  39089 |
| Hampstead Road, Euston                                   |  39031 |
| Victoria Park Road, Hackney Central                      |  38855 |
| New Fetter Lane, Holborn                                 |  38808 |
| St. Mary Axe, Aldgate                                    |  38806 |
| Woodstock Grove, Shepherd's Bush                         |  38771 |
| Royal London Hospital, Whitechapel                       |  38650 |
| Chapel Place, Marylebone                                 |  38586 |
| Warwick Avenue Station, Maida Vale                       |  38551 |
| Southampton Place, Holborn                               |  38459 |
| Rampayne Street, Pimlico                                 |  38454 |
| Watney Street, Shadwell                                  |  38454 |
| All Saints Church, Portobello                            |  38442 |
| York Hall, Bethnal Green                                 |  38285 |
| Whiteley's, Bayswater                                    |  38052 |
| Monument Street, Monument                                |  37850 |
| Devonshire Terrace, Bayswater                            |  37815 |
| Erin Close, Walham Green                                 |  37711 |
| Horseferry Road, Westminster                             |  37701 |
| Chadwell Street, Angel                                   |  37323 |
| Cadogan Gardens, Chelsea                                 |  37238 |
| Broadwick Street, Soho                                   |  37213 |
| Christopher Street, Liverpool Street                     |  36832 |
| Jubilee Plaza, Canary Wharf                              |  36804 |
| Boston Place, Marylebone                                 |  36755 |
| Smith Square, Westminster                                |  36737 |
| Sackville Street, Mayfair                                |  36693 |
| St. Chad's Street, King's Cross                          |  36666 |
| Page Street, Westminster                                 |  36530 |
| Kingsway Southbound, Strand                              |  36514 |
| Bramham Gardens, Earl's Court                            |  36438 |
| George Street, Marylebone                                |  36299 |
| Phillimore Gardens, Kensington                           |  36293 |
| Earnshaw Street , Covent Garden                          |  36200 |
| Appold Street, Liverpool Street                          |  36090 |
| Dorset Square, Marylebone                                |  36011 |
| Porchester Place, Paddington                             |  35666 |
| Cartwright Gardens , Bloomsbury                          |  35609 |
| Portland Place, Marylebone                               |  35560 |
| Gloucester Road (North), Kensington                      |  35383 |
| Stephendale Road, Sands End                              |  35367 |
| Parson's Green , Parson's Green                          |  35353 |
| Harriet Street, Knightsbridge                            |  35291 |
| Holy Trinity Brompton, Knightsbridge                     |  35197 |
| Embankment (Savoy), Strand                               |  35193 |
| St. George's Square, Pimlico                             |  35104 |
| Brunswick Square, Bloomsbury                             |  35007 |
| Fire Brigade Pier, Vauxhall                              |  34986 |
| Shoreditch Park, Hoxton                                  |  34858 |
| Lancaster Gate , Bayswater                               |  34572 |
| Cadogan Close, Victoria Park                             |  34317 |
| Old Montague Street, Whitechapel                         |  34301 |
| Taviton Street, Bloomsbury                               |  34272 |
| Concert Hall Approach 2, South Bank                      |  34149 |
| Bricklayers Arms, Borough                                |  34095 |
| Roscoe Street, St. Luke's                                |  34074 |
| Sardinia Street, Holborn                                 |  34032 |
| Walworth Road, Elephant & Castle                         |  33949 |
| Haggerston Road, Haggerston                              |  33890 |
| Millbank Tower, Pimlico                                  |  33878 |
| Grosvenor Square, Mayfair                                |  33777 |
| Empire Square, The Borough                               |  33770 |
| Palace Gardens Terrace, Notting Hill                     |  33740 |
| Drummond Street , Euston                                 |  33631 |
| Shadwell Station, Shadwell                               |  33594 |
| Charlotte Street, Fitzrovia                              |  33591 |
| Waterloo Road, South Bank                                |  33557 |
| Ada Street, Hackney Central                              |  33517 |
| LMU Commercial Road, Whitechapel                         |  33466 |
| Stepney Green Station, Stepney                           |  33457 |
| Turquoise Island, Notting Hill                           |  33410 |
| Commercial Street, Shoreditch                            |  33332 |
| The Green Bridge, Mile End                               |  33147 |
| Prince Consort Road, Knightsbridge                       |  33106 |
| Bell Lane, Liverpool Street                              |  33058 |
| Torrens Street, Angel                                    |  32952 |
| Barbican Centre, Barbican                                |  32842 |
| Hammersmith Road, Hammersmith                            |  32735 |
| Beaumont Street, Marylebone                              |  32710 |
| Tachbrook Street, Victoria                               |  32372 |
| Fisherman's Walk West, Canary Wharf                      |  32307 |
| Pitfield Street North,Hoxton                             |  32241 |
| Wapping Lane, Wapping                                    |  32221 |
| Farringdon Lane, Clerkenwell                             |  32215 |
| Gloucester Street, Pimlico                               |  32201 |
| St. Martin's Street, West End                            |  32196 |
| Barons Court Station, West Kensington                    |  32104 |
| Queen's Gate (South), South Kensington                   |  32024 |
| Fore Street, Guildhall                                   |  31933 |
| Old Brompton Road, South Kensington                      |  31807 |
| Baldwin Street, St. Luke's                               |  31800 |
| Tavistock Place, Bloomsbury                              |  31769 |
| Wenlock Road , Hoxton                                    |  31699 |
| Lower Marsh, Waterloo                                    |  31681 |
| Jewry Street, Aldgate                                    |  31679 |
| Green Street, Mayfair                                    |  31487 |
| North Audley Street, Mayfair                             |  31485 |
| Vicarage Crescent, Battersea                             |  31484 |
| New Cavendish Street, Marylebone                         |  31484 |
| Albert Bridge Road, Battersea Park                       |  31350 |
| Twig Folly Bridge, Mile End                              |  31303 |
| Gower Place , Euston                                     |  31206 |
| Crawford Street, Marylebone                              |  31161 |
| Kennington Oval, Oval                                    |  31086 |
| Butler Place, Westminster                                |  31043 |
| Ontario Street, Elephant & Castle                        |  30890 |
| Crisp Road, Hammersmith                                  |  30758 |
| Columbia Road, Shoreditch                                |  30698 |
| Concert Hall Approach 1, South Bank                      |  30674 |
| Bury Place, Holborn                                      |  30634 |
| Ilchester Gardens, Bayswater                             |  30438 |
| Penywern Road, Earl's Court                              |  30412 |
| Hertford Road, De Beauvoir Town                          |  30267 |
| Portman Square, Marylebone                               |  30164 |
| Snowsfields, London Bridge                               |  29938 |
| Euston Road, Euston                                      |  29880 |
| Charles II Street, West End                              |  29879 |
| Newton Street, Covent Garden                             |  29732 |
| St. George Street, Mayfair                               |  29727 |
| Normand Park, West Kensington                            |  29725 |
| Manresa Road, Chelsea                                    |  29541 |
| Eastbourne Mews, Paddington                              |  29500 |
| Ebury Bridge, Pimlico                                    |  29475 |
| St. Peter's Terrace, Fulham                              |  29439 |
| The Tennis Courts, The Regent's Park                     |  29353 |
| Alderney Street, Pimlico                                 |  29229 |
| Caldwell Street, Stockwell                               |  29083 |
| Swan Street, The Borough                                 |  29047 |
| Union Street, The Borough                                |  28959 |
| Crinan Street, King's Cross                              |  28944 |
| Christian Street, Whitechapel                            |  28939 |
| Great Percy Street, Clerkenwell                          |  28867 |
| Bishop's Bridge Road West, Bayswater                     |  28720 |
| Royal Avenue 2, Chelsea                                  |  28708 |
| Parkway, Camden Town                                     |  28693 |
| Clifton Road, Maida Vale                                 |  28683 |
| Grant Road East, Clapham Junction                        |  28639 |
| Olympia Way, Olympia                                     |  28544 |
| Blenheim Crescent, Ladbroke Grove                        |  28487 |
| Bell Street , Marylebone                                 |  28475 |
| Borough High Street, The Borough                         |  28352 |
| Red Lion Square, Holborn                                 |  28293 |
| Reardon Street, Wapping                                  |  28287 |
| Westfield Library Corner, Shepherd's Bush                |  28248 |
| Queen's Gate (North), Kensington                         |  28200 |
| Old Ford Road, Bethnal Green                             |  28084 |
| Pont Street, Knightsbridge                               |  28083 |
| Wood Street, Guildhall                                   |  28068 |
| Chesilton Road, Fulham                                   |  27974 |
| Wandsworth Town Station, Wandsworth                      |  27955 |
| Southwick Street, Paddington                             |  27933 |
| Coomer Place, West Kensington                            |  27919 |
| St. Katharine's Way, Tower                               |  27915 |
| Prince Albert Road, The Regent's Park                    |  27829 |
| Scala Street, Fitzrovia                                  |  27801 |
| Fashion Street, Whitechapel                              |  27740 |
| Cephas Street, Bethnal Green                             |  27726 |
| East Road, Hoxton                                        |  27645 |
| Phene Street, Chelsea                                    |  27500 |
| Chelsea Bridge, Pimlico                                  |  27334 |
| City Road, Angel                                         |  27251 |
| Podium, Queen Elizabeth Olympic Park                     |  27225 |
| Saunders Ness Road, Cubitt Town                          |  27162 |
| Percival Street, Finsbury                                |  26954 |
| Lansdowne Road, Ladbroke Grove                           |  26935 |
| Southwark Station 1, Southwark                           |  26931 |
| Queen's Gate (Central), South Kensington                 |  26901 |
| Park Street, Bankside                                    |  26827 |
| Maclise Road, Olympia                                    |  26825 |
| Gloucester Avenue, Camden Town                           |  26757 |
| Northdown Street, King's Cross                           |  26713 |
| New Road 1 , Whitechapel                                 |  26686 |
| Archbishop's Park, Waterloo                              |  26686 |
| Danvers Street, West Chelsea                             |  26675 |
| Driffield Road, Old Ford                                 |  26631 |
| Long Lane , Bermondsey                                   |  26617 |
| Windsor Terrace, Hoxton                                  |  26579 |
| Westbourne Grove, Bayswater                              |  26426 |
| Clifford Street, Mayfair                                 |  26407 |
| The Guildhall, Guildhall                                 |  26365 |
| Goldsmith's Row, Haggerston                              |  26122 |
| St. Luke's Church, Chelsea                               |  26005 |
| Baker Street, Marylebone                                 |  25943 |
| Great Suffolk Street, The Borough                        |  25910 |
| Rodney Road , Walworth                                   |  25875 |
| Nutford Place, Marylebone                                |  25840 |
| Harrington Square 2, Camden Town                         |  25827 |
| Arlington Road, Camden Town                              |  25741 |
| Bedford Way, Bloomsbury                                  |  25704 |
| Albert Gardens, Stepney                                  |  25574 |
| Clinton Road, Mile End                                   |  25553 |
| Finsbury Library , Finsbury                              |  25534 |
| Putney Bridge Station, Fulham                            |  25464 |
| Gunmakers Lane, Old Ford                                 |  25403 |
| Philpot Street, Whitechapel                              |  25392 |
| Bridge Avenue, Hammersmith                               |  25345 |
| De Vere Gardens, Kensington                              |  25337 |
| High Holborn , Covent Garden                             |  25319 |
| Wellington Row, Bethnal Green                            |  25286 |
| Westminster University, Marylebone                       |  25244 |
| Knightsbridge, Hyde Park                                 |  25200 |
| Margery Street, Clerkenwell                              |  24959 |
| Sutton Street, Shadwell                                  |  24896 |
| Hereford Road, Bayswater                                 |  24891 |
| Globe Town Market, Bethnal Green                         |  24856 |
| Durant Street, Bethnal Green                             |  24835 |
| Edgware Road Station, Marylebone                         |  24784 |
| King Edward Street, St Pauls                             |  24768 |
| Kennington Road  , Vauxhall                              |  24750 |
| Foley Street, Fitzrovia                                  |  24653 |
| Curzon Street, Mayfair                                   |  24596 |
| Upper Bank Street, Canary Wharf                          |  24476 |
| Finlay Street, Fulham                                    |  24425 |
| Rainville Road, Hammersmith                              |  24361 |
| North Wharf Road, Paddington                             |  24342 |
| Great Dover Street, The Borough                          |  24316 |
| Greyhound Road, Hammersmith                              |  24313 |
| Pancras Road, King's Cross                               |  24278 |
| Seymour Place, Marylebone                                |  24255 |
| Bethnal Green Garden, Bethnal Green                      |  24230 |
| Royal College Street, Camden Town                        |  24222 |
| Ampton Street , Clerkenwell                              |  24170 |
| Black Prince Road, Vauxhall                              |  24156 |
| Bruton Street, Mayfair                                   |  24140 |
| Albert Embankment, Vauxhall                              |  24107 |
| Gloucester Road (Central), South Kensington              |  24099 |
| Furze Green, Bow                                         |  24041 |
| Tyers Gate, Bermondsey                                   |  24039 |
| Knaresborough Place, Earl's Court                        |  24025 |
| Simpson Street, Clapham Junction                         |  24023 |
| West Cromwell Road, Earl's Court                         |  24009 |
| Royal Avenue 1, Chelsea                                  |  23988 |
| Paddington Street, Marylebone                            |  23856 |
| Waterloo Bridge, South Bank                              |  23810 |
| Garnet Street, Shadwell                                  |  23798 |
| Warwick Square, Pimlico                                  |  23779 |
| Rathbone Street, Fitzrovia                               |  23751 |
| Eaton Square, Belgravia                                  |  23700 |
| Salmon Lane, Limehouse                                   |  23671 |
| Bow Road Station, Bow                                    |  23655 |
| World's End Place, West Chelsea                          |  23596 |
| Sandilands Road, Walham Green                            |  23543 |
| Lexham Gardens, Kensington                               |  23493 |
| Doddington Grove, Kennington                             |  23477 |
| London Zoo Car Park, The Regent's Park                   |  23468 |
| Macclesfield Rd, St Lukes                                |  23465 |
| Regency Street, Westminster                              |  23431 |
| Westbourne Park Road, Portobello                         |  23378 |
| Emperor's Gate, South Kensington                         |  23377 |
| Hartington Road, Stockwell                               |  23368 |
| Ormonde Gate, Chelsea                                    |  23283 |
| The Vale, West Chelsea                                   |  23245 |
| Grove End Road, St. John's Wood                          |  23232 |
| St. Mark's Road, North Kensington                        |  23225 |
| Harcourt Terrace, West Brompton                          |  23072 |
| Lambeth North Station, Waterloo                          |  23049 |
| Prince of Wales Drive, Battersea Park                    |  23040 |
| Shouldham Street, Marylebone                             |  22982 |
| Michael Road, Walham Green                               |  22954 |
| Park Lane, Mayfair                                       |  22953 |
| Hoxton Station, Hoxton                                   |  22893 |
| New Kent Road, The Borough                               |  22748 |
| Vincent Street, Pimlico                                  |  22707 |
| Jubilee Street, Stepney                                  |  22593 |
| Riverlight North, Nine Elms                              |  22548 |
| Woodstock Street, Mayfair                                |  22455 |
| Chancery Lane, Holborn                                   |  22410 |
| Rochester Row, Westminster                               |  22389 |
| Chepstow Villas, Notting Hill                            |  22382 |
| Hampton Street, Walworth                                 |  22350 |
| Warwick Road, Olympia                                    |  22300 |
| Watney Market, Stepney                                   |  22297 |
| Battersea Church Road, Battersea                         |  22272 |
| George Place Mews, Marylebone                            |  22044 |
| Kensington Gore, Knightsbridge                           |  21947 |
| Queen Mary's, Mile End                                   |  21934 |
| Falkirk Street, Hoxton                                   |  21897 |
| Cadogan Place, Knightsbridge                             |  21866 |
| Union Grove, Wandsworth Road                             |  21860 |
| Trebovir Road, Earl's Court                              |  21785 |
| Lower Thames Street, Monument                            |  21769 |
| Fulham Park Road, Fulham                                 |  21755 |
| Cleveland Way, Stepney                                   |  21743 |
| William Morris Way, Sands End                            |  21737 |
| Strata, Elephant & Castle                                |  21690 |
| Mile End Stadium, Mile End                               |  21663 |
| St. John's Road, Clapham Junction                        |  21660 |
| Houndsditch, Aldgate                                     |  21637 |
| Fanshaw Street, Hoxton                                   |  21576 |
| Lambeth Palace Road, Waterloo                            |  21520 |
| Clifton Street, Shoreditch                               |  21415 |
| Tate Modern, Bankside                                    |  21327 |
| Colombo Street, Southwark                                |  21314 |
| St. John Street, Finsbury                                |  21253 |
| Argyll Road, Kensington                                  |  21183 |
| Chelsea Green, Chelsea                                   |  21152 |
| Bishop's Avenue, Fulham                                  |  21150 |
| Millennium Hotel, Mayfair                                |  21056 |
| South Park, Sands End                                    |  21034 |
| Austin Road, Battersea Park                              |  20958 |
| Brook Green South, Brook Green                           |  20922 |
| Imperial Road, Sands End                                 |  20895 |
| Richmond Way, Shepherd's Bush                            |  20831 |
| Aintree Street, Fulham                                   |  20825 |
| Surrey Lane, Battersea                                   |  20682 |
| London Zoo,  The Regent's Park                           |  20674 |
| Gloucester Terrace, Bayswater                            |  20621 |
| Mile End Park Leisure Centre, Mile End                   |  20607 |
| Nevern Place, Earl's Court                               |  20585 |
| Pembridge Villas, Notting Hill                           |  20503 |
| Bancroft Road, Bethnal Green                             |  20447 |
| Wapping High Street, Wapping                             |  20442 |
| Osiers Road, Wandsworth                                  |  20426 |
| Ravenscourt Park Station, Hammersmith                    |  20334 |
| Goswell Road (City Uni), Finsbury                        |  20313 |
| Shoreditch Court, Haggerston                             |  20286 |
| Lollard Street, Vauxhall                                 |  20281 |
| Hortensia Road, West Brompton                            |  20229 |
| Albany Street, The Regent's Park                         |  20221 |
| Vereker Road North, West Kensington                      |  20204 |
| Kennington Lane Tesco, Vauxhall                          |  20122 |
| Humbolt Road, Fulham                                     |  19955 |
| Disraeli Road, Putney                                    |  19858 |
| Westfield Southern Terrace ,Shepherd's Bush              |  19850 |
| Ram Street, Wandsworth                                   |  19848 |
| Montgomery Square, Canary Wharf                          |  19774 |
| Cheshire Street, Bethnal Green                           |  19771 |
| Silverthorne Road, Battersea                             |  19739 |
| Grafton Street, Mayfair                                  |  19729 |
| West Kensington Station, West Kensington                 |  19447 |
| Wandsworth Rd, Isley Court, Wandsworth Road              |  19444 |
| Antill Road, Mile End                                    |  19436 |
| Good's Way, King's Cross                                 |  19435 |
| Cleveland Gardens, Bayswater                             |  19403 |
| Stanhope Gate, Mayfair                                   |  19401 |
| Guildhouse Street, Victoria                              |  19302 |
| Bourne Street, Belgravia                                 |  19269 |
| Vauxhall Walk, Vauxhall                                  |  19267 |
| Belgrave Square, Belgravia                               |  19243 |
| Borough Road, Elephant & Castle                          |  19190 |
| Hammersmith Town Hall, Hammersmith                       |  19187 |
| Lots Road, West Chelsea                                  |  19171 |
| South Lambeth Road, Vauxhall                             |  19168 |
| Abingdon Villas, Kensington                              |  19098 |
| Sumner Place, South Kensington                           |  19086 |
| Vauxhall Bridge , Pimlico                                |  19074 |
| Cloudesley Road, Angel                                   |  18929 |
| Felsham Road, Putney                                     |  18898 |
| Claverton Street, Pimlico                                |  18898 |
| Charing Cross Hospital, Hammersmith                      |  18848 |
| Finnis Street, Bethnal Green                             |  18796 |
| Thorndike Close, West Chelsea                            |  18754 |
| Collingham Gardens, Earl's Court                         |  18745 |
| Central House, Aldgate                                   |  18707 |
| Alfreda Street, Battersea Park                           |  18704 |
| Hoxton Street, Hoxton                                    |  18691 |
| Beryl Road, Hammersmith                                  |  18659 |
| LSBU (Borough Road), Elephant & Castle                   |  18628 |
| Breams Buildings, Holborn                                |  18605 |
| Kennington Station, Kennington                           |  18603 |
| All Saints' Road, Portobello                             |  18476 |
| Eel Brook Common, Walham Green                           |  18437 |
| Hollybush Gardens, Bethnal Green                         |  18432 |
| Ashley Crescent, Battersea                               |  18431 |
| Shepherd's Bush Road North, Shepherd's Bush              |  18361 |
| Bow Church Station, Bow                                  |  18350 |
| Lavington Street, Bankside                               |  18296 |
| Harford Street, Mile End                                 |  18273 |
| Crabtree Lane, Fulham                                    |  18249 |
| Hardwick Street, Clerkenwell                             |  18245 |
| Battersea Park Road, Nine Elms                           |  18244 |
| Upcerne Road, West Chelsea                               |  18192 |
| Everington Street, Fulham                                |  18136 |
| Eaton Square (South), Belgravia                          |  17998 |
| Webber Street , Southwark                                |  17983 |
| Eversholt Street , Camden Town                           |  17892 |
| BBC White City, White City                               |  17809 |
| Mostyn Grove, Bow                                        |  17798 |
| Walnut Tree Walk, Vauxhall                               |  17797 |
| Lambeth Road, Vauxhall                                   |  17678 |
| Palissy Street, Shoreditch                               |  17520 |
| Queensbridge Road, Haggerston                            |  17507 |
| Star Road, West Kensington                               |  17493 |
| Kennington Road Post Office, Oval                        |  17445 |
| Carnegie Street, King's Cross                            |  17439 |
| Imperial Wharf Station, Sands End                        |  17401 |
| Carey Street, Holborn                                    |  17371 |
| Melton Street, Euston                                    |  17357 |
| Ford Road, Old Ford                                      |  17312 |
| Irene Road, Parsons Green                                |  17260 |
| Murray Grove , Hoxton                                    |  17204 |
| Hansard Mews, Holland Park                               |  17150 |
| Houghton Street, Strand                                  |  17149 |
| The Vale, Chelsea                                        |  17124 |
| Gaywood  Street, Elephant & Castle                       |  16936 |
| Princedale Road , Holland Park                           |  16931 |
| Whiston Road, Haggerston                                 |  16925 |
| Harrowby Street, Marylebone                              |  16829 |
| Rectory Square, Stepney                                  |  16810 |
| Greycoat Street , Westminster                            |  16806 |
| Longford Street, The Regent's Park                       |  16640 |
| Blackfriars Road, Southwark                              |  16572 |
| Burdett Road, Mile End                                   |  16548 |
| Hermitage Court, Wapping                                 |  16546 |
| Wellington Road, St. John's Wood                         |  16530 |
| Birkenhead Street, King's Cross                          |  16520 |
| Holland Park, Kensington                                 |  16401 |
| Vincent Square, Westminster                              |  16352 |
| Ashmole Estate, Oval                                     |  16320 |
| Lindfield Street, Poplar                                 |  16300 |
| Charlotte Terrace, Angel                                 |  16297 |
| South Audley Street, Mayfair                             |  16227 |
| Pitfield Street Central, Hoxton                          |  16208 |
| Lee Valley VeloPark, Queen Elizabeth Olympic Park        |  16186 |
| South Parade, Chelsea                                    |  16171 |
| Ansell House, Stepney                                    |  16150 |
| St. John's Wood Road, St. John's Wood                    |  16012 |
| Wendon Street, Old Ford                                  |  15986 |
| Victoria & Albert Museum, South Kensington               |  15886 |
| Bishop's Bridge Road East, Bayswater                     |  15881 |
| Hewison Street, Old Ford                                 |  15862 |
| Courland Grove, Wandsworth Road                          |  15834 |
| Aylward Street, Stepney                                  |  15807 |
| Portugal Street, Holborn                                 |  15803 |
| Sopwith Way, Battersea Park                              |  15730 |
| Charlbert Street, St. John's Wood                        |  15728 |
| Aberdeen Place, St. John's Wood                          |  15709 |
| East Village, Queen Elizabeth Olympic Park               |  15674 |
| Dorothy Road, Clapham Junction                           |  15567 |
| Southerton Road, Hammersmith                             |  15467 |
| Buxton Street 1, Shoreditch                              |  15456 |
| Farm Street, Mayfair                                     |  15371 |
| Rossmore Road, Marylebone                                |  15307 |
| Vaughan Way, Wapping                                     |  15230 |
| Gwendwr Road, West Kensington                            |  15173 |
| Sancroft Street, Vauxhall                                |  15170 |
| Monier Road, Hackney Wick                                |  15092 |
| Clarence Walk, Stockwell                                 |  15018 |
| Denyer Street, Knightsbridge                             |  14952 |
| Princes Square, Bayswater                                |  14822 |
| Grosvenor Road, Pimlico                                  |  14791 |
| Chrisp Street Market, Poplar                             |  14774 |
| Putney Pier, Wandsworth                                  |  14715 |
| St. John's Wood Church, The Regent's Park                |  14632 |
| Devons Road, Bow                                         |  14613 |
| Little Brook Green, Brook Green                          |  14535 |
| Lightermans Road, Millwall                               |  14461 |
| Kensington Olympia Station, Olympia                      |  14457 |
| Sail Street, Vauxhall                                    |  14444 |
| Belford House, Haggerston                                |  14440 |
| One Tower Bridge, Bermondsey                             |  14428 |
| Sheepcote Lane, Battersea                                |  14403 |
| Smugglers Way, Wandsworth                                |  14365 |
| Kensington Town Hall, Kensington                         |  14360 |
| Neville Gill Close, Wandsworth                           |  14360 |
| River Street , Clerkenwell                               |  14341 |
| Hawley Crescent, Camden Town                             |  14136 |
| Manbre Road, Hammersmith                                 |  14099 |
| East India DLR, Blackwall                                |  14087 |
| Grosvenor Crescent, Belgravia                            |  14039 |
| Southern Grove, Bow                                      |  14004 |
| Timber Lodge, Queen Elizabeth Olympic Park               |  13915 |
| Cotton Garden Estate, Kennington                         |  13861 |
| Claremont Square, Angel                                  |  13836 |
| Blythe Road West, Shepherd's Bush                        |  13701 |
| Stainsby Road , Poplar                                   |  13668 |
| Orbel Street, Battersea                                  |  13523 |
| Calshot Street , King's Cross                            |  13424 |
| Usk Road, Clapham Junction                               |  13403 |
| Stratford Station, Stratford                             |  13315 |
| New North Road 2, Hoxton                                 |  13263 |
| Lisson Grove, St. John's Wood                            |  13202 |
| Cleaver Street, Kennington                               |  13162 |
| Marloes Road, Kensington                                 |  12993 |
| Paddington Green Police Station, Paddington              |  12969 |
| Clarkson Street, Bethnal Green                           |  12720 |
| Ranelagh Gardens, Fulham                                 |  12685 |
| Risinghill Street, Angel                                 |  12551 |
| Dock Street, Wapping                                     |  12448 |
| Harper Road, The Borough                                 |  12378 |
| Dickens Square, Borough                                  |  12246 |
| Heron Quays DLR, Canary Wharf                            |  12221 |
| Montserrat Road , Putney                                 |  12221 |
| Westfield Ariel Way, White City                          |  12188 |
| Abyssinia Close, Clapham Junction                        |  12164 |
| Frampton Street, Paddington                              |  12048 |
| Clapham Common North side, Clapham Common                |  11973 |
| Sugden Road, Clapham                                     |  11908 |
| Napier Avenue, Millwall                                  |  11878 |
| Spindrift Avenue, Millwall                               |  11876 |
| Queen Street, Bank                                       |  11784 |
| Copper Box Arena, Queen Elizabeth Olympic Park           |  11751 |
| Clarendon Road, Avondale                                 |  11618 |
| Vicarage Gate, Kensington                                |  11578 |
| Halford Road, West Kensington                            |  11298 |
| Elysium Place, Fulham                                    |  11251 |
| Vauxhall Street, Vauxhall                                |  11231 |
| Belvedere Road 1, South Bank                             |  11219 |
| Hampstead Road (Cartmel), Euston                         |  11206 |
| Aston Street, Stepney                                    |  11035 |
| Canton Street, Poplar                                    |  11004 |
| Russell Gardens, Olympia                                 |  10954 |
| Campden Hill Road, Notting Hill                          |  10951 |
| Penfold Street, Marylebone                               |  10944 |
| Thurtle Road, Haggerston                                 |  10839 |
| Clarges Street, Mayfair                                  |  10816 |
| Northfields, Wandsworth                                  |  10815 |
| Hibbert Street, Battersea                                |  10811 |
| Teversham Lane, Stockwell                                |  10761 |
| Pennington Street, Wapping                               |  10696 |
| Selby Street, Whitechapel                                |  10540 |
| Broadley Terrace, Marylebone                             |  10527 |
| Addison Road, Holland Park                               |  10494 |
| Imperial Wharf Station                                   |  10380 |
| Walmer Road, Avondale                                    |  10376 |
| Westferry Circus, Canary Wharf                           |  10290 |
| Lincoln's Inn Fields, Holborn                            |  10248 |
| Putney Rail Station, Putney                              |  10191 |
| Killick Street, King's Cross                             |  10123 |
| Stanley Grove, Battersea                                 |  10111 |
| Bradmead, Battersea Park                                 |  10003 |
| Culvert Road, Battersea                                  |   9886 |
| Grenfell Road, Avondale                                  |   9856 |
| Southwark Street, Bankside                               |   9855 |
| Upper Grosvenor Street, Mayfair                          |   9598 |
| Cromer Street, Bloomsbury                                |   9572 |
| Alpha Grove, Millwall                                    |   9529 |
| King Edward Walk, Waterloo                               |   9418 |
| Rifle Place, Avondale                                    |   9409 |
| Plough Terrace, Clapham Junction                         |   9391 |
| Kings Gate House, Westminster                            |   9306 |
| St Martins Close, Camden Town                            |   9286 |
| Clapham Common North Side, Clapham Common                |   9278 |
| Peterborough Road, Sands End                             |   9196 |
| Ossulston Street, Somers Town                            |   9036 |
| Ackroyd Drive, Bow                                       |   8974 |
| Geraldine Street, Elephant & Castle                      |   8950 |
| Abbotsbury Road, Holland Park                            |   8906 |
| Green Park Station, West End                             |   8902 |
| Hurlingham Park, Parsons Green                           |   8862 |
| Heath Road, Battersea                                    |   8670 |
| East Ferry Road, Cubitt Town                             |   8593 |
| Westminster Bridge Road, Elephant & Castle               |   8556 |
| Jubilee Crescent, Cubitt Town                            |   8422 |
| Putney Bridge Road, East Putney                          |   8414 |
| Oval Way, Vauxhall                                       |   8389 |
| Naval Row, Blackwall                                     |   8300 |
| Lancaster Drive, Blackwall                               |   8176 |
| Mallory Street, Marylebone                               |   8141 |
| Coram Street, Bloomsbury                                 |   8112 |
| Holden Street, Battersea                                 |   8102 |
| Newby Place, Poplar                                      |   8080 |
| Bevington Road West, North Kensington                    |   8070 |
| Paddington Green, Paddington                             |   8057 |
| Albert Square, Stockwell                                 |   8041 |
| Riverlight South, Nine Elms                              |   7984 |
| Embankment (Horse Guards), Westminster                   |   7981 |
| Ilchester Place, Kensington                              |   7969 |
| New North Road 1, Hoxton                                 |   7968 |
| Broomhouse Lane, Parsons Green                           |   7874 |
| Freston Road, Avondale                                   |   7777 |
| Bromley High Street, Bow                                 |   7761 |
| Alma Road, Wandsworth                                    |   7723 |
| Thessaly Road North, Wandsworth Road                     |   7617 |
| Cantrell Road, Bow                                       |   7508 |
| Evesham Street, Avondale                                 |   7406 |
| Mudchute DLR, Cubitt Town                                |   7265 |
| Clapham Road, Lingham Street, Stockwell                  |   7262 |
| Orsett Terrace, Bayswater                                |   7224 |
| Upper Richmond Road, Putney                              |   7152 |
| Hibbert Street, Clapham Junction                         |   7111 |
| Greenberry Street, St.John's Wood                        |   7099 |
| Belvedere Road 2, South Bank                             |   7079 |
| Vereker Road, West Kensington                            |   7027 |
| Queensdale Road, Shepherd's Bush                         |   7006 |
| Harrington Square 1, Camden Town                         |   6870 |
| Exhibition Road Museums, Knightsbridge                   |   6770 |
| Stebondale Street, Cubitt Town                           |   6750 |
| Sirdar Road, Avondale                                    |   6645 |
| Preston's Road, Cubitt Town                              |   6631 |
| Langdon Park, Poplar                                     |   6585 |
| Rodney Street, Angel                                     |   6485 |
| Handyside Street, King's Cross                           |   6464 |
| Ingrave Street, Clapham Junction                         |   6341 |
| Abingdon Green, Great College Street                     |   6160 |
| Stewart's Road, Wandsworth Road                          |   6123 |
| Oxford Road, Putney                                      |   6046 |
| Spencer Park, Wandsworth Common                          |   6037 |
| St. Mary & St. Michael Church, Stepney                   |   6024 |
| Morie Street, Wandsworth                                 |   5883 |
| New Spring Gardens Walk, Vauxhall                        |   5741 |
| Lord's, St. John's Wood                                  |   5728 |
| Limerston Street, Chelsea                                |   5711 |
| Mexfield Road, East Putney                               |   5617 |
| Esmond Street, Putney                                    |   5331 |
| Millharbour, Millwall                                    |   5311 |
| Theobalds Road , Holborn                                 |   5290 |
| Ladbroke Grove Central, Ladbroke Grove                   |   5226 |
| Malmesbury Road, Bow                                     |   5223 |
| Lansdowne Walk, Ladbroke Grove                           |   5168 |
| Park Road (Baker Street), Regent's Park                  |   5128 |
| Cadogan Gardens, Sloane Square                           |   5025 |
| Grant Road West, Clapham Junction                        |   4896 |
| Drayton Gardens, Chelsea                                 |   4841 |
| Limburg Road, Clapham Junction                           |   4814 |
| Stockwell Roundabout, Stockwell                          |   4649 |
| Santos Road, Wandsworth                                  |   4484 |
| Columbia Road, Weavers                                   |   4383 |
| Fishermans Walk West, Canary Wharf                       |   4342 |
| Teviot Street, Poplar                                    |   4188 |
| Spanish Road, Clapham Junction                           |   4186 |
| Lodge Road, St. John's Wood                              |   4148 |
| Queen Marys, Mile End                                    |   4118 |
| Pitfield Street (North),Hoxton                           |   4108 |
| Nantes Close, Clapham Junction                           |   4060 |
| Grant Road Central, Clapham Junction                     |   4056 |
| Merchant Street, Bow                                     |   4055 |
| Millbank House, Pimlico                                  |   3786 |
| Great Dover Street, Borough                              |   3733 |
| Cleveland Way, Bethnal Green                             |   3727 |
| St. John's Park, Cubitt Town                             |   3718 |
| New Road  2, Whitechapel                                 |   3568 |
| Sidney Street, Stepney                                   |   3553 |
| Walworth Road, Southwark                                 |   3538 |
| Fawcett Close, Clapham Junction                          |   3466 |
| Normand Park, Fulham                                     |   3455 |
| Aberfeldy Street, Poplar                                 |   3370 |
| The Tennis Courts, Regent's Park                         |   3306 |
| Nantes Close, Wandsworth                                 |   3296 |
| Thornfield House, Poplar                                 |   3288 |
| Colet Gardens, Hammersmith                               |   3238 |
| Bethnal Green Gardens, Bethnal Green                     |   3225 |
| Edgware Road Station, Paddington                         |   3217 |
| Here East North, Queen Elizabeth Olympic Park            |   3214 |
| Hoxton Station, Haggerston                               |   3111 |
| Manfred Road, East Putney                                |   3069 |
| St Katharines Way, Tower                                 |   3044 |
| Gun Makers Lane, Old Ford                                |   3003 |
| Goldsmiths Row, Haggerston                               |   2986 |
| Prince Albert Road, Regent's Park                        |   2971 |
| Lansdowne Way Bus Garage, Stockwell                      |   2938 |
| Castalia Square, Cubitt Town                             |   2931 |
| Austin Road, Battersea                                   |   2802 |
| London Zoo Car Park, Regent's Park                       |   2801 |
| Fawcett Close, Battersea                                 |   2767 |
| Simpson Street, Battersea                                |   2643 |
| Altab Ali Park, Whitechapel                              |   2619 |
| South Quay East, Canary Wharf                            |   2591 |
| Albany Street, Regent's Park                             |   2586 |
| Churchill Place, Canary Wharf                            |   2394 |
| New Kent Road, Borough                                   |   2291 |
| Longford Street, Regent's Park                           |   2259 |
| Kingsway, Covent Garden                                  |   2250 |
| London Zoo, Regents Park                                 |   2219 |
| Hansard Mews, Shepherds Bush                             |   2115 |
| Victory place, Walworth                                  |   2091 |
| Strata, Southwark                                        |   2024 |
| Clapham Common Northside, Clapham Common                 |   1972 |
| Killick Street, Kings Cross                              |   1852 |
| Waterloo Roundabout, Waterloo                            |   1805 |
| Bradmead, Nine Elms                                      |   1797 |
| Bevington Road, North Kensington                         |   1738 |
| Westfield Eastern Access Road, Shepherd's Bush           |   1693 |
| Victoria and Albert Museum, Cromwell Road                |   1638 |
| St. John's Wood Church, Regent's Park                    |   1595 |
| Clarges Street, West End                                 |   1521 |
| Battersea Power Station, Battersea Park                  |   1412 |
| Harper Road, Borough                                     |   1360 |
| Sugden Road, Battersea                                   |   1260 |
| Russell Gardens, Holland Park                            |   1259 |
| Walmer Road, Notting Hill                                |   1250 |
| St Martin's Close, Camden Town                           |   1199 |
| Usk Road, Battersea                                      |   1083 |
| Oval Way, Lambeth                                        |   1022 |
| Bromley High Street, Bromley                             |    949 |
| South Quay West, Canary Wharf                            |    906 |
| Thessaly Road North, Nine Elms                           |    895 |
| Upper Richmond Road, East Putney                         |    880 |
| Halford Road, Fulham                                     |    812 |
| St. Mary and St. Michael Church, Stepney                 |    805 |
| Central House, Whitechapel                               |    778 |
| Harrington Square, Camden Town                           |    740 |
| Mechanical Workshop Clapham                              |    727 |
| Mechanical Workshop Penton                               |    663 |
| Ingrave Street, Battersea                                |    639 |
| Victory Place, Walworth                                  |    609 |
| Lansdowne Walk, Notting Hill                             |    602 |
| Stewart's Road, Nine Elms                                |    572 |
| Spanish Road, Wandsworth                                 |    537 |
| Ladbroke Grove Central, Notting Hill                     |    526 |
| Limburg Road, Clapham Common                             |    481 |
| St John's Park, Cubitt Town                              |    393 |
| Coborn Street, Mile End                                  |    315 |
| Here East South, Queen Elizabeth Olympic Park            |    241 |
| Contact Centre, Southbury House                          |     60 |
| Pop Up Dock 1                                            |     41 |
| Monier Road, Newham                                      |     36 |
| Pop Up Dock 2                                            |     19 |
| Monier Road                                              |     14 |
| PENTON STREET COMMS TEST TERMINAL _ CONTACT MATT McNULTY |     11 |
| LSP1                                                     |      6 |
| Blackfriars road, Southwark                              |      5 |
| Electrical Workshop PS                                   |      2 |
| LSP2                                                     |      2 |
+----------------------------------------------------------+--------+
883 rows in set (0.16 sec)

mysql>
```

```sql
mysql> DELETE FROM london1 WHERE num=0;
Query OK, 1 row affected (0.17 sec)

mysql> DELETE FROM london2 WHERE num=0;
Query OK, 1 row affected (0.16 sec)

mysql>
```



```sql
mysql> INSERT INTO london1 (start_station_name, num) VALUES ("test destination", 1);
Query OK, 1 row affected (0.17 sec)

mysql>
```

### UNION keyword

The last SQL keyword that you'll learn about is `UNION`. This keyword combines the output of two or more `SELECT` queries into a result-set. 



```sql
mysql> SELECT start_station_name AS top_stations, num FROM london1 WHERE num>100000
    -> UNION
    -> SELECT end_station_name, num FROM london2 WHERE num>100000
    -> ORDER BY top_stations DESC;
+-------------------------------------+--------+
| top_stations                        | num    |
+-------------------------------------+--------+
| Wormwood Street, Liverpool Street   | 119447 |
| Wormwood Street, Liverpool Street   | 129376 |
| Wellington Arch, Hyde Park          | 110260 |
| Wellington Arch, Hyde Park          | 105729 |
| Waterloo Station 3, Waterloo        | 201630 |
| Waterloo Station 3, Waterloo        | 193200 |
| Waterloo Station 1, Waterloo        | 145910 |
| Waterloo Station 1, Waterloo        | 141733 |
| Triangle Car Park, Hyde Park        | 108347 |
| Triangle Car Park, Hyde Park        | 107372 |
| Newgate Street , St. Paul's         | 108223 |
| Hyde Park Corner, Hyde Park         | 215629 |
| Hyde Park Corner, Hyde Park         | 215038 |
| Hop Exchange, The Borough           | 115135 |
| Hop Exchange, The Borough           | 156964 |
| Finsbury Circus, Liverpool Street   | 105810 |
| Finsbury Circus, Liverpool Street   | 116011 |
| Craven Street, Strand               | 104457 |
| Brushfield Street, Liverpool Street | 103114 |
| Brushfield Street, Liverpool Street | 120659 |
| Black Lion Gate, Kensington Gardens | 161952 |
| Black Lion Gate, Kensington Gardens | 156020 |
| Bethnal Green Road, Shoreditch      | 100005 |
| Bethnal Green Road, Shoreditch      | 100590 |
| Belgrove Street , King's Cross      | 234458 |
| Belgrove Street , King's Cross      | 231802 |
| Albert Gate, Hyde Park              | 155647 |
| Albert Gate, Hyde Park              | 157943 |
+-------------------------------------+--------+
28 rows in set (0.16 sec)

mysql>
```

The first `SELECT` query selects the two columns from the  "london1" table and creates an alias for "start_station_name", which  gets set to "top_stations". It uses the `WHERE` keyword to only pull rideshare station names where over 100,000 bikes start their journey.

The second `SELECT` query selects the two columns from the "london2" table and uses the `WHERE` keyword to only pull rideshare station names where over 100,000 bikes end their journey.

The `UNION` keyword in between combines the output of  these queries by assimilating the "london2" data with "london1". Since  "london1" is being unioned with "london2", the column values that take  precedence are "top_stations" and "num".

`ORDER BY` will order the final, unioned table by the "top_stations" column value alphabetically and in descending order.



As you see, 13/14 stations share the top spots for rideshare starting  and ending points. With some basic SQL keywords you were able to query a sizable dataset, which returned data points and answers to specific  questions.



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-401.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-402.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-403.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-404.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-405.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-406.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-407.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-408.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-409.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-410.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-411.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-412.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-413.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-414.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-415.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-416.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-417.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-418.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-419.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-420.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-421.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-422.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-423.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-424.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-425.png)
