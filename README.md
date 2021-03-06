# Neural Networks and Visual Illusions
Final project for neural networks and machine learning

## 1. Introduction
Humans are able to process myriad complex stimuli throughout the day with next to zero challenges, but certain stimuli seem to trick our visual systems. Visual illusions are a fascinating tactic used by researchers to push on the limitations of our visual system. With research in computer vision rising in popularity, studying the similarities and differences between human and computer vision is of increasing importance. Although not there have been very few studies looking into this problem, one study in particular had interestng findings. Gomez-Villa et al. found that convolutional neural networks trained on specific visual tasks, such as deblurring or color constancy, could perform similarly to humans on certain visual illusion tasks (Gomez-Villa et al., 2018). To build off of this work, I hoped to investigate whether a neural network trained on naturalistic images could also be fooled by visual illusions in the same way humans are. In this project, a numerosity illusion was tested on a pretrained VGG16 network (Simonyan & Zisserman, 2015). 

The approximate number system is present in humans and many other animals to help us guess the number of objects in front of us when we do not have sufficient time to count them all. An illusion that works on this system is called the "coherence effect" - visually coherent groups of stimuli appear more numerous than variable groups. For example, a group of items with the same orientation will seem more numerous than a groups of the same number of items with variable orientations (Dewind et al., 2020). Using this illusion on a classifier network seemed like an interesting way to gain insight into how illusions might affect neural networks. 

<p>
<img src="Screenshot (96).png"
     alt="stimulus example"
     height=160
     style="float: left; margin-right: 10px;" />
<img src= "Screenshot (28).png"
     alt="stimulus example"
     height=180
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
<img src="teststim_28_62.jpeg"
     alt="stimulus example"
     height=200
     style="float: left; margin-right: 10px;" />
<em><br><strong>Figure 3.</strong> Examples of coherent stimuli. These were used for test set. </em>
<p>   

### Creating a Dataset

After creating the stimuli, I had to add them into the network by creating a Dataset. The outputs of my dataset were "images" (stimulus images themselves), "number_objects" (the actual number of items in the image, with a range of 20-89), and "label" (the number of objects minus 20, creating a range of 0-70).
```
class test_orientations(Dataset):
  def __init__(self, csv_file, root_dir, transform=None):
        """
        Args:
            csv_file (string): Path to the csv file with annotations.
            root_dir (string): Directory with all the images.
            transform (callable, optional): Optional transform to be applied
                on a sample.
        """
        self.test_ori_frame = pd.read_csv(csv_file)
        self.root_dir = root_dir
        self.transform = transform
        
  def __len__(self):
    return len(self.test_ori_frame)
    
  def __getitem__(self, idx):
    if torch.is_tensor(idx):
      idx = idx.tolist()

    img_name = os.path.join(self.root_dir,
                                self.test_ori_frame.iloc[idx, 0])
    image = io.imread(img_name)
    number_objects = self.test_ori_frame.iloc[idx, 1]
    number_objects = np.array([number_objects])
    number_objects = number_objects.astype('float')#.reshape(-1, 2)
    label = self.test_ori_frame.iloc[idx, 2]
    label = np.array([label])
    label = label.astype('int').squeeze()
    sample = {'image': image,'number_objects': number_objects, 'label':label}

    if self.transform:
      image = self.transform(image) 
    sample = {'image': image, 'number_objects': number_objects,'label':label} 
    return image, number_objects, label
```

### Network Architecture

To start, I used a pretrained VGG net and interrupted it at the 31st layer. I then created a linear decoder from that layer and trained it on my training set images. Finally, I tested performance of the linear decoder on my test set images. For the first iteration of the study, I trained the network over one epoch before testing it, but accuracy was very low (around 0.0155). I decided to add 20 epochs to the training and then test performance on the test set. Below is the training and testing loop along with the calculation of accuracy. 

