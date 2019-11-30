
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
 - We pad all sequences to be the same length as the longest one with <start>, <end> tokens.

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

{{< figure src="/img/posts/img_captioning/captioning_loss.png" title="Figure: Loss changes over epochs" >}}


### Caption Evaluation
We now implement the caption evaluator function which similar to the training loop, but without teacher forcing. The input to the decoder at each time step is its previous predictions along with the hidden state and the encoder output. We stop predicting when the model predicts the end token and store the attention weights for every time step. Below is the implementation of evaluate function-

```
def evaluate(image):
    attention_plot = np.zeros((max_length, attention_features_shape))

    hidden = decoder.reset_state(batch_size=1)

    temp_input = tf.expand_dims(load_image(image)[0], 0)
    img_tensor_val = image_features_extract_model(temp_input)
    img_tensor_val = tf.reshape(img_tensor_val, (img_tensor_val.shape[0], -1, img_tensor_val.shape[3]))

    features = encoder(img_tensor_val)

    dec_input = tf.expand_dims([tokenizer.word_index['<start>']], 0)
    result = []

    for i in range(max_length):
        predictions, hidden, attention_weights = decoder(dec_input, features, hidden)

        attention_plot[i] = tf.reshape(attention_weights, (-1, )).numpy()

        predicted_id = tf.argmax(predictions[0]).numpy()
        result.append(tokenizer.index_word[predicted_id])

        if tokenizer.index_word[predicted_id] == '<end>':
            return result, attention_plot

        dec_input = tf.expand_dims([predicted_id], 0)

    attention_plot = attention_plot[:len(result), :]
    return result, attention_plot

# captions on the validation set
rid = np.random.randint(0, len(img_name_val))
image = img_name_val[rid]
real_caption = ' '.join([tokenizer.index_word[i] for i in cap_val[rid] if i not in [0]])
result, attention_plot = evaluate(image)

print ('Real Caption:', real_caption)
print ('Prediction Caption:', ' '.join(result))
plot_attention(image, result, attention_plot)
# opening the image
Image.open(img_name_val[rid])
```
 ### Limitation of the model
 For some images the model fails to generate relevant captions and for some images it just generates repeated words.
 
 {{< figure src="/img/posts/img_captioning/fail_boy.jpg" title="Caption: a boy sits at a boxy and a boxy and a boxy and a boxy and a boxy and a boxy and a boxy and a boxy and a boxy and a boxy and a boxy and a boxy and a boxy and a boxy and a boxy and" >}}
 
 {{< figure src="/img/posts/img_captioning/fail_man_repeated.jpg" title="Caption: a man and a man and a man and a man and a man and a man and a man and a man and a man and a man and a man and a man and a man and a man and a man and a man and a" >}}


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
