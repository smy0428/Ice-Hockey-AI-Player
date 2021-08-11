# Ice-Hockey-AI-Player
### UT Summer 2021 Deep Learning Final Project
### Team Members: Siyuan Shi, Siddhartha Arora

Implementation detail has been omitted in order to preserve the integrity of the assignment

https://user-images.githubusercontent.com/61920576/129111754-168d121f-02a4-45d0-aa83-f15a875284bd.mp4


### 1. Introduction
Upon reading the objective of the problem, we figured out that there were 2 approaches to solving this. One was to apply reinforcement learning and try to imitate the AI and the other was to apply supervised learning to try to figure out the puck location and then build a sophisticated controller around it. After watching the AI play a number of times we thought that the first approach would not be the best because we saw that the AI was not a perfect agent to learn from as in our perspective it was not good enough in playing the game. We hence decided to go with the second approach and that worked well for us.


### 2. Data

The major challenge initially was to figure out what kind of training data is necessary and what kind of labelling would help us best make decisions. Should we try to label the images with the puck's location or should we label actions taken instead?

Our first idea was to collect the images and label them with the corresponding AI actions, and build a model which is able to predict the appropriate action from the image. It sounded promising and straight forward, however as mentioned in the introduction, the AI in the game is not that smart. After some research on the code base of how the AI controller is implemented as well as by watching the AI play, we agreed that the AI was not performing the best actions in our opinion.

Our second idea was to collect the images and label them with the puck position and the kart position in the global view (namely a vector starting from the kart to the puck). We wanted to train a model that could predict this vector (x, y), which could gives us information about the direction and the distance of the puck from the kart. This again seemed to be reasonable, but it not only involved complex co-ordinate calculations, but it also ignored the fact that the puck was sometimes not in the kart’s view, which would make it quite hard to design a proper model on it. 

Our third and the final approach was to collect the images and label them with the normalised / image coordinates of the puck in the kart's view. We still had to deal with the fact that the puck might not be in the kart's view and we handled such cases by converting image to semantic map and clipping the normalized coordinates to [-1,1]. The clipped coordinates helped in implementing the dribbling logic of the puck in the controller as mentioned below and during training this data helped us to train and detect the puck with relative scores and potential positions. An image with puck not visible always lead to a low score.

### 3. Model

We now had the training data with images and their labels being the normalised puck coordinates. We also initially included the box/gift and the nitro locations as labels but we later removed them as they did not turn out to be that useful.

The first idea is to use 3 hidden convolution blocks and a soft-argmax at the end to predict the puck position. However, we faced the problem that it was not easy to deal with cases in which the puck was not in the camera view, as the model always predicted a position and we are not able to output the puck location as being invisible.

Our second idea is to use a fully convolutional segmentation network for object detection to predict a dense heat-map of object centres. This architecture turned out to suite our needs quite well. In order to speed up the process, we decided to scale down the input image from 300 x 400 to 96 x 128 and decreased the channels to 75%, average AI respond time dropped from 0.12s to 0.04s.

### 4. Controller

We implemented various game strategies. Our two players has their different roles in the game; different actions will be taken based on the puck position, for example in front of, alongside, or behind the kart; how to rescue and retreat, dribble and attack in an intelligent way, etc.

### 5. Optimisations

With our first version of the model and the controller, our agent score 3 goals against the blue AI and score 1 goal against the red AI. 

We figured that the training data was produced by the AI karts upon which we had no control and for which the puck was not visible for approximately 50% of the time, and hence the data was not very efficient. Our controller was well built till this time and we were chasing the puck for the majority of the time. We hence generated new training data by making our agents play against AI and previously only the AI is involved. 

With our second version of the model and the controller, our agent score 3 goals against the blue AI and score 2 goals against the red AI.

After a couple of mock tournaments we were still not satisfied with the number of goals our agent was scoring. As our agent was playing even better, we decided to generate the training data this time by making our agents play against each other and not involve AI at all. At the same time we realised that our model was not adhering to the time limit and hence we decided to make our model thinner by decreasing the number of channels in all layers. By doing so we were able to decrease the overall runtime to approximately 37% of our earlier time. 

With our latest version of the model and the controller, our agent score 4 goals against the blue AI and score 3 goals against the red AI.