```
acc_batch = []
acc_hist_train = []
acc_hist_test = []

for epoch in range(20):
  lin.train()
  torch.cuda.empty_cache()
  for data, num_objects, target in tqdm.tqdm(iter(train_loader)):   
    loss_ = train_step(data, target, lin, opt, crossentloss)    
  acc_batch = []
  for data, num_objects, target in tqdm.tqdm(iter(train_loader)):
    output=model(data.cuda())
    flattenedoutput = torch.flatten(output.detach(),1)
    y = lin(flattenedoutput)
    acc_batch.append(torch.mean((target.cuda() == y.argmax(1)).float()))
  acc_hist_train.append(torch.mean(torch.FloatTensor(acc_batch)))  

  acc_batch = []
  lin.eval()
  for data, num_objects, target in tqdm.tqdm(iter(test_loader)):
    output=model(data.cuda())
    flattenedoutput = torch.flatten(output.detach(),1)
    y = lin(flattenedoutput)
    acc_batch.append(torch.mean((target.cuda() == y.argmax(1)).float()))
  acc_hist_test.append(torch.mean(torch.FloatTensor(acc_batch)))
  print('Accuracy:', torch.mean(torch.FloatTensor(acc_batch)))
```


## 3. Results
After 20 epochs of training, the network performs very well on the training set (with variable orientations), but very poorly on the testing set (with coherent orientations) (fig 4). As shown in the figure below, the training accuracy is close to 100% after just 10 epochs. The testing accuracy on the other hand vacilates around 5% over the 20 epochs, indicating no improvement in test accuracy as a result of training.
<p>
<img src="results_nn.png"
     alt="stimulus example"
     height=200
     style="float: left; margin-right: 10px;" /> 
<em><br><strong>Figure 4.</strong> Preliminary results from training over 20 epochs. Orange is training and blue is testing. </em>
<p>

So far, the results are just preliminary, but they have given me insight into how difficult it is to train a network on images like these. The poor performance could be indicative of an illusive effect, but I will need to do furter evaluations to test if this is actually the case.  

## 4. Future Directions

### Varying coherence
One limitation of this study is that I only investigated coherence on a binary (completely coherent or randomly selected orientations). In the future, I would like to varying coherence on a spectrum as Dewind et al. did. This could improve performance of the network and make it more comparable to the human experiment.

### Test set vs Train set
In this experiment, I made the training images all randomly oriented stimuli and the test set all coherently oriented. While this can offer important insights, I believe in order to interpret these findings, we would need to compare them to performance by the network when the trainiing and testing stimuli are both coherent or variable. If the network trained on variable stimuli performs similarly when tested on variable and coherent test stimuli, then we might be able to conclude that the illusion does not work on the network. 

### Pretrained networks
For simplicity's sake, I used a pretrained VGG net in this project. Because the network was initially trained on ImageNet, it was difficult to train the network to do a completely different classification with very different stimuli. In light of this, I would like to try training a network on numerical stimuli in the future. For example, it could be trained to count the number of items in naturalistic images like apples in a basket or leafs on a plant. This might make the training/testing of the illusion easier and would also mimic human numerical cognition. 

## 5. Final note

Though this project was not exactly the idealic human-like neural network I had in mind when I set out on this adventure, I have learned quite a bit throughout the process. First, I learned to use PsychoPy for the first time and will continue using it for my research. Second, I learned how to make my own datasets in Python which will allow me to continue experimenting with training neural networks on my own images in the future. Third, debugging was incredibly hard and time-consuming, but I did learn a LOT while doing so (and from the people helping me debug). Without the help of my peers and professor, I probably wouldn't have been able to finish this project. Thank you to: Dr. Emre Neftci, Tim Lui, Helio Tejeda Lemus, and Graham Smith for all the help!

## 6. References
 
Gomez-Villa, A., Martín, A., Vazquez-Corral, J., & Bertalmío, M. (2018). Convolutional Neural Networks Deceived by Visual Illusions. ArXiv:1811.10565 [Cs]. http://arxiv.org/abs/1811.10565

Simonyan, K., & Zisserman, A. (2015). Very Deep Convolutional Networks for Large-Scale Image Recognition. ArXiv:1409.1556 [Cs]. http://arxiv.org/abs/1409.1556

DeWind, N. K., Bonner, M. F., & Brannon, E. M. (2020). Similarly oriented objects appear more numerous. Journal of Vision, 20(4), 4–4. https://doi.org/10.1167/jov.20.4.4
