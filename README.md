
# Lab: Counting words with Spark and Map-Reduce

## SWBAT

* Describe Map-Reduce operation in a big data context.
* Perform basic NLP tasks with a given text corpus.
* Perform basic analysis from the experiment findings towards identifying writing styles

### MAP-REDUCE Start-up task:

* We have a huge text document
* We need to count the number of times each distinct word appears in the document


* Sample application:

    * Analyze web server logs to find popular URLs
    * Analyze texts for content or style 

## Introduction

We shall illustrate a MapReduce computation for a standard example application, i.e. counting the number of occurrences for each word in a text corpus. In this example, the input file is a repository of documents, and each document is an element. We shall count the frequency of stop words for style identification as stop words might have unique features which can potentially describe author's writing style based on their use of stop words while writing. We shall look at some texts by Shakspeare and Jane Austin following this motivation. 

Map-Reduce in PySpark provides a practical and efficient way of achieving this goal as it: 

    * works if the file is too large for memory

    * works even if the ouput is too large for memory

    * is naturally parallelizable

    * captures the essence of MapReduce

### Map-Reduce Approach

* Sequentially read a lot of data (text files in this case)


* Map:
    * Extract something you care about


* Group by key: Sort and Shuffle


* Reduce:
    * Aggregate, summarize, filter or transform


* Write the result 


### Initialize SparkContext()

Let's import the pyspark module into python environment and initialize a `SparkConext()`

**Note: for virtual machines, you can only initiate a single spark context.** 


```python
# Start a local SparkContext
import pyspark
sc = pyspark.SparkContext('local[*]') # [*] represents a local context i.e. no cluster
```

To test our code, we shall start with a single text file, hamlet.txt. First we shall set a file path variable.


```python
file = 'text/hamlet.txt'

```

### Read text file contents into RDD

