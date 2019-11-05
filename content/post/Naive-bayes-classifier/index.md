
+++
title = "Text Classification using Naive Bayes Classifier"
summary = "About implementation of Naive Bayes Text classifier to classify fashion products based on product description"
+++
 
Text classification plays an important role in information mining. Basically text classification is identifying the class based on some text (Description, Review etc). There are many algorithms for classification, but for text classification Naive Bayes classification gives overall good performance. I implemented a text classifier using Naive Bayes algorithm to classify the product category based on product description. The classifier is based on [this](https://www.kaggle.com/paramaggarwal/fashion-product-images-small) data set which contains information on thousands of life style products.


## Data Pre-processing
The data set I choose had a csv file with products in the row. It had some small attributes in each column. To search over the properties and product description I combined all the properties and the description in one column. There were some rows with empty description filed. I removed those products.

Next, I removed the most common words which itself doesn't carry any significance like 'a', 'an', 'to', 'for'. These are called stopwords. Next we will use [Lemmatization](https://nlp.stanford.edu/IR-book/html/htmledition/stemming-and-lemmatization-1.html) (Finding the root word of a term) and [Stemming](https://nlp.stanford.edu/IR-book/html/htmledition/stemming-and-lemmatization-1.html) (Trimming the last part of a word based on set of rule) to find similar words.

Following code applies these techniques:

```
import nltk
nltk.download('stopwords')
nltk.download('wordnet')

from nltk.corpus import stopwords
from nltk.stem import PorterStemmer
from nltk.stem import WordNetLemmatizer

def prepareParams(self):
  self.stopwords = set(stopwords.words('english'))
  self.stemmer = PorterStemmer()
  self.lemmatizer = WordNetLemmatizer()
  
def getTerms(self, doc):
  doc = doc.lower()
  doc = re.sub(r'[^a-z0-9 ]',' ',doc) #put spaces instead of non-alphanumeric characters
  terms = doc.split()

  terms = [term for term in terms if term not in self.stopwords]
  terms = [self.lemmatizer.lemmatize(term) for term in terms]
  terms = [self.stemmer.stem(term) for term in terms]
  return terms

```

### Naive Bayes Algorithm
Naive Bayes algorithm uses Bayes theorem. It states that: The probability of a document d being in class c is computed as-

$$ P(c \mid d) = P( c ) . \prod_{k=1}^{n\_{d}} P(t\_{k} \mid c) $$

where $$ P(t\_{k} \mid c) $$ is the conditional probability of term $$ t\_{k} $$ occurring in a document of class c. P( c ) is the prior probability of a document occurring in class c. The class with highest probability is selected as the result.

To create the model we will need the term frequencies for each documents in each class.

### Indexing
For each class we will store each term with it's frequency for all the documents of that class. Given a query, we use the index to return the list of documents relevant for this query.

```
def addToIndex(self, products_of_category, category):
    counter = 0  
    for product in products_of_category: #for every word in preprocessed description
      token_words = self.getTerms(product)
      for token in token_words:
        counter += 1
        if not token in self.index[category]:
          self.index[category][token] = 0
        self.index[category][token]+=1 #increment in its count

  def buildInvertedIndex(self):
    self.index = np.array([defaultdict(int) for i in range(self.categories.shape[0])])
    
    for idx, category in enumerate(self.categories):
      all_category_products = [x for x, label in zip(self.productDescriptions, self.labels) if label == category]

      desc = [category_product for category_product in all_category_products]

      np.apply_along_axis(self.addToIndex,0, desc,idx)
```

### Precalculating the probabilities
Now we will pre calculate the term probabilities using the formula above so that during classification request we can just get the data from this calculation.

During this calculation of probabilities if some term doesn't occur in the index, then the whole probability will become 0 as it is multiplication. To avoid this, we will add a smoothing param = 1 to each term frequency. This is called **_add-one or Laplace smoothing_.**

In above equation, many conditional probabilities are multiplied. This may result in a floating point underflow. To avoid this, instead of multiplying probabilities we will use **logarithm**. The class with the highest log probability score is still the most probable; log(xy) = log(x) + log(y)

Following is the code for computing the probabilities
```
def precalcNBValues(self):
    probability_classes = np.empty(self.categories.shape[0])
    all_words = []
    cat_word_counts = np.empty(self.categories.shape[0])

    for idx, cat in enumerate(self.categories):
        #Calculating prior probability p(c) for each class
        all_category_products = [x for x, label in zip(self.productDescriptions, self.labels) if label == cat]
        probability_classes[idx] = len(all_category_products) / len(self.labels)
        
        #Calculating total counts of all the words of each class 
        cat_word_counts[idx] = np.sum(np.array(list(self.index[idx].values())))+self.smoothing # |v| is remaining to be added
        
        #get all words of this category                                
        all_words+=self.index[idx].keys()
                                                  
    
    #combine all words of every category & make them unique to get vocabulary -V- of entire training set
    
    self.vocab=np.unique(np.array(all_words))
    self.vocab_length=self.vocab.shape[0]
    # print(self.vocab_length)
    #computing denominator value                                      
    denoms=np.array([cat_word_counts[idx]+self.vocab_length+1 for idx,category in enumerate(self.categories)])

    self.cats_info=[(self.index[idx],probability_classes[idx],denoms[idx]) for idx,cat in enumerate(self.categories)]
    self.cats_info=np.array(self.cats_info)
```

<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>



## Handling Classification Request
For classification query we apply the same preprocessing on the terms then calculate probabilities using the above formula.

Following code with comments will explain the steps:

```
def classify(self,test_example):
      likelihood_prob=np.zeros(self.categories.shape[0]) #to store probability w.r.t each class
      
      terms = self.getTerms(test_example)

      for idx,cat in enumerate(self.categories): 
                            
          for test_token in terms: #split the test example and get p of each test word
              #get total count of this test token from it's respective training dict to get numerator value  
              test_token_counts=self.cats_info[idx][0].get(test_token,0)+self.smoothing
              
              #now get likelihood of this test_token word                              
              test_token_prob=test_token_counts/float(self.cats_info[idx][2])
              
              #Use log to prevent underflow!
              likelihood_prob[idx]+=np.log(test_token_prob)
                                            
      post_prob=np.empty(self.categories.shape[0])
        
      for idx,cat in enumerate(self.categories):
          post_prob[idx]=likelihood_prob[idx]+np.log(self.cats_info[idx][1])
    
      return post_prob   
```

## Contributions

1. Applied **nltk's PorterStemmer**

2. Applied **nltk's stopwords** to reduce terms

3. Tried changing the smoothing parameter and evaluated accuracy, precision, recall and F1 score in the range of 100, 10, 1, 0.1, ..., 0.000000001 for smoothing value.

## Challenges faced

1. When the code was hosted in **pythonAnywhere**, there was no starting entry point for the client request. I have precalculated the **index** . After that I was loading the **index** file in memory, so that when a new query request comes, the results can be returned quickly. But as there was no starting entry point for initial code to run, there was no way to load the **index** _Ahead-of-Time_. 
I used caching this to load the **index** whenever a client request is performed first time after the backend is live. So during every next request, the **index** will remain loaded in memory.

2. After hosting the first part (Search engine) in **pythonAnywhere** there were not enough space left for second part (Classifier). I tried compressing the other files.



## Evaluation
In the dataset there were around 44000 items. For the classification purpose we used 60:40 split for Train:Test. We tried to tune the smoothing hyperparameter in range [100, 10, 1, 0.1, 0.01, ...., 0.0000000001] and found below result graph which shows accuracy, precision, recall and F1 mesures:

{{< figure src="/img/posts/naiveBayesClassifier/evaluation_scores.png" title="Figure: Different scores for the classifier with different smoothing on X-Axis." >}}

From the graphs we can see for lower smoothing values the the accuracy and precision increases upto 99% but recall and F1 score has the peak for smoothing value of 1.

So we used **1** for our smoothing value. Which gives **98.6%** accuracy and **81%** F1 score. 




#### Referrences

*  https://machinelearningmastery.com/naive-bayes-classifier-scratch-python/
*  https://towardsdatascience.com/na%C3%AFve-bayes-from-scratch-using-python-only-no-fancy-frameworks-a1904b37222d
*  https://towardsdatascience.com/unfolding-na%C3%AFve-bayes-from-scratch-2e86dcae4b01#08ef
*  https://towardsdatascience.com/train-validation-and-test-sets-72cb40cba9e7
*  https://towardsdatascience.com/multi-class-metrics-made-simple-part-ii-the-f1-score-ebe8b2c2ca1





#### **[Source codes](https://github.com/anur0n/FashionStoreSearch)**

#### Read More
*  [Implementing search engine using tf-idf](https://ashaduzzaman-rubel.netlify.com/post/tf-idf_search_implamentation/)
