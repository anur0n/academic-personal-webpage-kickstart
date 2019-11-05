
+++
title = "Text Classification using Naive Bayes Classifier"
summary = "About implementation of Naive Bayes Text classifier to classify fashion products based on product description"
+++
 
Text classification plays an important role in information mining. Basically text classification is identifying the class based on some text (Description, Review etc). There are many algorithms for classification, but for text classification Naive Bayes classification gives overall good performance. I implemented a text classifier using Naive Bayes algorithm to classify the product category based on product description. The classifier is based on [this](https://www.kaggle.com/paramaggarwal/fashion-product-images-small) data set which contains information on thousands of life style products.

$$\gamma_{n} = \frac{ \left | \left (\mathbf x_{n} - \mathbf x_{n-1} \right )^T \left [\nabla F (\mathbf x_{n}) - \nabla F (\mathbf x_{n-1}) \right ] \right |}{\left \|\nabla F(\mathbf{x}_{n}) - \nabla F(\mathbf{x}_{n-1}) \right \|^2}$$

