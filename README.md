# Neural Networks and Visual Illusions
Final project for neural networks and machine learning

## 1. Introduction
Humans are able to process myriad complex stimuli throughout the day with next to zero challenges, but certain stimuli seem to trick our visual systems. Visual illusions are a fascinating tactic used by researchers to push on the limitations of our visual system. With research in computer vision rising in popularity, studying the similarities and differences between human and computer vision is of increasing importance. Although not there have been very few studies looking into this problem, one study in particular had interestng findings. Gomez-Villa et al. found that convolutional neural networks trained on specific visual tasks, such as deblurring or color constancy, could perform similarly to humans on certain visual illusion tasks (Gomez-Villa et al., 2018). To build off of this work, I hoped to investigate whether a neural network trained on naturalistic images could also be fooled by visual illusions in the same way humans are. In this project, a numerosity illusion was tested on a pretrained VGG16 network (Simonyan & Zisserman, 2015). 

The approximate number system is present in humans and many other animals to help us guess the number of objects in front of us when we do not have sufficient time to count them all. An illusion that works on this system is called the "coherence effect" - visually coherent groups of stimuli appear more numerous than variable groups. For example, a group of items with the same orientation will seem more numerous than a groups of the same number of items with variable orientations (Dewind et al., 2020). Using this illusion on a classifier network seemed like an interesting way to gain insight into how illusions might affect neural networks. 

<p>
<img src="Screenshot (96).png"
     alt="stimulus example"
     height=170
     style="float: left; margin-right: 10px;" />
<img src= "Screenshot (28).png"
     alt="stimulus example"
     height=200
     style="float: left; margin-right: 10px;" />  
<em><br><strong>Figure 1.</strong> Images from Dewind et al., 2020. (Left) Examples of the range of coherent to variable stimuli used in the comparison task. (Right) Data from comparison task. The proportion of trials in which participants chose the right (as opposed to left) stimulus at each numerical ratio sand each difference in orientation coherence. </em>
<p>


## 2. Methods

### Stimuli

The stimuli were created using PsychoPy. The number of items per stimulus image ranged from 20-89. Stimuli used for training had random orientations, ranging from 0-180 (fig 2) and stimuli used for testing had coherent orientations randomly chosen from a list of 4 orientations (0, 45, 90, 135) (fig 3). The training set consisted of 3000 images and the testing set consisted of 2000 images.

<p>
<img src="stim_1_39.jpeg"
     alt="stimulus example"
     height=200
     style="float: left; margin-right: 10px;" />
<img src="stim_1_69.jpeg"
     alt="stimulus example"
     height=200
     style="float: left; margin-right: 10px;" /> 
<img src="stim_2_23.jpeg"
     alt="stimulus example"
     height=200
     style="float: left; margin-right: 10px;" /> 
<img src="stim_9_82.jpeg"
     alt="stimulus example"
     height=200
     style="float: left; margin-right: 10px;" /> 
<em><br><strong>Figure 2.</strong> Examples of variable stimuli. These were used for training set. </em>
<p>

<p>
<img src="teststim_28_62.jpeg"
     alt="stimulus example"
     height=200
     style="float: left; margin-right: 10px;" />
<img src="teststim_29_59.jpeg"
     alt="stimulus example"
     height=200
     style="float: left; margin-right: 10px;" /> 
<img src="teststim_78_80.jpeg"
     alt="stimulus example"
     height=200
     style="float: left; margin-right: 10px;" /> 
<img src="teststim_90_76.jpeg"
     alt="stimulus example"
     height=200
     style="float: left; margin-right: 10px;" /> 
<em><br><strong>Figure 3.</strong> Examples of coherent stimuli. These were used for test set. </em>
<p>
     
### Network Architecture

To start, I used a pretrained VGG net and interrupted it at the 31st layer. I then created a linear decoder from that layer and trained it on my training set images. Finally, I tested performance of the linear decoder on my test set images. For the first iteration of the study, I trained the network over one epoch before testing it, but accuracy was very low (around 0.0155). I decided to add 20 epochs to the training and then test performance on the test set. 

### Creating a Dataset



## 3. Results

## 4. Future Directions

### Varying coherence
One limitation of this study is that I only investigated coherence on a binary (completely coherent or randomly selected orientations). In the future, I would like to varying coherence on a spectrum as Dewind et al. did. This could improve performance of the network and make it more comparable to the human experiment.

### Test set vs Train set
In this experiment, I made the training images all randomly oriented stimuli and the test set all coherently oriented. While this can offer important insights, I believe in order to interpret these findings, we would need to compare them to performance by the network when the trainiing and testing stimuli are both coherent or variable. If the network trained on variable stimuli performs similarly when tested on variable and coherent test stimuli, then we might be able to conclude that the illusion does not work on the network. 


