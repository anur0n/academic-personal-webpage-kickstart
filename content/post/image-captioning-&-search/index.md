
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

### Preparing dataset 
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
We first split our 30K data set in 80:20 ratio. Then follow below steps-

 - We load the extracted features stored in the respective .npy files and then pass those features through the encoder.
 - The encoder output, hidden state(initialized to 0) and the decoder input (which is the start token) is passed to the decoder.
 - The decoder returns the predictions and the decoder hidden state.
 - The decoder hidden state is then passed back into the model and the predictions are used to calculate the loss.
 - Use teacher forcing to decide the next input to the decoder. Teacher forcing is the technique where the target word is passed as the next input to the decoder.
 - The final step is to calculate the gradients and apply it to the optimizer and backpropagate.

Below graph shows the loss changes over the epochs-

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

## Image Search

Now that we have our captioning model ready and evaluate function implemented, we use this model to generate caption for each of our 10K flickr images and create a mapping file of each images and there captions generated.
After that we create a **TF-IDF** based search index file and use it to search for images with query texts.

To search images with image, we generate a caption for the 'Query image' with this trained model and then use that caption text as query text for searching. For detail about search implementation check [this (Implementing search engine using TF-IDF)] (https://ashaduzzaman-rubel.netlify.com/post/tf-idf_search_implamentation/) article.


## Deploying Image Search/Image Captioning model in PythonAnywhere or other CPU only hosting platforms

The training of the captioning model above takes around 3 hours when run in Google Colab. So we need to use the pretrained models. To do that first we need to save all the trained model weights. We use tensorflow's save_weight(path_to_save, save_format='tf') method.

```
decoder.save_weights('/content/decoder.gru', save_format='tf')
encoder.save_weights('/content/encoder.gru', save_format='tf')
decoder.attention.save_weights('/content/attention.gru', save_format='tf')
```
**Note that** We also need to save **_decoder.attention_** weights too. Otherwise it won't work.

Then, we also need to save the tokenizer and other meta-data the was changed during preprocessing and training. We use dictionary for meta-data and use pickle to save it.
```
with open('/content/tokenizer.gru.pickle', 'wb') as handle:
    pickle.dump(tokenizer, handle, protocol=pickle.HIGHEST_PROTOCOL)

meta_dict = {}
meta_dict['max_length'] = max_length
meta_dict['attention_features_shape'] = attention_features_shape
meta_dict['embedding_dim'] = embedding_dim
meta_dict['units'] = units
meta_dict['vocab_size'] = vocab_size

with open('/content/meta.gru.pickle', 'wb') as handle:
    pickle.dump(meta_dict, handle, protocol=pickle.HIGHEST_PROTOCOL)
```

After all saving, following files will be generated (names will vary depending on the name given during the saving)

```
attention.gru.data-00000-of-00002
attention.gru.data-00001-of-00002
attention.gru.index
checkpoint
decoder.gru.data-00000-of-00002
decoder.gru.data-00001-of-00002
decoder.gru.index
encoder.gru.data-00000-of-00002
encoder.gru.data-00001-of-00002
encoder.gru.index
meta.gru.pickle
tokenizer.gru.pickle

```

Download the generated files and upload it to your pythonAnywhere project directory and load them in new encoder/decoders.

Now when generating caption for a new image we create the Encoder, Decoders that were defined in the image captioning models and load these saved weights to those models. Also we load the meta-dictionary.

```
with open('/content/tokenizer.gru.pickle', 'rb') as handle:
    tokenizer = pickle.load(handle)

with open('/content/meta.gru.pickle', 'rb') as handle:
    meta_dict = pickle.load(handle)

max_length = meta_dict['max_length']
attention_features_shape = meta_dict['attention_features_shape']
embedding_dim = meta_dict['embedding_dim']
units = meta_dict['units']
vocab_size = meta_dict['vocab_size']

encoder_new = CNN_Encoder(embedding_dim)
decoder_new = RNN_Decoder(embedding_dim, units, vocab_size)

image_model = tf.keras.applications.InceptionV3(include_top=False, 
                                                weights='imagenet')

new_input = image_model.input
hidden_layer = image_model.layers[-1].output
image_features_extract_model_new = tf.keras.Model(new_input, hidden_layer)

encoder_new.load_weights('/content/encoder.gru')  # path to the saved weight files 
decoder_new.load_weights('/content/decoder.gru')
decoder_new.attention.load_weights('/content/attention.gru')
```

This way we can save the trained model and load it on pythonAnywhere.

**But now there will be an issue. Our trained model may not generate captions properly.** The problem here is due to below code in the Image captioning tutorial link-

```
def gru(units):
  # If you have a GPU, we recommend using the CuDNNGRU layer (it provides a 
  significant speedup).
  if tf.test.is_gpu_available():
    return tf.keras.layers.CuDNNGRU(units, 
                                    return_sequences=True, 
                                    return_state=True, 
                                    recurrent_initializer='glorot_uniform')
  else:
    return tf.keras.layers.GRU(units, 
                               return_sequences=True, 
                               return_state=True, 
                               recurrent_activation='sigmoid', 
                               recurrent_initializer='glorot_uniform')
```

The problem is we trained our model in google Colab or other computer's with GPU it uses the **CuDNNGRU** layer. But when the model is loaded in pythonAnywhere or other CPU only (or with Cuda incompatible GPUs) machines it uses **GRU** layer. So our saved weights were based on CuDNNGRU model but loaded on GRU model. So the caption generation doesn't work.

One solution is, when training use only **GRU** layer in Colab.

```
def gru(units):
  return tf.keras.layers.GRU(units, 
                               return_sequences=True, 
                               return_state=True, 
                               recurrent_activation='sigmoid', 
                               recurrent_initializer='glorot_uniform')
```

Now save the weights and load again. It should work.

Now you can upload an image in pythonAnywhere and generate a caption with trained model.


## Challenges and Contributions

 - I was strugling to run the caption generation offline, i.e. with trained models even when ran on Google Colab in a separate notebook file. It was challenging to find out the reason why the model was not working. First I saved weights of encoder and decoder. But still it failed to generate caption. Later I discovered that I need to save the attention model inside the decoder also.
 - The second challenge I faced when running on pythonAnywhere. Again it took a lot of time to find out the issue was due to CPU, GPU incompatibility. For which I already mentioned the solution above.
 - I found out a way to run the captioning model in pythonAnywhere wrote the approch above. Hope this helps others.


#### Referrences

*  https://www.tensorflow.org/guide/keras/save_and_serialize
*  https://www.tensorflow.org/tutorials/text/image_captioning
*  https://github.com/tensorflow/tensorflow/blob/r1.13/tensorflow/contrib/eager/python/examples/generative_examples/image_captioning_with_attention.ipynb





#### **[Source codes](https://github.com/anur0n/FashionStoreSearch)**

#### Read More
*  [Implementing search engine using TF-IDF](https://ashaduzzaman-rubel.netlify.com/post/tf-idf_search_implamentation/)
*  [Naive Bayes Text classifier](https://ashaduzzaman-rubel.netlify.com/post/naive-bayes-classifier/)
