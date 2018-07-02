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
  map{wr => ( wr._2.header.warcTargetUriStr, getContent(wr._2) )}.
  filter{ _._2.contains("in WK 2018")}
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

We then define a dictionary of all the words that we want to count, the 32 participating countries.

```scala
val dictionary = Map(
    """Rusland""" -> 1,
    """Egypte""" -> 2,
    """Uruguay""" -> 3,
    """Saudi-Arabië""" -> 4,
    """Spanje""" -> 5,
    """Portugal""" -> 6,
    """Marokko""" -> 7,
    """Iran""" -> 8,
    """Iran""" -> 8,
    """Frankrijk""" -> 9,
    """Denemarken""" -> 10,
    """Peru""" -> 11,
    """Australië""" -> 12,
    """Kroatië""" -> 13,
    """IJsland""" -> 14,
    """Argentinië""" -> 15,
    """Nigeria""" -> 16,
    """Servië""" -> 17,
    """Brazilië""" -> 18,
    """Rica""" -> 19,
    """Zwitserland""" -> 20,
    """Mexico""" -> 21,
    """Zweden""" -> 22,
    """Duitsland""" -> 23,
    """Zuid-Korea""" -> 24,
    """België""" -> 25,
    """Engeland""" -> 26,
    """Tunesië""" -> 27,
    """Panama""" -> 28,
    """Japan""" -> 29,
    """Senegal""" -> 30,
    """Colombia""" -> 31,
    """Polen""" -> 32
  )

val validWords = dictionary.keys.toSet
```
Now we have all the articles we want and the words we want to count in them we can split all the articles into words and filter only the words we want. Then we add a count 1 to every word and use *reduceByKey(_+_)* to count the number of times the word is mentioned.

```scala
val article_textstelegraaf = warcctelegraaf.map{ tt => (StringUtils.substring(tt._2, 0, 1000000000))}
val article_textsnos = warccnos.map{ tt => (StringUtils.substring(tt._2, 0, 1000000000))}
val article_textsad = warccad.map{ tt => tt._2}
val article_textsnu = warccnu.map{ tt => (StringUtils.substring(tt._2, 0, 1000000000))}



val wordstelegraaf = article_textstelegraaf.flatMap(line => line.split(" ")).filter(_ != "")
val wordsnos = article_textsnos.flatMap(line => line.split(" ")).filter(_ != "")
val wordsad = article_textsad.flatMap(line => line.split(" ")).filter(_ != "")
val wordsnu = article_textsnu.flatMap(line => line.split(" ")).filter(_ != "")

val filteredWordstelegraaf = wordstelegraaf.filter(word => validWords.contains(word)).map( word => word.replace("Rica","Costa Rica")).map(word => (word,1))
val filteredWordsnos = wordsnos.filter(word => validWords.contains(word)).map( word => word.replace("Rica","Costa Rica")).map(word => (word,1))
val filteredWordsad = wordsad.filter(word => validWords.contains(word)).map( word => word.replace("Rica","Costa Rica")).map(word => (word,1))
val filteredWordsnu = wordsnu.filter(word => validWords.contains(word)).map( word => word.replace("Rica","Costa Rica")).map(word => (word,1))

val wctelegraaf = filteredWordstelegraaf.reduceByKey(_ + _)
val wcnos = filteredWordsnos.reduceByKey(_ + _)
val wcad = filteredWordsad.reduceByKey(_ + _)
val wcnu = filteredWordsnu.reduceByKey(_ + _)
```
This results in the word counts for every country per news website. To find the number of articles about a certain country we first set a random number generator `cajfaljgalj;ahgadfa`. We then add a random number in front of every article. Then we split the article into tuples of (word, vector(random numbers)). Finally we map this to a tuple (word, number of distinct numbers).

```scala
val word_occurencetelegraaf = article_textstelegraaf.map(x=>r.nextFloat+x).map(_.split(" "))
      .flatMap(x => x.map(y => (y, x(0))))
      .groupBy(_._1)
      .map(p => (p._1, p._2.map(_._2).toVector))

val nrdoctelegraaf = word_occurencetelegraaf.filter( x => validWords.contains(x._1)).map(x => (x._1,x._2.distinct.length))
```
We do this for all 4 newspapers to find the numbers of articles about every country.


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