We shall read the text file into Spark RDDs by using `sc.textFile()` method for loading the text file into the `lines` RDD. The documentation on RDDs can be found [here!!](https://spark.apache.org/docs/latest/rdd-programming-guide.html)

`textFile(path)` method reads a text file from HDFS/local file system/any hadoop supported file system, into the number of partitions specified and returns it as an RDD of Strings. In order to view the contents of RDD, we will use `RDD.collect()` method as calling the RDD by name will not return the contents, only the object type and relevant information 


```python
# use .textfile() to load text into the RDD
lines = None
lines

# text/hamlet.txt MapPartitionsRDD[36] at textFile at NativeMethodAccessorImpl.java:0
```


```python
# Print the text, line-by-line using a for loop 
# This will output the whole of hamlet text, one line at a time. 




# 1604


# THE TRAGEDY OF HAMLET, PRINCE OF DENMARK


# by William Shakespeare
# .
# . lots
# . of
# . text
# . 
# Take up the bodies. Such a sight as this
#     Becomes the field but here shows much amiss.
#     Go, bid the soldiers shoot.
#             Exeunt marching; after the which a peal of ordnance
#                                                    are shot off.  


# THE END

```

Now that we have the text files into `lines` RDD, we can easily use map function to break it down further into individual words and parallelize it accoridngly. (Note: Paralellization is handled by Spark environment accordnig to available infrastructure and doesnt need any further configuration)

## The MAP function

The Map function for this example uses keys that are of type String (the words) and values that are integers. The Map task reads a document and breaks it into its sequence of words `w1, w2, . . . , wn`. It then emits a sequence of key-value pairs for each word where the word iteself is the key and the value is always 1. That is, the output of the Map task for this document is the sequence of key-value pairs as shown below:

> `(w1, 1), (w2, 1), . . . ,(wn, 1)`

Map Function is the first step in MapReduce Algorithm. It takes input tasks and divides them into smaller sub-tasks and then performs required computation on each sub-task in parallel.

This step performs the following two sub-steps:

* Splitting step takes input DataSet from Source and divide into smaller Sub-Sets.
* Mapping step takes those smaller Sub-DataSets and perform required action or computation on each Sub-DataSet.

The output of this Map Function is a set of key and value pairs as <Key, Value> as shown in the below diagram.



![](map.png)


### Spark Map functions

`map(func)`	return a new distributed dataset formed by passing each element of the source through a function `func`.

`flatMap(func)` maps each input item to 0 or more output items (so `func` should return a Seq rather than a single item).

---

>Use `RDD.flatMap` to split the lines by the spaces and collect into one flat RDD.

>The transformation is defined in the lambda expression, where the input x is defined as producing result  
`x.split(' ')`.

>Use the `RDD.take(n)` method to pick n words from the top of the sequence. 

`flatMap()` breaks the output of the lambda function into individual RDD elements (as opposed to map).



```python
# split the lines into words based on blanks ' ' and show ten elements from the top 

# ['', '1604', '', '', 'THE', 'TRAGEDY', 'OF', 'HAMLET,', 'PRINCE', 'OF']
```


### Create a Tuple as (k,v)

Map each words to a tuple of (word, 1).
Map doesn't break up the output of the lambda expression, so that the tuples stay intact.


```python
# Use a lambda function with map to add a 1 to each word and output a tuple
# (word, 1) - Take ten elements



# [('', 1),
#  ('1604', 1),
#  ('', 1),
#  ('', 1),
#  ('THE', 1),
#  ('TRAGEDY', 1),
#  ('OF', 1),
#  ('HAMLET,', 1),
#  ('PRINCE', 1),
#  ('OF', 1)]
```

### Change the words to lower case to ensure integrity

As we can see from the output above, the text contains words in capital as well as lower case. By default, 'THE' and 'the' would be considered two separate words due to case sensitivity. We can modify the map function above to change all the words to lowercase using a `.lower()` inside the lambda function.



```python
# Change the words in words tuples to lowercase - take 10 elements 



# [('', 1),
#  ('1604', 1),
#  ('', 1),
#  ('', 1),
#  ('the', 1),
#  ('tragedy', 1),
#  ('of', 1),
#  ('hamlet,', 1),
#  ('prince', 1),
#  ('of', 1)]
```

## REDUCE Function
The Reduce function’s argument is a pair consisting of a key and its list of associated values as the pairs created above. The output of the Reduce function is a sequence of zero or more key-value pairs. These key-value pairs can be of a type different from those sent from Map tasks to Reduce tasks, but often they are the same type.

We shall refer to the application of the Reduce function to a single key and its associated list of values as a reducer.

![](reduce.png)

Use `RDD.reduceByKey` to add up all the words. the new k,v pairs would have word as the key and number of occurances as a value. 

Here, the lambda has two arguments (x and y) that are added.


```python
# USe reduceByKey with tuplesLCase to add all values under same keys - take 10



# [('', 20383),
#  ('1604', 1),
#  ('tragedy', 1),
#  ('of', 670),
#  ('prince', 2),
#  ('denmark', 10),
#  ('shakespeare', 1),
#  ('dramatis', 1),
#  ('claudius,', 2),
#  ('king', 43)]
```

### Filter rare words

Following the standard NLP approach, we can add a filtering step to remove all words which appear less than some thershaold value, say, with less than 5 occurrences. 

This can be useful to identify common topics between documents, where very rare words can be misleading. 
For this step we shall use the `RDD.filter(func)` where func is a lambda function that filters out any word which appears less than or equal to 5 times. You may also use a spearate function to achieve this. 


```python
# Remove all rare words with frequency less than 5 - take 10 



# [('', 20383),
#  ('of', 670),
#  ('denmark', 10),
#  ('king', 43),
#  ('son', 11),
#  ('polonius,', 6),
#  ('horatio,', 15),
#  ('hamlet.', 25),
#  ('courtier.', 7),
#  ('rosencrantz,', 6)]
```

### List  Stopwords

Add a filtering step to retain only words included in a list of stopwords. 

Stopwords can be useful for recognising the style of an author. Removing stopwords can be useful in regocnising the topic of a document. For stopword removal, we use the `RDD.filter(func)` again with a lambda function that uses a stop word list to extract the key value pairs for only the words that are present in the stop word list. Use a simple list as the one shown below:
> ['', 'the','a','in','of','on','at','for','by','I','you','me'] 



```python
# Use the stop word list above to count the occcurances of these words in the document
# show stopword frequency in the output



# [('', 20383),
#  ('of', 670),
#  ('at', 87),
#  ('i', 523),
#  ('in', 420),
#  ('the', 1083),
#  ('by', 111),
#  ('a', 540),
#  ('you', 433),
#  ('for', 231),
#  ('me', 144),
#  ('on', 108)]
```

### List of keep words

We can modify the filter operation above to keep all the words found in the text **except** the stop words as well. 


```python
# Modify above filter to show top ten keep words by frequency



# [('and', 939),
#  ('to', 727),
#  ('my', 519),
#  ('ham.', 358),
#  ('that', 343),
#  ('is', 327),
#  ('it', 327),
#  ('his', 302),
#  ('not', 274),
#  ('with', 267)]
```

### Putting it all together 

Combine above code as a function and pass on three works of Shakespeare (romeandjuliet.txt, hamlet.txt, othello.txt) and observe the frequency of stop words. Repeat the same exercise for three works of Jane Austin (senseandsensibility.txt, prideandprejudice.txt and emma.txt). 

> Can you recognise the writing styles of these authors based on their use of stop words ?
> What can you do to improve the style recognition ability ??


```python
def wordCount(filename, stopWordlist):
    
    return None
```

## Summary 

In this simple exercise , we saw map-reduce in action towards solving a basic NLP task i.e. counting the stop words and keep words frequency of a text corpus. This exercise can be seen as a first step towards text analystics on big data platforms. 
