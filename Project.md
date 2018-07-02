# The project

The goal of our project is to find out which countries or teams are the most popular in the Netherlands during the Football World Cup in Russia, we analyze this by looking at 4 Dutch newswebsites and we count both the number of times each country is named and the number of articles that the country appears in. 

## The data

We used wget with recursion to download the websites from NOS, NU, AD and de Telegraaf.

**__Same number of files, new wget?__**

## The code
We have 4 RDD's, one for every news website. Of course for every website the way they make their links are different, therefore we have to use 4 different ways to filter out the data we want to use, namely the news articles about the World Cup.

```scala
  val warcctelegraaf = warcftelegraaf.
  filter{ _._2.header.warcTypeIdx == 2 /* response */ }.
  filter{ _._2.getHttpHeader().statusCode != 404 }.
  map{wr => ( wr._2.header.warcTargetUriStr, getContent(wr._2) )}.filter{ _._2.contains("in WK 2018")}
  ```
  
```scala
  val warccnos = warcfnos.
  filter{ _._2.header.warcTypeIdx == 2 /* response */ }.
  filter{ _._2.getHttpHeader().statusCode != 404 }.
  filter{ _._2.header.warcTargetUriStr.startsWith("https://nos.nl/wk2018/")}.
  map{wr => ( wr._2.header.warcTargetUriStr, HTML2Txt(getContent(wr._2)) )}
```

```scala
  val warccad = warcfad.
  filter{ _._2.header.warcTypeIdx == 2 /* response */ }.
  filter{ _._2.getHttpHeader().statusCode != 404 }.
  filter{ _._2.header.warcTargetUriStr.contains("wk-2018")}.
  map{wr => ( wr._2.header.warcTargetUriStr, HTML2Txt(getContent(wr._2)) )}
```

```scala
  val warccnu = warcfnu.
  filter{ _._2.header.warcTypeIdx == 2 /* response */ }.
  filter{ _._2.getHttpHeader().statusCode != 404 }.
  filter{ _._2.header.warcTargetUriStr.contains("wk-2018")}.
  map{wr => ( wr._2.header.warcTargetUriStr, HTML2Txt(getContent(wr._2)) )}
```




# The results
We find the following results for the different news websites

**AD**

|Number of documents|Number of mentions|
|---|---|
|![alt text](https://github.com/TomValckx/hello-world/blob/master/DocCountAD.png "Number of documents in AD")|![alt text](https://github.com/TomValckx/hello-world/blob/master/WordCountAD.png "Number of mentions in AD")|

**NU**

|Number of documents|Number of mentions|
|---|---|
|![alt text](https://github.com/TomValckx/hello-world/blob/master/DocCountNU.png "Number of documents in NU")|![alt text](https://github.com/TomValckx/hello-world/blob/master/WordCountNU.png "Number of mentions in NU")|

**NOS**

|Number of documents|Number of mentions|
|---|---|
|![alt text](https://github.com/TomValckx/hello-world/blob/master/DocCountNOS.png "Number of documents in NOS")|![alt text](https://github.com/TomValckx/hello-world/blob/master/WordCountNOS.png "Number of mentions in NOS")|

**Telegraaf**

|Number of documents|Number of mentions|
|---|---|
|![alt text](https://github.com/TomValckx/hello-world/blob/master/DocCountTG.png "Number of documents in Telegraaf")|![alt text](https://github.com/TomValckx/hello-world/blob/master/WordCountTG.png "Number of mentions in Telegraaf")|









# Problems?
