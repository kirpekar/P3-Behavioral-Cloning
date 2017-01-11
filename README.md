## Model architecture:

Because it is relatively easy to implement new models in Keras, I experimented with a few different models. The two models that worked relatively well are as follows:

Model (a)
Input Images --> 4 layer CNNs --> 1 fully connected layer --> final output

Model (b), similar, but not exactly, to the NVIDIA model
Input Images --> 4 layer CNNs --> 3 fully connected layers --> final output

Both models worked sufficiently well to drive the car across the track. 

Somehow, if I reduced the number of CNNs the performance would decrease. However the number of FC layers did not matter very much.

I used jupyter notebooks to run all this, so I am submitting .ipynb files instead of .py files.

## Training inputs:

Initially I used just the keyboard, but after studying the forums and slack, I used a PS3 controller for finer control.

I used an iterative method to get to the best model, something like this: 

Start with some training data --> Train model --> Save model --> Test 
New training data --> Retrain saved model with smaller LR --> Save model --> Test 
New training data --> Retrain saved model with smaller LR --> Save model --> Test 
Repeat...

My intial data was simply driving around the track 3 times. This got me about 4000 images, it started off the car, but it quickly went off path. 

I then added a few more laps of normal driving - helped. 
Then I added a couple of "left recover laps" and "right recovery laps" - made the car mroe stable. 
Finally I also added 3-4 laps of erratic, zig-zag driving - also helped, slightly. 

Each retrain used a successively smaller learning rate so as to not "mess up" the working parts of the models. 

Finally made it around the lap without running off.

## Input pre-processing:

As I started to feed the model 20k+ images to train, it would slow down drastically, and I was looking for ways to speed it up.

I did a few A/B tests:

Cutting out the top 50 px of the image did not make much of a difference, so I did that to improve speed.

Shrinking down the image by 50% was a huge speed gain, and also did not change the model performance very much.

I included the left and right camera images in addition to the center one. I gave the left image a steering angle offset of +0.08 (turn right if facing left) and vice versa for the right image.

I also mirrored images using .transpose(Image.FLIP_LEFT_RIGHT) and this seemed to cause a huge gain in performance, although it doubled the size of the training set.

## Other model considerations:

Dropout - improved performance significantly, but 25% was about ideal, anything more than that would cause the car to understeer (and the model to underfit).

Learning rate - initially set to 0.001 using the Adam optimizer, but with every successive data set of new images, I dropped it by a factor of ~10. Eventually I set the LR to 1e-6 for fine tuning the performance.

Optimizer - Adam worked much better than SGD, I did not try anything else.

Epochs - I ran many tests from 3 epochs to 1000 epochs. I found inconsistent results, i.e. often smaller number of epochs with larger mse values performed better on the track compared to a large number of epochs with smaller mse values.

## drive.py:

I set the speed to 0.1, which gave me better performance on my model. Also higher speeds caused my computer to freeze up (using a virtual Linux setup on Windows).

I also found it useful to slow down in a turn. So if the steering angle is more than |0.1| I slow down by 50%. This helps the stability of the car and gives it enough time to correct for errors.

## model_01.json:

Does well most places, but comes close to the edge of the track at the sharp right turn after the bridge.

## model_02.json:

Runs a bit wobbly at the start, bouncing left and right, but runs smooth thereafter. Interestingly the second & third laps are smoother than the first.

I left both models running overnight in autonomous mode and the car kept going for 12+ hours.