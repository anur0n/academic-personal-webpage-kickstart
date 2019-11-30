
+++
title = "Implementing Image search feature and Caption generation"
summary = "Adding image search feature with generating caption for an uploaded image and searching over other images based on the caption."
+++
 
<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>
 
Image search can be a very useful feature for many applications. In this article let's look how we can implement an image search feature.
In our search approach we will first generate caption for each image in our [dataset](https://www.kaggle.com/hsankesara/flickr-image-dataset). For caption generation we will follow [this](https://www.tensorflow.org/tutorials/text/image_captioning) tensorflow tutorial. Then we will apply **TF-IDF** indexing over the captions and implement searching.

But before that let's talk about the pre-processing of our data.

## Data Pre-processing
The flickr 30K data set contains 30,000 images. But for my search engine implementation I randomly selected 10,000 images. These 10,000 images has been uploaded to [this](https://github.com/anur0n/flicker-dataset-for-img-captioning) repository searving as a file server.

## Caption generation
We will use the MS-COCO dataset (Containng over 82,000 images and over 400,000 captions), preprocess and cache a subset of images using Inception V3, train an encoder-decoder model, and generates captions on new images using the trained model. We use a subset of 30,000 captions and around 20,000 images for our training.
Before training the model we will perform some preprocessing on the training data set.
 ### Preparing dataset-
 - Download and extract the MS-COCO dataset
 - Store captions and image names in vectors and shuffle the vectors
 - Select first 30K captions from the shuffled vectors
 ### Extracting classification features with InceptionV3 model
 We will classify each image and extract features using the Pretrained (on Imagenet) Inception V3 model. We follow the steps below-
 - Convert the images into InceptionV3's expected format by Resizing the image to **299px x 299px**
 - Preprocess the images using the preprocess_input method to normalize the image so that it contains pixels in the range of -1 to 1 matching the format of the images used to train InceptionV3.
 - Create a tf.keras model where the output layer is the last convolutional layer in the InceptionV3 architecture. The shape of the output of this layer is 8x8x2048.
 - Pre-process each image with InceptionV3 and cache the output to disk in .npy files of each image.
 ### Preprocess and tokenize the captions
 We follow below pre-processing steps:
 - Tokenize the captions. This gives us a vocabulary of all of the unique words in the data.
 - Limit the vocabulary size to the top 5,000 words (to save memory). We'll replace all other words with the token "UNK" (unknown).
 - Create word-to-index and index-to-word mappings.
 - We pad all sequences to be the same length as the longest one.
 ### Captionning Model
 The model architecture is inspired by the Show, Attend and Tell paper.

 - We already have the extracted features from the lower convolutional layer of InceptionV3 giving us a vector of shape (8, 8, 2048).
 - We squash that to a shape of (64, 2048).
 - This vector is then passed through the CNN Encoder (which consists of a single Fully connected layer).
 - The RNN (here GRU) attends over the image to predict the next word.
 ### Training
 - We load the extracted features stored in the respective .npy files and then pass those features through the encoder.
 - The encoder output, hidden state(initialized to 0) and the decoder input (which is the start token) is passed to the decoder.
 - The decoder returns the predictions and the decoder hidden state.
 - The decoder hidden state is then passed back into the model and the predictions are used to calculate the loss.
 - Use teacher forcing to decide the next input to the decoder. Teacher forcing is the technique where the target word is passed as the next input to the decoder.
 - The final step is to calculate the gradients and apply it to the optimizer and backpropagate.


### Inverted Index
An inverted index is a data structure that we build while parsing the documents that we are going to answer the search queries on. Given a query, we use the index to return the list of documents relevant for this query. The inverted index contains mappings from terms (words) to the documents that those terms appear in. Each vocabulary term is a key in the index whose value is its postings list. A term’s postings list is the list of documents that the term appears in.


### TF-IDF
Tf-idf is a weighting scheme that assigns each term in a document a weight based on its term frequency (tf) and inverse document frequency (idf).  The terms with higher weight scores are considered to be more important. It’s one of the most popular weighting schemes in Information Retrieval.


### Term Frequency – tf
Term Frequency is calculated for a term t in document d. It is basically the number of occurrences of the term in the document.

$$ tf\_{t,d} = N\_{t,d} $$

A term appears more in the document it becomes more important, which is logical. However, there is a drawback, by using term frequencies we lose positional information. The ordering of terms doesn’t matter, instead the number of occurrences becomes important. This is known as the bag of words model.

While using term frequencies if we use pure occurrence counts, longer documents will be favored more. So, we length normalize term frequencies. So, the term frequency of a term t in document D now becomes:

$$ tf\_{t,d} = \dfrac{{N\_{t,d}{||D||} $$


### Inverse Document Frequency – idf
We can’t only use term frequencies to calculate the weight of a term in the document, because tf considers all terms equally important. However, some terms occur more rarely and they are more discriminative than others. The idf of a term is the number of documents in the corpus divided by the document frequency of a term.

$$ idf_t = 1 + log\dfrac{N}{df_t} $$

### Tf-idf scoring
We have defined both tf and idf, and now we can combine these to produce the ultimate score of a term t in document d. We represent the document as a vector, with each entry being the tf-idf weight of the corresponding term in the document. The tf-idf weight of a term t in document d is simply the multiplication of its tf by its idf:

$$ tf\mbox{-}idf\_{t,d} = tf\_{t,d} \cdot idf_t $$


## Handling Query
If we compute the cosine similarity between the query vector and all the document vectors, sort them in descending order, and select the documents with top similarity, we will obtain an ordered list of relevant documents to this query.


To Summarize these steps in sudo code:

  Term frequency calculation:
  <pre>
  Normalization = sum(number of terms in doc)^2)
  term-tf-doc = sqrtnumber of times term appears in doc / sqrt(Normalization)
  
  </pre>
  Inverse document frequency for a term calculation:
  <pre>
  term-idf = 1 + log(total number of docs in the dataset / number of docs term occurs in)
  </pre>
  TF-IDF calculation:
  <pre>
  term-doc-score = term-tf-doc * term-idf
  </pre>


Following is the code used to prepare the index with TF-IDF


```
def createTfIdfIndex(self):
    self.prepareParams()

    with open(STYLE_WITH_DESC_N_TITLE, 'r', encoding='latin-1') as csvfile:
      reader = csv.reader(csvfile)

      for rowNo, row in enumerate((reader)):
        docId = row[COL_INDEX_ID]
        terms = self.getTerms(row[COL_INDEX_DESC_TITLE])
        if terms == None:
          continue

        self.titleIndex[docId] = row[COL_INDEX_DISPLAY_NAME]
        self.numberOfDocs += 1

        termDict = {}
        for position, term in enumerate(terms):
          try:
            termDict[term][1].append(position)
          except:
            termDict[term]=[docId, array('I',[position])]

        # Normalize the doc
        norm = 0
        for term, posting in termDict.items():
          norm += len(posting)**2
        norm = math.sqrt(norm)

        #calculate tf and df weights
        for term, posting in termDict.items():
          self.tf[term].append('%.4f' % (len(posting[1])/norm))
          self.df[term] += 1

        for term, posting in termDict.items():
          self.index[term].append(posting)

      self.writeIndexToFile()
      
def writeIndexToFile(self):
    '''write the inverted index to the file'''
    file=open(self.indexFile, 'w')
    print(self.numberOfDocs, file = file)
    self.numberOfDocs = float(self.numberOfDocs)

    for term in self.index.keys():
        postingList=[]
        for posting in self.index[term]:
            docID=posting[0]
            positions=posting[1]
            postingList.append(':'.join([str(docID) ,','.join(map(str,positions))]))
        postingData = ';'.join(postingList)
        tfData = ','.join(map(str, self.tf[term]))
        idfData = '%4f'%(1+np.log(self.numberOfDocs / self.df[term]))

        print('|'.join((term, postingData, tfData, idfData)), end="\n", file = file)
    file.close()
```

### Highlighting the Query terms in result
To highlight the query words in the result, following steps were done.

- Split all terms in the doc
- For each doc terms, applied reduction and checked if it is available in the query term list
- If available replaced the **non reduced doc term** with  **<mark>non reduced doc term</mark>**
- Used a dictionary to avoid replacing multiple time for same doc term

## Contributions

1. Applied **nltk's WordNetLemmatizer**

2. Applied **nltk's stopwords** to reduce terms (In the reference there was manual stopword list which had around 70 less words)

3. Applied query term Highlighting

## Challenges faced

1. My data set had two version one was large(12GB), and another one was small (~300MB). But the smaller one didn't have the description of the products. While hosting in **pythonAnywhere** it only provides 512MB storage, while larger data set was **12GB** in size. To overcome this challenge, I parsed the large data set's json files for each product to extract the description and merged with smaller data set.

2. When the code was hosted in **pythonAnywhere**, there was no starting entry point for the client request. I have precalculated the **index** file(Containing TF, IDF, Postings for all terms and documents). After that I was loading the **index** file in memory, so that when a new query request comes, the results can be returned quickly. But as there was no starting entry point for initial code to run, there was no way to load the **index** _Ahead-of-Time_. 
I used caching this to load the **index** whenever a client request is performed first time after the backend is live. So during every next request, the **index** will remain loaded in memory.


## Stemming and Lemmatization (With vs Without)
* Tried different combination of word reduction. Comparisons are listed below:
<table style="width:100%">
  <tr>
    <th>Type of reduction</th>
    <th>Wordcount after reduction</th>
  </tr>
  <tr>
      <td>No reduction</td>
      <td align = "center">9580</td>
  </tr>
  <tr>
      <td>Only NLTK's stopwords removal</td>
      <td align = "center">9444</td>
  </tr>
  <tr>
      <td>Stopwords + Wordnet Lemmatizer</td>
      <td align = "center">8515</td>
  </tr>
  <tr>
      <td>Stopwords + Porter Stemming</td>
      <td align = "center">7097</td>
  </tr>
  <tr>
      <td>Stopwords + Porter Stemming + Lemmatization</td>
      <td align = "center">7080</td>
  </tr>
  <tr>
      <td>Stopwords + Snowball Stemming</td>
      <td align = "center">7057</td>
  </tr>
</table>


* Without lemmatization and stemming my search engine was not returning any result for queries that doesn't exactly match the data sets. Like for '_Narrowed_', there was no result, although there was products with '_narrow_' term in the description.

* Also without these pre-processing, there were around 25% more terms in the index, which increased the **index** file size by **50%**!! (6 MB without reduction, and ~4MB after reduction)


#### Referrences

*  https://www.tensorflow.org/guide/keras/save_and_serialize
*  https://www.tensorflow.org/tutorials/text/image_captioning
*  https://github.com/tensorflow/tensorflow/blob/r1.13/tensorflow/contrib/eager/python/examples/generative_examples/image_captioning_with_attention.ipynb





#### **[Source codes](https://github.com/anur0n/FashionStoreSearch)**

#### Read More
*  [Naive Bayes Text classifier](https://ashaduzzaman-rubel.netlify.com/post/naive-bayes-classifier/)
