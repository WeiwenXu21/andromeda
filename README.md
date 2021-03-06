# Scalable Document Classification

This repository contains a Naive Bayes classifier implemented on document classification which is completed on CSCI 8360, Data Science Practicum at the University of Georgia, Spring 2018.

This project uses the Reuters Corpus, a set of news stories split into a [hierarchy of categories](https://github.com/dsp-uga/team-andromeda-p1/wiki/Hierarchy-of-Categories), but only specifies four categories as follows:

1. **CCAT**: Corporate / Industrial
2. **ECAT**: Economics
3. **GCAT**: Government / Social
4. **MCAT**: Markets

For those documents with more than one categories (which usually happen), we regard them as if we observed the same document once for each categories. For instance, for the document with CCAT and MCAT categories, we duplicate the document, and pair one of them with CCAT and one of them with MCAT.

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

### Prerequisites

- [Python 3.6](https://www.python.org/downloads/release/python-360/)
- [Apache Spark 2.2.1](http://spark.apache.org/)
- [Pyspark 2.2.1](https://pypi.python.org/pypi/pyspark/2.2.1) - Python API for Apache Spark
- [Google Cloud Platform](https://cloud.google.com)
- [Anaconda](https://www.anaconda.com/) - packages manager for [nltk](http://www.nltk.org/), [string](https://docs.python.org/3/library/string.html)

### Environment Setup

### Anaconda

Anaconda is a complete Python distribution embarking automatically the most common packages, and allowing an easy installation of new packages.

Download and install Anaconda (https://www.continuum.io/downloads).

The `environment.yml` file for conda is placed in [Extra](https://github.com/dsp-uga/team-andromeda-p1/tree/master/Extra) for your ease of installation

### Spark

Download the latest, pre-built for Hadoop 2.6, version of Spark.
* Go to http://spark.apache.org/downloads.html
* Choose a release (prendre la dernière)
* Choose a package type: Pre-built for Hadoop 2.6 and later
* Choose a download type: Direct Download
* Click on the link in Step 4
* Once downloaded, unzip the file and place it in a directory of your choice

Go to [WIKI](https://github.com/dsp-uga/team-andromeda-p1/wiki) tab for more details of running IDE for Pyspark. ([IDE Setting for Pyspark](https://github.com/dsp-uga/team-andromeda-p1/wiki/IDE-Setting-for-Pyspark))

### NLTK
To Download NLTK stopwords on GCP
First
```
$ pip install nltk
```
Once installed you'll have to Download the stopwords file
```
$ python
```
Once python prompt has started
```
>>> import nltk
>>> nltk.download()
```
This will start a UI or a Command prompt input based instructions

#### Comand prompt Mode
Type d and enter and type stopwords this will initiate a download
Best way would be ```python -m nltk.downloader -d /usr/local/share/nltk_data popular```

#### UI mode
Search for the sopword button and press download

Post this please make sure that all nltk files are in ```/usr/local/share```



## Running the tests

You can run `p1.py` via regular **python** or run the script via **spark-submit**. You should specify the path to your spark-submit.

```
$ python p1.py [training-directory] [testing-directory] [optional args]
```
```
$ usr/bin/spark-submit p1.py [training-directory] [testing-directory] [optional args]
```

The output file `pred_test_<size>.txt` can be customized by the size you selected, and saved to directory you specified. The required and optional arguments are as follows:

  - **Required Arguments**

    - `ptrain`: Directory contains the input training files

    - `ptest`: Directory contains the input testing files

  - **Optional Arguments**

    - `-s`: Sizes to the selected file. (Default: `vsmall`)

       `vsmall` for very small dataset, `small` for small dataset, and `large` for large dataset. The output file will be connected with this selected size, e.g. `pred_test_vsmall.txt`.
    - `-o`: Path to the output directory where outputs will be written. (Default: root directory)
    - `-a`: Accuracy of the testing prediction. (Default: `True`)

       The options gives you the accuracy of the prediction. If the file of testing label does not exist, it will still output the file but print out `Accuracy is not available!`.

### Packages used

After Splitting the document content, we implement punctuation stripping and words stemming by several python APIs. There are some brief explanations about the packages and more details in the  [WIKI](https://github.com/dsp-uga/team-andromeda-p1/wiki) tab.


#### Punctuation: `string`

```Python
import string
PUNC = string.punctuation
```

Import the `string` package, then the `string.punctuation` gives you the list of all punctuation.
We remove all punctuation before and after the word, but ignore those punctuation between two words, e.g. `happy--newyear`, `super.....bowl`

#### Stopwords: `nltk.corpus`

```Python
import nltk
nltk.download('stopwords')
from nltk.corpus import stopwords
SW = stopwords.words('english')
```

Import the `stopwords` under `nltk.corpus`, then `stopwords.words('english')` gives you the stopwords in English. Notice that you should import `nltk` first, and download `stopwords` from it before importing it from corpus.
We remove those stopwords that might confuse our classifier of the important words, e.g. `the`, `is`, `you`.

#### Words Stemming: `nltk.stem`

```Python
import nltk

from nltk.stem.lancaster import LancasterStemmer
lancaster_stemmer = LancasterStemmer()
word = lancaster_stemmer.stem(word)

from nltk.stem.porter import PorterStemmer
ps = PorterStemmer()
word = ps.stem(word)

nltk.download('wordnet')
from nltk.stem.wordnet import WordNetLemmatizer
wnl = WordNetLemmatizer()
word = wnl.lemmatize(word)
```

We have tried three stemming packages from `nltk.stem` in this project. The examples of each stemming packages (Lemmatizer, Lancaster, Porter) are introduced in the [WIKI](https://github.com/dsp-uga/team-andromeda-p1/wiki) tab. Notice that you have to download `wordnet` before importing it.
After implementing words stemming, all the words are transferred to their stems, e.g. `cars`, `car's`, `car` all become `car`.

### Algorithm

#### Overview

This project mainly uses [Naive Bayes classifier](https://github.com/dsp-uga/team-andromeda-p1/wiki/Naive-Bayes#naive-bayes-classifier) with several preprcessing methods. There is a brief flow of what we did:

1. Words Splitting by white space (Optional: Replace double hyphens `--` by white space)
2. Words Tokenizing
3. Punctuations Removing - remove punctuations before and after the words
4. Words Stemming - Lancaster, Porter, or Lemmatizer stemming methods
5. Punctuations Removing again
6. Stopwords Removing
7. Naive Bayes Classifier
8. Prediction

See more details of each section in [WIKI](https://github.com/dsp-uga/team-andromeda-p1/wiki) tab.


#### Data Structure

We expressed the data structure inside RDD for each stage as follows:

- Input data with label

```
RDD([(doc_id_0, document_0, label_0),
     (doc_id_1, document_1, label_1), ...])
```

- Training set after preprocessing

```
RDD([((label_0, word_0), (word_count_0, word_total_count_in_label_0)),
     ((label_0, word_1), (word_count_1, word_total_count_in_label_0)), ...])
```

- Training set after NB classifier

```
rdd_train = [rdd_train_labword_cp, rdd_train_lab_cp0, rdd_train_lab_pp]
rdd_train_labword_cp
  = RDD([((label_0, word_0), cond_prob_of_word_0),
         ((label_0, word_1), cond_prob_of_word_1), ...])
rdd_train_lab_cp0
  = RDD([(label_0, cond_prob_of_count0_in_label_0),
         (label_1, cond_prob_of_count0_in_label_1), ...])
rdd_train_lab_pp
 = RDD([(label_0, prior_prob_in_label_0),
        (label_1, prior_prob_in_label_1), ...])
```

- Testing set before prediction

```
RDD([((label_0, word_0), doc_id_0),
     ((label_0, word_1), doc_id_0), ...])
```

- Training and Testing set during prediction

```
RDD([((doc_id_0, label_0), (cond_prob_0, prior_prob_0)),
     ((doc_id_0, label_0), (cond_prob_1, prior_prob_0)), ...])
```

You can read the script [p1.py](https://github.com/dsp-uga/team-andromeda-p1/blob/master/src/p1.py) to know more details of how the formats work for NB classifier by the comments we left in the codes.

#### Preprocessing
1. Join Label and Content
```
rdd = rdd_train_data.join(rdd_train_label).map(lambda x: x[1])
```
Join two rdds that have been zipped with index to link labels with documents

2. Duplicate Documents with multiply labels
```
rdd = rdd.map(lambda x: (x[0], x[1].split(',')))
rdd = rdd.flatMapValues(lambda x: x).filter(lambda x: 'CAT' in x[1]).map(lambda x: (x[1],x[0]))
```

3. Tokenize Words and Remove "&quot"
```
def tokenize_words(no_quot_words):
    no_quot_words = no_quot_words.split("&quot") #.replace(".", " ").replace("--", " ")
    new = []
    for item in no_quot_words:
        new.extend(item.split(" "))
    return new
```
We reomve "&quot" by replacing it with space. Therefore, words can be easily splitted by .split(" ")。
We also did experments on removing "." and/or "--".

4. Remove Punctuation
```
def remove_punctuation_from_end(word):
    punctuation = PUNC.value
    if len(word)>0 and word[0] in punctuation:
        word = word[1:]
    if len(word)>0 and word[-1] in punctuation:
        word = word[:-1]
    return word

def check_punctuation(word):
    punctuation = PUNC.value
    while len(word)>0 and (word[0] in punctuation or word[-1] in punctuation):
        word = remove_punctuation_from_end(word)
    return word
```
This part only removes all the punctuations that's either in the end or in the beginning. But it will not remove punctuation inside a word, preventing words like "we're" to be splitted.

5. Stemming
```
def cleanup_word(word):
    w = check_punctuation(word)
    lancaster_stemmer = LancasterStemmer()

#    ps = PorterStemmer()
    wnl = WordNetLemmatizer()
#    w = wnl.lemmatize(w.lower())
#    w = ps.stem(w.lower())
#    w = ps.stem(wnl.lemmatize(w.lower()))
    w = lancaster_stemmer.stem(wnl.lemmatize(w.lower()))
    w = check_punctuation(w)
    return w
```
This part is mainly for stemming. But you'll notice we did check and/or remove punctuaction twice. The former one is for cleaning the word for the use of stemmer. If a word like ``` cars. ``` has punctuaction with it, the stemmer will not stem the word. If the word is cleaned, namely if it becomes ``` cars ```, the stemmer will then stem the word, making it to be ```car```, which is what we want.

Experiments that we did here are using different or combination of stemmers and lemmatizer.

#### Naive Bayes Classifier

For each label, we kept those words not in the label but in other labels with count 0. Conditional probability of word i, given label k, are calculated by the word count in the label k divided by the total word count in the label k with laplace smoothing.
For example, the conditional probability of word `happy`, given the catogory `CCAT` is calculated by following equation:

<p align = "center">
<img align = "center"  src="https://latex.codecogs.com/gif.latex?\frac{\text{count}_\text{happy}&plus;1}{\text{total-count}_\text{CCAT}&plus;\text{V}}" title="\frac{\text{count}_\text{happy}+1}{\text{total-count}_\text{CCAT}+\text{V}}" />
</p>

where the value V is the distinct amount of words in training data without considering the label.
More details about naive Bayes theory and laplace smoothing are in [WIKI](https://github.com/dsp-uga/team-andromeda-p1/wiki) tab.


#### Prediction

The prediction of each document in testing data are selected by the category with largest value of sum of conditional probabilities and prior probability after log transformation. For example, in document 1 of `vsmall` data, we'll have to calculate following values of each category:

<p align="center">
<img src="https://latex.codecogs.com/gif.latex?log(P(\text{category}_k))&space;&plus;&space;\sum_i&space;log({P(\text{word}_i|\text{category}_k)})" title="log(P(\text{category}_k)) + \sum_i log({P(\text{word}_i|\text{category}_k)})" />
</p>

Since we got -436.92 for category MCAT, -429.91 for category CCAT, -447.68 for category GCAT, and -441.24 for category ECAT, then we assigned CCAT for this document 1. Once the predicted category is one of the category of the document category list, we regarded it as success prediction. The classifier will automatically output a prediction list file (`pred_test_vsmall.txt`) and prediction accuracy if the testing label file exists.

## Test results

We tried several different situations in preprocessing section and the results are as follows:

|Tokenizing           |Stemming                    |Accuracy|
|---------------------|----------------------------|--------|
|Remove double hyphens|Lemmatizer                  |94.51%  |
|Remove double hyphens|Lemmatizer + Porter         |94.21%  |  
|                     |Lemmatizer + Lancaster      |94.04%  |
|                     |Porter                      |94.19%  |
|                     |Lemmatizer                  |94.52%  |
|Remove dots          |Lemmatizer                  |94.28%  |

Therefore, we recommend using only Lemmatizer words stemming.

## Future Research

Since Naive Bayes classifier considers count for calculating the probabilities, it is tricky to implement TF-IDF in NB classifier. However, TF-IDF (Term Frequency Inverse Document Frequency) is reasonable to scale important words in each category. To improve this classifier, we expect to further the project by implementing TF-IDF to Logistic Regression classifier or K Nearest Neighbor classifier.

## Issues

You might encounter different issues when running this classifier on local machine and Google Cloud Platform for the first time. See the following list of issues and solutions or more other issues are documented in the [ISSUES](https://github.com/dsp-uga/team-andromeda-p1/issues) tab:

**Local Machine (Windows)**

- [[Pycharm] Error: too many values to unpack](https://github.com/dsp-uga/team-andromeda-p1/issues/17)
- [[Pyspark] Error: cannot find the file specified](https://github.com/dsp-uga/team-andromeda-p1/issues/18)
- [Worker and Driver has different version](https://github.com/dsp-uga/team-andromeda-p1/issues/19)

**Google Cloud Platform**

- [ImportError: No module named nltk](https://github.com/dsp-uga/team-andromeda-p1/issues/33)
- [ImportError: No module named nltk.stem.wordnet](https://github.com/dsp-uga/team-andromeda-p1/issues/35)
- [Resource wordnet not found](https://github.com/dsp-uga/team-andromeda-p1/issues/36)
- [Name node is in safe mode](https://github.com/dsp-uga/team-andromeda-p1/issues/37)

## References

Some issues were resolved using the help of the almighty Stackoverflow.

[1] [Naive Bayes Text Classification - Stanford University](https://nlp.stanford.edu/IR-book/html/htmledition/naive-bayes-text-classification-1.html)

[2] [A Review of Machine Learning Algorithms for
Text-Documents Classification](http://www.jait.us/uploadfile/2014/1223/20141223050800532.pdf)



## Authors

* **Weiwen Xu** - [WeiwenXu21](https://github.com/WeiwenXu21)
* **I-Huei Ho** - [melanieihuei](https://github.com/melanieihuei)
* **Nihal Soans** - [nihalsoans91](https://github.com/nihalsoans91)

See the [CONTRIBUTORS](https://github.com/dsp-uga/team-andromeda-p1/blob/master/CONTRIBUTORS.md) file for details.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details
