---

layout: single
title:  "BigQuery: Qwik Start - Console"
date:   2023-03-31 04:59:04 +0530
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

# BigQuery: Qwik Start - Console

Storing and querying massive datasets can be time consuming and  expensive without the right hardware and infrastructure. BigQuery is an  [enterprise data warehouse](https://cloud.google.com/solutions/bigquery-data-warehouse) that solves this problem by enabling super-fast SQL queries using the processing power of Google's infrastructure.

The BigQuery console provides an interface to query tables, including  [public datasets](https://cloud.google.com/bigquery/public-data) offered by BigQuery.  The query you will run accesses a table from a  public dataset that BigQuery provides. It uses standard query language  to search the dataset, and limits the results returned to 10.

```sql
#standardSQL
SELECT
 weight_pounds, state, year, gestation_weeks
FROM
 `bigquery-public-data.samples.natality`
ORDER BY weight_pounds DESC LIMIT 10;
```

**Note:**  You can browse the schema of other public datasets in BigQuery by clicking  **+ Add** > **Public Datasets**, then search for "bigquery public data" in the Search Marketplace field.

To load custom data into a table, you perform the following tasks:

- Create a dataset
- Create a table
- Add data to your project (to a storage bucket)
- Load the data from the bucket to the table you created

```sh
05:22:40 ide-dev@cloudlearningservices ~ → gsutil cp gs://spls/gsp072/baby-names.zip .
Copying gs://spls/gsp072/baby-names.zip...
/ [1 files][  6.7 MiB/  6.7 MiB]                                                
Operation completed over 1 objects/6.7 MiB.                                      
05:22:59 ide-dev@cloudlearningservices ~ → unzip baby-names.zip
Archive:  baby-names.zip
  inflating: yob1880.txt             
  inflating: yob1881.txt             
  inflating: yob1882.txt             
  inflating: yob1883.txt             
  inflating: yob1884.txt             
  inflating: yob1885.txt             
  inflating: yob1886.txt             
  inflating: yob1887.txt             
  inflating: yob1888.txt             
  inflating: yob1889.txt             
  inflating: yob1890.txt             
  inflating: yob1891.txt             
  inflating: yob1892.txt             
  inflating: yob1893.txt             
  inflating: yob1894.txt             
  inflating: yob1895.txt             
  inflating: yob1896.txt             
  inflating: yob1897.txt             
  inflating: yob1898.txt             
  inflating: yob1899.txt             
  inflating: yob1900.txt             
  inflating: yob1901.txt             
  inflating: yob1902.txt             
  inflating: yob1903.txt             
  inflating: yob1904.txt             
  inflating: yob1905.txt             
  inflating: yob1906.txt             
  inflating: yob1907.txt             
  inflating: yob1908.txt             
  inflating: yob1909.txt             
  inflating: yob1910.txt             
  inflating: yob1911.txt             
  inflating: yob1912.txt             
  inflating: yob1913.txt             
  inflating: yob1914.txt             
  inflating: yob1915.txt             
  inflating: yob1916.txt             
  inflating: yob1917.txt             
  inflating: yob1918.txt             
  inflating: yob1919.txt             
  inflating: yob1920.txt             
  inflating: yob1921.txt             
  inflating: yob1922.txt             
  inflating: yob1923.txt             
  inflating: yob1924.txt             
  inflating: yob1925.txt             
  inflating: yob1926.txt             
  inflating: yob1927.txt             
  inflating: yob1928.txt             
  inflating: yob1929.txt             
  inflating: yob1930.txt             
  inflating: yob1931.txt             
  inflating: yob1932.txt             
  inflating: yob1933.txt             
  inflating: yob1934.txt             
  inflating: yob1935.txt             
  inflating: yob1936.txt             
  inflating: yob1937.txt             
  inflating: yob1938.txt             
  inflating: yob1939.txt             
  inflating: yob1940.txt             
  inflating: yob1941.txt             
  inflating: yob1942.txt             
  inflating: yob1943.txt             
  inflating: yob1944.txt             
  inflating: yob1945.txt             
  inflating: yob1946.txt             
  inflating: yob1947.txt             
  inflating: yob1948.txt             
  inflating: yob1949.txt             
  inflating: yob1950.txt             
  inflating: yob1951.txt             
  inflating: yob1952.txt             
  inflating: yob1953.txt             
  inflating: yob1954.txt             
  inflating: yob1955.txt             
  inflating: yob1956.txt             
  inflating: yob1957.txt             
  inflating: yob1958.txt             
  inflating: yob1959.txt             
  inflating: yob1960.txt             
  inflating: yob1961.txt             
  inflating: yob1962.txt             
  inflating: yob1963.txt             
  inflating: yob1964.txt             
  inflating: yob1965.txt             
  inflating: yob1966.txt             
  inflating: yob1967.txt             
  inflating: yob1968.txt             
  inflating: yob1969.txt             
  inflating: yob1970.txt             
  inflating: yob1971.txt             
  inflating: yob1972.txt             
  inflating: yob1973.txt             
  inflating: yob1974.txt             
  inflating: yob1975.txt             
  inflating: yob1976.txt             
  inflating: yob1977.txt             
  inflating: yob1978.txt             
  inflating: yob1979.txt             
  inflating: yob1980.txt             
  inflating: yob1981.txt             
  inflating: yob1982.txt             
  inflating: yob1983.txt             
  inflating: yob1984.txt             
  inflating: yob1985.txt             
  inflating: yob1986.txt             
  inflating: yob1987.txt             
  inflating: yob1988.txt             
  inflating: yob1989.txt             
  inflating: yob1990.txt             
  inflating: yob1991.txt             
  inflating: yob1992.txt             
  inflating: yob1993.txt             
  inflating: yob1994.txt             
  inflating: yob1995.txt             
  inflating: yob1996.txt             
  inflating: yob1997.txt             
  inflating: yob1998.txt             
  inflating: yob1999.txt             
  inflating: yob2000.txt             
  inflating: yob2001.txt             
  inflating: yob2002.txt             
  inflating: yob2003.txt             
  inflating: yob2004.txt             
  inflating: yob2005.txt             
  inflating: yob2006.txt             
  inflating: yob2007.txt             
  inflating: yob2008.txt             
  inflating: yob2009.txt             
  inflating: yob2010.txt             
  inflating: yob2011.txt             
  inflating: yob2012.txt             
  inflating: yob2013.txt             
  inflating: yob2014.txt             
  inflating: yob2015.txt             
  inflating: yob2016.txt             
  inflating: yob2017.txt             
  inflating: NationalReadMe.pdf      
05:23:10 ide-dev@cloudlearningservices ~ → 
```





```sh
05:25:14 ide-dev@cloudlearningservices ~ → gsutil cp yob2014.txt gs://pradeepgadde
Copying file://yob2014.txt [Content-Type=text/plain]...
/ [1 files][417.9 KiB/417.9 KiB]                                                
Operation completed over 1 objects/417.9 KiB.                                    
05:25:29 ide-dev@cloudlearningservices ~ → 

```

Next you create a table inside the **babynames** dataset, then load the data file from your storage bucket into the new table.

Running a query against custom data is identical to the [querying a public dataset](https://cloud.google.com/bigquery/quickstart-web-ui#query_a_public_dataset) that you did earlier, except that now you're querying your own table instead of a public table.



```sql
#standardSQL
SELECT
 name, count
FROM
 `babynames.names_2014`
WHERE
 gender = 'M'
ORDER BY count DESC LIMIT 5;
```



We used the BigQuery Web UI to query public tables and load sample data into BigQuery.



  ![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-435.png)

  ![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-436.png)

  ![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-437.png)

  ![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-438.png)

  ![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-439.png)

  ![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-440.png)

  ![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-441.png)

  ![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-442.png)

  ![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-443.png)

  ![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-444.png)

  ![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-445.png)

  ![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-446.png)

