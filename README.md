# Identify High Latitude Dust in given MODIS imagery

## Data Fetching
At first, all the shapefiles are downloaded and saved in the folder `Shapefiles` in proper hierarchal structure. Each shapefile has a different folder.

The XML file from the AWS URL, https://s3.amazonaws.com/hld-datashare, is created and saved as hld.xml. Then the code in **Data Fetching** portion of the notebook is run. After that, we will cut the polygon portion defined in the shapefiles from the API called image. That portion acts as the mask and the masked images are formed using the portion. For creating the masked images, bounding box dimension of the shapefile is used. Caching is done for possible connection errors. Also, the failed API calls (because of file not present error, corrupt file error, etc) are ignored.

## Data Preprocessing
As the pixel values for images and labels range from 1 to 255, the normalization of the data is done by dividing each pixel by 255. As we are not doing much hyperparameters tuning for our task, we have just divided the data into two sets, training and validation. The division ratio is 7:3. Ideally, we would want the aspect ratio for all the training images to be same but the way we downloaded data gives varying aspect ratio. For our purpose, I took the input size as 256 by 256 as the image size for the images vary around that value.

We have also used data augmentation here as that helps to bring variety in the training data. For that and for fetching data into the models, we have written custom generators. The cells in the data preprocessing section can be run sequentially to do the preprocessing and generators construction.

## Applying algorithms
We use U-Net model to perform the image segmentation task as it is especially good with segmentation tasks. Unlike the original model, we will add batch normalization to each of our blocks.

The Keras functional API is used here. It is a powerful API that allows to manipulate tensors and build complex graphs with intertwined datastreams easily. In addition, it makes layers and models both callable on tensors.

We will build these helper functions that will allow us to emsemble our model block operations easily and simply.

We have just used U-net here but we could test with other segmentation algorithms to see how they perform in our task. U-net, an algorithm that has performed really well in many segmentation tasks, is a good place to start but there might be obviously better algorithms and models for our task.

It is easy to define loss and metric with Keras. We can define a function that takes both the True labels for a given example and the Predicted labels for the same given example.

Here, we have used custom loss which has dice loss and cross entropy loss. Dice loss measures overlap between predicted mask and true mask. It was introduced in this (paper)[http://campar.in.tum.de/pub/milletari2016Vnet/milletari2016Vnet.pdf]. 

The algorithm can be applied by following the "Applying algorithms" block sequentially in the notebook.

## Evaluating the model
We see the performance of our model over training and validation set for increasing epochs. Though there is a little decrease in the loss, we would have liked to see more decrease. This is a good starting point for data pipelining and model creation. Trying out different other models and hyperparameters tuning is the next step for the modelling. We use Model callback, ModelCheckpoint, that will save the model to disk after each epoch. We also configure it such that it only saves our highest performing model.

## Visualizing the results
We visualize the results by taking validation examples and see how the segmentation is carried out by our model.

## Possible ways to improve the model
1. We have used simple scaling algorithm to create the masked images from the shapefiles and TIFF images. We can use other more accurate techniques (possibly using the knowledge of different projections for different locations) to extract the portion of dust.
2. We have used one channel for the masked image. If we use three channels for the masked image, we might be able to see the dust portion in predicted mask by some color more clearly. 
3. The downloaded training images have varying aspect ratio. If we could download the images such that aspect ratio remains same across all the images, we could reduce the noise produced by reshaping.
4. Instead of putting data in Google Drive, we could put data in more efficient format (like tf.record) and load from there. That makes the training faster, so allowing us to do more hyperparamter tuning.
5. We could try other image segmentation models like FCN, SCNN, etc and see how they perform.
6. The non-dust portions of the satellite images are varying in color (some with white patches and some with black patches). The variation does not make the uniformity in a non-dust label. It would be great if we can have more masks for non-dust portions (like some mask for white patches, and some mask for dark patches).

---

_All the steps can be run sequentially in the colab notebook. The code is well documented and the colab is divided into portions based on the above machine learning steps._