# image-inpainting
TLDR: Building a decision tree and neural network from scratch in Python to perform image inpainting on two images: a set of leaves and wooden planks (Done for my final project in Machine Learning Principles). 

## The Task

Given two 900-pixel by 900-pixel images(one of the leaves on a tree and another of a wood deck) that have the center 1/9th missing, generate two machine learning systems to fill in that missing portion. 

## Input Space, Output Space, & Preprocessing:
	
Thinking of the input space for this problem, initially, I thought of inputting the entire image 900 by 900 image with the mask and then outputting the filled-in 900 by 900 image. However, this approach has a problem, mainly with input and output size. Going from a masked image to the full image would require a lot of training data to give the model some prior knowledge on what to predict for that massive missing piece. Furthermore, outputting a 900 by 900 image is a rather large output(in NumPy this is a 900 by 900 by 3 array). There are better ways for me to use my data. After thinking through this problem, I decided that the way to approach this problem was to use the 300 by 300 patch of pixels right above the missing patch as the input, and the output would be the row of 300 pixels underneath it. My idea was to train the model using those initial top-center 300 by 300 pixels as my  “x data” and the row beneath that as my “y data.” After training my model, I would start predicting the values of the patch row by row, using the 300 by 300 pixel patch above that row. 

Now, on to whether I am using numerical or categorical data.  I decided to keep the data numerical for my approaches, meaning that I was solving a regression problem. I decided to use regression models for my approaches because this would allow a greater range of colors for the image and give a broader range of possibilities. Another thing I decided to do was split up my data so that I had three different datasets for each image that corresponded to the image's red, green, and blue values. I did this to simplify things so that instead of training and a three-dimensional array, I would be dealing with three two-dimensional arrays. I then divided these arrays by 255 so that the range of the inputs from each array would be from 0 to 1. Traditionally, RGB values are represented by integers ranging from 0 to 255. To turn this range into a range of floats, I divided the R, G, and B arrays by 255. This would make computations smaller, and the regression output would be more helpful.  When inputting the data to train my model, I also slightly changed its structure to make more sense in terms of predicting for the model. What I mean by this is that for my 300 by 300 patch of training data, I wanted to treat each row of that patch as a feature of the data and each column of that data as an observation. This was necessary because my row of “y data” that was under my training data patch would be considered 300 observations, so by using the 300 columns of my training data as observations, I would see how each observation in my y training data was related to the 300 pixels above it. So when inputting my 300 by 300 patch into my models for training, I rotated them by 90 degrees clockwise so that the rows would be turned into columns(features) and the columns would be turned into rows(observations).

So, in terms of my input and output space, my training data is the first batch of 300 by 300 pixels and the row underneath it, as this is the data I train my models on. The testing data would then be the rest of the rows that I predict, which can be generalized to the complete area I fill. If my models overfit on the training data, the overall picture will come out poorly. If my model is overfitted to the one row it is trained on, this model will not generalize well to the rest of the rows. Although I don’t necessarily need more training data/testing data outside of the given masked images, I decided to use another leaves image and wood image, which I then masked myself. I got these images from Google Images by looking for images that had leaves/wood decks and were 900 by 900 pixels large. This worked for the leaves image, however, for the wood image, I had to find a larger image and crop down to 900 by 900 pixels. These images would allow me to see how my model performs as I know what the output for these images should look like. 

## The Models

As I said, I am solving a regression problem since my output is numerical. So my model space consists of using regression models. This includes simple linear regression, regression trees, neural networks, etc. For my two models, I decided to use regression trees(because of the tree-based system constraint of the problem), as well as a simple neural network. More importantly, the loss for a regression problem such as this is mean squared error, i.e., the mean of the squared difference between the predicted and actual values. In this case for training error, it would be the mean of the squared difference between the predicted row of pixels and the actual row of pixels. For testing data, it would be the mean of the squared difference between the predicted image and the actual image. Another note for the loss is that I scaled the predicted and actual values by 255 again as having values between 0 and 1 results in small errors in the first place, so multiplying the arrays by 255 again gives me a broader view of the error. 

My first model is the regression tree (decision tree for numerical response). The regression tree model takes the training x data matrix and y data, and builds a decision tree where decisions(splitting the given data) are made by seeing if a value for a specified feature is greater than or less than some threshold value. The specified feature is found by looking for the feature of the data matrix most correlated with the y data. The threshold value for the feature with the highest correlation is found by finding the threshold value that gives the smallest total error from the data after splitting. After training the regression tree this way, I can pass my image patches through the tree to get my prediction. I built three regression trees for my input data of three different R, G, and B matrices for each image. So each regression tree predicts values for the image’s red, green, and blue components. From these red, green, and blue regression trees, I would then iteratively start predicting each row of the missing patch using the 300 by 300 pixel patch above that row. When training my trees, I noticed that it was better to use a shorter tree(maximum depth of 3 and minimum sample size of 5). With a regression tree, it is really easy to overfit the model since a larger, more complex tree could result in a training error of 0, this tree would not generalize well to other data. So with my best regression tree, for the leaves image I found from Google I got a training error of around 550-600, and the testing error was around 600. For the wood image I found from Google I got a training error of around 50-100, and the testing error was around 50. 

my second model is a simple neural network. The structure of this network is based on the autoencoder neural network done previously in class. Essentially, I start out with an input layer of 300 features plus the bias, each corresponding to a feature from the 300 by 300 image patch. Then using a tanh activation function, these inputs are linearly combined to produce a hidden layer of some size k. I wanted k to be a number larger than 300, as this would mean that I am trying to pull more information from the input to produce the final output. Then from the hidden layer, I linearly combine the inputs from the hidden layer without an activation function to produce one output node. Essentially, I use the 300 features to generate one predicted observation using this neural network. In terms of building the system and predicting this image data, I essentially repeated the same process as the regression tree model, i.e., building three neural networks for the training data's red, green, and blue components. Then using these three neural networks, I would predict row by row the R, G, and B values for the missing patch using the 300 by 300 pixel patch above the row. It is possible to overfit the neural network like the decision tree. To combat this, I used early termination, which essentially meant I played with the number of epochs I trained the model for to see what the best balance of training vs testing error was. However, something weird I found when working with my neural network models was that the individual training errors were several times larger than the testing error and the training error from the regression tree models. Although the training error was very large, the testing error on the predicted images was actually smaller than the testing error from the regression tree models. I could not figure out why this was the case, and ultimately I settled to use a hidden layer of size 400, a learning rate of 0.0001, and training the network for 750 epochs. Another thing that was a problem was that when predicting using these neural networks, I saw that I got values less than 0, which does not work for RGB values. So in my code, I had to make sure I accounted to set negative values equal to 0, and values greater than 1 to 1. Using these parameters and changes, for my images from Google, I got a training error of around 2000 to 3000 and a testing error of 481 for the leaves image, and a training error of around 250 to 600, and a testing error of 43 for the wood image.


