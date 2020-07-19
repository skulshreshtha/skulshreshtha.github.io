---
layout: post
title: Building spam detector app using Flask
subtitle: A simple spam predictor app built using Python and Flask
cover-img: /assets/img/spam-img.jpg
gh-repo: skulshreshtha/Spam-Detector-App
gh-badge: [star, fork, follow]
tags: [SpamDetection, MachineLearning, Flask, Python, NLP]
comments: true
---

**Spam detection** is one of the basic classification problems which have been solved very efficiently by applying machine learning concepts. As most projects limit themselves to the normal routine of machine learning,i.e. training models, making predictions, checking accuracy, and iterating these steps to achieve the right mix, I decided to take a step further and convert the output of this process into a basic standard tool which can be used by uninitiated user to identify spam/ham from their messages. It is part of the wide NLP application gamut, where you derive insights from textual information.

This project utilizes the SMS dataset taken from the [SMS Spam Collection Dataset](https://www.kaggle.com/uciml/sms-spam-collection-dataset). For this project, I  am starting with a Jupyter Notebook as it eases the data cleansing, model training & testing pipeline workload by not requiring you to re-run the entire script every time. Once we have settled on a model configuration, we can create separate scripts for deploying it in Flask.

### Importing the basic necessities


```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

%matplotlib inline
sns.set_style("whitegrid")
```

### Reading the dataset


```python
sms = pd.read_csv("spam.csv",encoding='latin-1')
sms.columns = ['label', 'message']

sms.head()
```




<div class="table-container">
<table>
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>label</th>
      <th>message</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ham</td>
      <td>Go until jurong point, crazy.. Available only ...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ham</td>
      <td>Ok lar... Joking wif u oni...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>spam</td>
      <td>Free entry in 2 a wkly comp to win FA Cup fina...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ham</td>
      <td>U dun say so early hor... U c already then say...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ham</td>
      <td>Nah I don't think he goes to usf, he lives aro...</td>
    </tr>
  </tbody>
</table>
</div>



### Exploratory Data Analysis


```python
sms.describe()
```




<div class="table-container">
<table>
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>label</th>
      <th>message</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>5572</td>
      <td>5572</td>
    </tr>
    <tr>
      <th>unique</th>
      <td>2</td>
      <td>5169</td>
    </tr>
    <tr>
      <th>top</th>
      <td>ham</td>
      <td>Sorry, I'll call later</td>
    </tr>
    <tr>
      <th>freq</th>
      <td>4825</td>
      <td>30</td>
    </tr>
  </tbody>
</table>
</div>




```python
sms['label'].value_counts()
```




    ham     4825
    spam     747
    Name: label, dtype: int64



So we have `4825` ham and `747` spam messages


```python
sms['label_num'] = sms['label'].map({'ham': 0, 'spam': 1})
sms.head()
```




<div class="table-container">
<table>
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>label</th>
      <th>message</th>
      <th>label_num</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ham</td>
      <td>Go until jurong point, crazy.. Available only ...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ham</td>
      <td>Ok lar... Joking wif u oni...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>spam</td>
      <td>Free entry in 2 a wkly comp to win FA Cup fina...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ham</td>
      <td>U dun say so early hor... U c already then say...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ham</td>
      <td>Nah I don't think he goes to usf, he lives aro...</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



Let's add a basic feature 'message length' to our dataset. You may find many advanced feature engineering techniques for spam classification problem but at the end it all boils down to your domain knowledge and expected accuracy of results. For now, we will add only this feature


```python
sms['message_len'] = sms.message.apply(len)
sms.head()
```




<div class="table-container">
<table>
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>label</th>
      <th>message</th>
      <th>label_num</th>
      <th>message_len</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ham</td>
      <td>Go until jurong point, crazy.. Available only ...</td>
      <td>0</td>
      <td>111</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ham</td>
      <td>Ok lar... Joking wif u oni...</td>
      <td>0</td>
      <td>29</td>
    </tr>
    <tr>
      <th>2</th>
      <td>spam</td>
      <td>Free entry in 2 a wkly comp to win FA Cup fina...</td>
      <td>1</td>
      <td>155</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ham</td>
      <td>U dun say so early hor... U c already then say...</td>
      <td>0</td>
      <td>49</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ham</td>
      <td>Nah I don't think he goes to usf, he lives aro...</td>
      <td>0</td>
      <td>61</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.figure(figsize=(12, 8))

sms[sms.label=='ham'].message_len.plot(bins=35, kind='hist', color='blue', 
                                       label='Ham messages', alpha=0.6)
sms[sms.label=='spam'].message_len.plot(kind='hist', color='red', 
                                       label='Spam messages', alpha=0.6)
plt.legend()
plt.xlabel("Message Length")
```

![png](/assets/img/spam_output_11_1.png)


It appears that spam messages are usually longer than ham messages. Let's check out each category separately to get a numeric view on this


```python
sms[sms.label=='ham'].describe()
```




<div class="table-container">
<table>
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>label_num</th>
      <th>message_len</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>4825.0</td>
      <td>4825.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>0.0</td>
      <td>71.023627</td>
    </tr>
    <tr>
      <th>std</th>
      <td>0.0</td>
      <td>58.016023</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.0</td>
      <td>2.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>0.0</td>
      <td>33.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>0.0</td>
      <td>52.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>0.0</td>
      <td>92.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>0.0</td>
      <td>910.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
sms[sms.label=='spam'].describe()
```




<div class="table-container">
<table>
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>label_num</th>
      <th>message_len</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>747.0</td>
      <td>747.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>1.0</td>
      <td>138.866131</td>
    </tr>
    <tr>
      <th>std</th>
      <td>0.0</td>
      <td>29.183082</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.0</td>
      <td>13.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>1.0</td>
      <td>132.500000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>1.0</td>
      <td>149.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>1.0</td>
      <td>157.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>1.0</td>
      <td>224.000000</td>
    </tr>
  </tbody>
</table>
</div>



As we can see above, 75% of the ham messages have length less than 92 characters whereas 75% of the spam messages have length more than 132 characters. So basically, if you just judge a message by its length, you would be right 75% of the time.
**Pretty Interesting, isn't it?**

### Text Pre-Processing

Before we get into any kind of machine learning classification models, let's clean our messages of `punctuation` and `stopwords`. The former is pretty obvious to everyone, but for those who are not familiar with the latter one, /*stopwords*/ are commonly occuring words found in any launguage text which help form continuity and structure but attribute no particular or unique meaning to the sentence, for e.g. "The","is","at", etc. Python's `nltk` library has a list of stopwords that we can use to filter out these.


```python
import string
from nltk.corpus import stopwords

def text_process(message):
    """
    Takes in a string of text and remove punctuation & stopwords
    Return type: string
    Returns: String of cleaned message
    """
    STOPWORDS = stopwords.words('english') + ['u', 'ü', 'ur', '4', '2', 'im', 'dont', 'doin', 'ure']
    # Check characters to see if they are in punctuation
    nopunc = [char for char in message if char not in string.punctuation]

    # Join the characters again to form the string.
    nopunc = ''.join(nopunc)
    
    # Now just remove any stopwords
    return ' '.join([word for word in nopunc.split() if word.lower() not in STOPWORDS])
```

Let's apply the above function and create a new field having clean messages


```python
sms['clean_message'] = sms['message'].apply(text_process)
```


```python
sms.head()
```




<div class="table-container">
<table>
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>label</th>
      <th>message</th>
      <th>label_num</th>
      <th>message_len</th>
      <th>clean_message</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ham</td>
      <td>Go until jurong point, crazy.. Available only ...</td>
      <td>0</td>
      <td>111</td>
      <td>Go jurong point crazy Available bugis n great ...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ham</td>
      <td>Ok lar... Joking wif u oni...</td>
      <td>0</td>
      <td>29</td>
      <td>Ok lar Joking wif oni</td>
    </tr>
    <tr>
      <th>2</th>
      <td>spam</td>
      <td>Free entry in 2 a wkly comp to win FA Cup fina...</td>
      <td>1</td>
      <td>155</td>
      <td>Free entry wkly comp win FA Cup final tkts 21s...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ham</td>
      <td>U dun say so early hor... U c already then say...</td>
      <td>0</td>
      <td>49</td>
      <td>dun say early hor c already say</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ham</td>
      <td>Nah I don't think he goes to usf, he lives aro...</td>
      <td>0</td>
      <td>61</td>
      <td>Nah think goes usf lives around though</td>
    </tr>
  </tbody>
</table>
</div>



Let's take a look at what are the most commonly occuring words in each category of messages


```python
from collections import Counter

# Splitting sentences into tokens
ham_tokenized = sms[sms.label=='ham']['clean_message'].apply(lambda x: [word.lower() for word in x.split()])
spam_tokenized = sms[sms.label=='spam']['clean_message'].apply(lambda x: [word.lower() for word in x.split()])

# Creating counter objects for keeping track of token frequency
ham_words = Counter()
spam_words = Counter()

# Iterating and updating counter for all messages in the category
for msg in ham_tokenized:
    ham_words.update(msg)
for msg in spam_tokenized:
    spam_words.update(msg)
    
print("Top 10 words in ham messages are:\n",ham_words.most_common(10))
print("\nTop 10 words in spam messages are:\n",spam_words.most_common(10))
```

    Top 10 words in ham messages are:
     [('get', 303), ('ltgt', 276), ('ok', 272), ('go', 247), ('ill', 236), ('know', 232), ('got', 231), ('like', 229), ('call', 229), ('come', 224)]
    
    Top 10 words in spam messages are:
     [('call', 347), ('free', 216), ('txt', 150), ('mobile', 123), ('text', 120), ('claim', 113), ('stop', 113), ('reply', 101), ('prize', 92), ('get', 83)]
    

If we observe the above results, we can see that there are different forms of the same root word getting repeated, such as "get"-"got","come"-"go". As these hold similar significance in a message from a spam/ham classification perspective, let's just reduce every word to its root word (aka `stem`) using the `Porter Stemmer`. If you need more background on the difference between `lemmatization` and `stemming`, when to use which, and how to implement it, feel free to browse this link from [Datacamp](https://www.datacamp.com/community/tutorials/stemming-lemmatization-python)

Let's redefine out `text_process` function incorporating Stemming in there. This time we will also use nltk's `tokenize` function to convert strings into tokens instead of our normal split methodology


```python
from nltk.tokenize import word_tokenize
from nltk.stem import PorterStemmer

def text_process_stem(message):
    """
    Takes in a string of text and performs the following steps:
    1. Remove punctuation
    2. Tokenize
    3. Remove stopwords
    4. Stems words to their root forms
    Return type: string
    Returns: String of cleaned & stemmed message
    """
    STOPWORDS = stopwords.words('english') + ['u', 'ü', 'ur', '4', '2', 'im', 'dont', 'doin', 'ure']
    # Check characters to see if they are in punctuation
    nopunc = [char for char in message if char not in string.punctuation]

    # Join the characters again to form the string
    nopunc = ''.join(nopunc)
    
    # Instantiating a PorterStemmer object
    porter = PorterStemmer()
    token_words = word_tokenize(nopunc)
    stem_message=[]
    for word in token_words:
        stem_message.append(porter.stem(word))
        stem_message.append(" ")
    return ''.join(stem_message)
```


```python
sms['clean_message_stem'] = sms['message'].apply(text_process)
sms.head()
```




<div class="table-container">
<table>
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>label</th>
      <th>message</th>
      <th>label_num</th>
      <th>message_len</th>
      <th>clean_message</th>
      <th>clean_message_stem</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ham</td>
      <td>Go until jurong point, crazy.. Available only ...</td>
      <td>0</td>
      <td>111</td>
      <td>Go jurong point crazy Available bugis n great ...</td>
      <td>Go jurong point crazy Available bugis n great ...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ham</td>
      <td>Ok lar... Joking wif u oni...</td>
      <td>0</td>
      <td>29</td>
      <td>Ok lar Joking wif oni</td>
      <td>Ok lar Joking wif oni</td>
    </tr>
    <tr>
      <th>2</th>
      <td>spam</td>
      <td>Free entry in 2 a wkly comp to win FA Cup fina...</td>
      <td>1</td>
      <td>155</td>
      <td>Free entry wkly comp win FA Cup final tkts 21s...</td>
      <td>Free entry wkly comp win FA Cup final tkts 21s...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ham</td>
      <td>U dun say so early hor... U c already then say...</td>
      <td>0</td>
      <td>49</td>
      <td>dun say early hor c already say</td>
      <td>dun say early hor c already say</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ham</td>
      <td>Nah I don't think he goes to usf, he lives aro...</td>
      <td>0</td>
      <td>61</td>
      <td>Nah think goes usf lives around though</td>
      <td>Nah think goes usf lives around though</td>
    </tr>
  </tbody>
</table>
</div>



### Model Build & Testing

The classification alogrithms we would be using for our problem usually work with numerical feature vector. For that reason, we will need to convert our messages into numerical features. There are multiple methods to convert a corpus of text into a numerical vector, simplest of which is `bag-of-words` approach, where each sentence is expressed using a bag of words that we call `vocabulary` and every word occurs a number of times in that sentence. For performing this conversion, we will use the `CountVectorizer` available in scikit-learn.

To perform this modelling, we will be subjecting our data to three sequential steps:
1. **Bag-of-Words creation**: We will use the `CountVectorizer` to create word-message matrix with each element representing the freuquency of occurence for that word-message combination. You can imagine it to look like the following table:

| Word/Message | Message 1 | Message 2 | ... | Message N |
| --- | --- | --- | --- | --- |
| Word 1 | 0 | 1 | ... | 1 |
| Word 2 | 2 | 0 | ... | 0 |
| Word 3 | 1 | 1 | ... | 1 |

2. **Transforming using TF-IDF***: TF-IDF stands for "Term Frequency - Inverse Document Frequency", what it does is it positively weighs every term for its occurence in a message and also negatively weighs that term for occuring frequently in general, which results into a normalized weight associated to every token. Taking from the [official documentation](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.TfidfTransformer.html#sklearn.feature_extraction.text.TfidfTransformer)
> The goal of using tf-idf instead of the raw frequencies of occurrence of a token in a given document is to scale down the impact of tokens that occur very frequently in a given corpus and that are hence empirically less informative than features that occur in a small fraction of the training corpus

3. **Multinomial Naive Bayes Model**: Using the normalized weightages for each word in the corpus as input, we then fit a Naive Bayes classifier model to this data. Naive Bayes works great in spam filtering use case because it assumes that every feature (token in our case) is independently related to the overall outcome. Quoting the official documentation:
> The multinomial Naive Bayes classifier is suitable for classification with discrete features (e.g., word counts for text classification). The multinomial distribution normally requires integer feature counts. However, in practice, fractional counts such as tf-idf may also work.

Lastly, to evaluate the efficacy of our model we will be using the precision score. The reason for choosing precision is that it penalizes the model for false positives, i.e. in our case classifying a normal/important message as Spam and putting it in Junk. To get a good idea about the different evaluation metrics and when to use them, you can refer to this [article](https://towardsdatascience.com/top-10-model-evaluation-metrics-for-classification-ml-models-a0a0f1d51b9)

Let's split the dataset into train and test sets, followed by creating a machine-learning pipeline comprising the above three steps


```python
from sklearn.model_selection import train_test_split
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.feature_extraction.text import TfidfTransformer
from sklearn.naive_bayes import MultinomialNB
from sklearn.pipeline import Pipeline
from sklearn import metrics

X = sms.clean_message
y = sms.label_num
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=1)
pipe = Pipeline([('bow', CountVectorizer()), 
                 ('tfid', TfidfTransformer()),  
                 ('model', MultinomialNB())])
pipe.fit(X_train, y_train)
```




    Pipeline(memory=None,
         steps=[('bow', CountVectorizer(analyzer='word', binary=False, decode_error='strict',
            dtype=<class 'numpy.int64'>, encoding='utf-8', input='content',
            lowercase=True, max_df=1.0, max_features=None, min_df=1,
            ngram_range=(1, 1), preprocessor=None, stop_words=None,
            strip_...ear_tf=False, use_idf=True)), ('model', MultinomialNB(alpha=1.0, class_prior=None, fit_prior=True))])




```python
y_pred = pipe.predict(X_test)

# Evaluating model
print("Accuracy is: {:2f}".format(metrics.accuracy_score(y_test,y_pred)))
print("Precision is: {:2f}".format(metrics.precision_score(y_test,y_pred)))
print("ROC-AUC is: {:2f}".format(metrics.roc_auc_score(y_test,y_pred)))
```

    Accuracy is: 0.966978
    Precision is: 1.000000
    ROC-AUC is: 0.872222
    

That's a good precision but not so good AUC score. Let's try comparing this with logistic regression model


```python
from sklearn.linear_model import LogisticRegression

pipe_log = Pipeline([('bow', CountVectorizer()), 
                 ('tfid', TfidfTransformer()),  
                 ('logmodel', LogisticRegression(solver='liblinear'))])
                     
pipe_log.fit(X_train, y_train)

y_pred = pipe_log.predict(X_test)

# Evaluating model
print("Accuracy is: {:2f}".format(metrics.accuracy_score(y_test,y_pred)))
print("Precision is: {:2f}".format(metrics.precision_score(y_test,y_pred)))
print("ROC-AUC is: {:2f}".format(metrics.roc_auc_score(y_test,y_pred)))
```

    Accuracy is: 0.963388
    Precision is: 0.977778
    ROC-AUC is: 0.865430
    

As we see, logistic regression is coming out to be a better model here rather than Naive-Bayes. We could further tune the count vectorizer we are using or the stemming/lemmatization techniques to increase the efficacy of this model. But for now, let's just use this model and build a web app based on it

**Let's deploy it in Flask now!!**

### Creating Flask App

If you do not have it installed, you will first need to install flask, which you can easily do using `pip install flask`. To get familiar with Flask, and to get started with it, please refer to the [official documentation](https://flask.palletsprojects.com/en/1.1.x/) which preps you quite well to take a stab at it.

Flask requires you to have a basic folder structure setup in your project folder, similar to this screenshot here.

![folder_structure_sample](/assets/img/folder_structure.jpg)

The essential thing to note is having a folder titled `static` which holds the css styling file and any kind of static content to be used on your web-app like photos, videos, etc.
Another folder that you need to have is `templates` which would house the html templates for rendering the different pages on your web-app.

To start with, let's create a python script file 'app.py' in the project folder containing the following building blocks for the flask app

```python
from flask import Flask,render_template,url_for,request

app = Flask(__name__)

@app.route('/')
def home():
	return render_template('home.html')

app.run(debug=True)
```

A quick overview of what we just did in there:
1. Importing required libraries: 
	a. Flask - Pretty obvious what it does
	b. render_template - Renders the HTML pages
	c. url_for - Generates url to a given endpoint with the method provided
	d. request - Enables HTML request methods 
2. Instantiating a flask app object using `app = Flask(__name__)`
3. Create a function for rendering a page
	a. Each function definition is preceeded by `@app.route('<fill_page_route>')` where home page has the route "/" and then other pages have page addresses as "/page1"
	b. The home page function is defined with name 'home'
	c. After putting in the required server-side function for a page in the page function, it is must to use `render_template` to render the page at the end
4. Lastly, we run the app passing turning on the debugging mode 
	
For this basic app, we have created just two pages, where one allows you to enter your custom message and the other one shows you the predicted result. Let's take a brief look at how the HTML templates for these two pages are structured.

#### Home Page

This is how the home page looks when you run the app

![home_page_image](/assets/img/home_page_img.jpg)

If you are familiar with HTML, you must have guessed that the major components here are:
1. /*Input Form*/ - Allowing user to enter custom message
2. /*Submit buttom*/ - Creating a request for Flask app to navigate to the next page and render the prediction results

This is what the code for these components in the file `home.html` looks like:

```
<div class="ml-container">

	<form action="{{ url_for('Predict')}}" method="POST">
		<p>Enter Your Message Here</p>
		<!-- <input type="text" name="comment"/> -->
		<textarea name="message" rows="4" cols="50"></textarea>
		<br/>

		<input type="submit" class="btn-info" value="Predict">
		
	</form>
		
</div>
```
In the form action above, we use the url_for decorator to generate a POST request url for "Predict", which is then captured by the app.route we defined in Flask server to perform the next step

#### Results Page

This is how the results page looks when you push the 'predict' button

![results_page_image](/assets/img/results_page_img.jpg)

For this page, we need a binary classification output (0/1) coming from the server-side created by predicting custom message class using the already trained model. To do this, we first copy the code required for reading in the training data and fitting our model from our Jupyter notebook into the script `app.py`. This is how the script looks after pasting

```python
from flask import Flask,render_template,url_for,request
import pandas as pd
import string
from nltk.corpus import stopwords
from nltk.tokenize import word_tokenize
from nltk.stem import PorterStemmer
from sklearn.model_selection import train_test_split
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.feature_extraction.text import TfidfTransformer
from sklearn.naive_bayes import MultinomialNB
from sklearn.linear_model import LogisticRegression
from sklearn.pipeline import Pipeline


app = Flask(__name__)

# Adding in code from the Jupyter for training the model
sms = pd.read_csv("spam.csv", encoding="latin-1")
sms.columns = ['label', 'message']
# Features and Labels
sms['label_num'] = sms['label'].map({'ham': 0, 'spam': 1})
	
def text_process(message):
	"""
	Takes in a string of text and performs the following steps:
	1. Remove punctuation
	2. Tokenize
	3. Remove stopwords
	4. Stems words to their root forms
	Return type: string
	Returns: String of cleaned & stemmed message
	"""
	STOPWORDS = stopwords.words('english') + ['u', 'ü', 'ur', '4', '2', 'im', 'dont', 'doin', 'ure']
	# Check characters to see if they are in punctuation
	nopunc = [char for char in message if char not in string.punctuation]

	# Join the characters again to form the string
	nopunc = ''.join(nopunc)
	
	# Instantiating a PorterStemmer object
	porter = PorterStemmer()
	token_words = word_tokenize(nopunc)
	stem_message=[]
	for word in token_words:
		stem_message.append(porter.stem(word))
		stem_message.append(" ")
	return ''.join(stem_message)

sms['clean_message'] = sms['message'].apply(text_process)

X = sms.clean_message
y = sms.label_num

pipe = Pipeline([('bow', CountVectorizer()), 
				 ('tfid', TfidfTransformer()),  
				 ('model', LogisticRegression(solver='liblinear'))])

pipe.fit(X, y)
	

@app.route('/')
def home():
	return render_template('home.html')

@app.route('/Predict',methods=['POST'])
def Predict():
	
	if request.method == 'POST':
		message = request.form['message']
		data = [message]
		my_prediction = pipe.predict(data)
	return render_template('result.html',prediction = my_prediction)

app.run(debug=True)
```

In addition to pasting the model code from Jupyter, we have also added an `app.route('/Predict',methods=['POST'])` which captures POST request generated on pressing the 'Predict' button on home page. Followed by that, we define the function for "Predict", which basically just reads in the custom message entered by user, runs the predict function on it and returns our rendered HTML page `result.html`. If you observe, this time we have also passed a parameter `prediction` to the render_template function. This parameter contains the binary classification result (0/1) for the custom message. Our HTML script `result.html` will take this as input and apply a logic to display the right result.

Let's take a look at the logic in `result.html` file

```
{% if prediction == 1%}
<img src="../static/spam.jpg" alt="Spam alert" class="center">
<h2 style="color:red;">Your message is potentially Spam!</h2>
{% elif prediction == 0%}
<img src='../static/ham.jpg' alt='ham cartoon image' class="center">
<h2 style="color:green;">Not a Spam (It is a Ham)</h2>
{% endif %}
```

This is how we use if-else in HTML to render two different results.

![that_was_it](/assets/img/done_spam.gif)

#### Viola! We have completed building a basic spam classifier and deploying it in a web based app.