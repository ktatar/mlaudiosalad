Audio Salad

by XXX

Install following MAX packages from the package manager:

- ml.star 
- zsa.descriptors
- Sadam Library
- ejies 
- jasch objects 

Install the following packages from the links below:

- MuBu for MAX: This library is also available on Max package manager; however, it is an older version 1.9.12 that has a bug. Please avoid the MuBu on Max package manager, and download MuBu >= 1.9.13 from this link: https://forum.ircam.fr/projects/detail/mubu/. Move the MuBu for Max folder to Documents\Max 8\Packages

- Download this repo. 

Go to the Max 8 menu -> Options -> File preferences. Using the plus button and then pressing the choose button; add the abstractions folder of this repo, and make sure it appears in your MAX file path list.   

Restart your Max after you install all libraries and adding the path of the abstractions folder in this repo to the file preferences in Max.

# Training on your own dataset

- Create a folder for your dataset. Name doesn't matter. We will refer to this folder as dataset_folder from now on.

## Preparing your audio files for training

- In your dataset_folder create three folder named,
    - audio   
    - samples
    - forms


Please notice the no-caps. All these folders should have audio files in the format of .wav, 16-bit, 44.1kHz. The difference is,

- audio: This folder should have the recordings of yours, that you would like to use both as an audio sample library, and also as a musical form to train the sequence modelling algorithms. In brief, this folder is for your own compositions where there is no issue of copyright infringement.

- samples: This folder is for individual samples, not for compositions. The .wav files in this folder are for the playback. The files in this folder do not go through a segmentation, they are supposed to be individual sound objects. They are included in the audio memory of the agent. However, they are not used for musical structure learning since they are just samples. 

- forms: This folder is for the compositions that you want to use for the musical structure learning, but you do not want the agent to play any audio segment from these files.    

The dataset expects audio files in these folders. You should convert all your audio files to wav CD format in advance (.wav, 16-bit - 44.1 kHz) because of [MuBu](https://forum.ircam.fr/projects/detail/mubu/). If your files are in another type, you can easily convert them using Audacity:
https://www.audacityteam.org/.

Audacity allows you to create a macro that goes through all your files to convert them to wave files. Under the Audacity menu, go to Tools->Macros. Create a new macro with any name, and insert "Export as wav" command to your macro. The order of commands matter. You can also normalize your files if you want, by inserting a "Normalize" command before "Export as wav".

Before starting the learning, decide a minimum segment duration. Make sure that the audio files in the folders audio and forms are longer than the minimum segment duration to avoid errors.

## Learning 

- Open the Traininig\LEARNING-MAIN.maxpat

We will follow the steps in the given order in this Max patch. Please find extra details and explanations of each step below. If your Max freezes at any step, please be patient and wait. Max scheduler is the most fun when it comes to procedural algorithms or loops. Even though your computer thinks that Max is not responding, it is highly likely that Max is still progressing with the training.

## 1- Import your dataset

Drop your dataset_folder to the area in the step 1. You should see the dataset_folder path under step 1 updated with your folder name. 

## 2- Segmentation and Feature Extraction

This second part goes through each .wav file in your audio folder and creates data files with the segments, and the audio features of the segments. The patch saves the data in the audio folder, as a text file in a folder with the same name of the .wav file. In the text file, each line is the features of a segment in the format, 

Segment-number, File-name Segment-start Segment-duration Valence Arousal Loudness-mean Loudness-std MFCC-mean(13 entries) MFCC-std(13 entries) Perceptual-Spectral-Decrease-mean Perceptual-Spectral-Decrease-std

To run this section, 
- Set your minimum and maximum segment duration in the UI. The segmentation is automatic; however, we can still set the minimum and maximum durations. 
- Press the start button. This will take some time, depending on how many files you have. 

**Debugging:** The patch goes through all .wav files one by one. Sometimes, you may end up with a problematic audio file and the algorithm may get stuck. In my experience, sometimes MuBu cannot load a file with foreign letters in the name. Also, the format of the audio files may cause an issue. In those cases where there is an issue with a file, the patch would get stuck. Check the filename under the overall progress bar to have an idea of which file causes the issue. You can fix or remove the problematic file, and continue the segmentation from where it was by pressing the continue button. Changing the index number also starts the training from the file with the index number in the coll list. You can change the index number as you like, and press continue to start the segmentation from any file you like. 

## 3- Feature Extraction - Samples

Similar to the step 2, and this section goes through the audio sample files in your samples folder to tag them with audio features. Different than step 2, this step does not apply segmentation to the audio files in the samples folder. 

## 4- Concatenate the data

This step first creates a folder called "data" in your dataset_folder. Then, this step concatenates all data files in your audio and samples folder into one text file with the name data-concatenated.txt. This text file becomes MASOM's sound memory. 

- Press the start button. 

To confirm if this step was successful, check if data-concatenated.txt exists in the dataset_folder\data.

## 5- Self-Organizing Map Training

The agent uses a Self-Organizing Map to cluster similar sounds together. It takes around 30 minutes to train an SOM on 10000 audio segments. This step is the most computationally expensive part of the training; hence, this step may take some time.

- (A) We first initiate the parameters of SOM. 
- (B) You don't neccessarily need to do anything in this section since all parameters in this section are set to a default number with the initiate button in step (A). You can change the SOM and training parameters if you would like: 
  - Epochs: You can change the number of epoch of the training. An epoch goes through the dataset for training once. The default is 1000 epochs. 
  - Size of the SOM: SOM map is a square, 2D map in this implementation. The initiate button automatically creates a map that aims to, total number of SOM nodes = total number of segments / 6; hence, approximating 6 sounds per cluster. In my experience, this was a good ratio that gave the least amount of SOM nodes with no sounds after the training. Still, the interface allows you to set the SOM size to any number you like. 
  - Neighborhood divider: SOM training expects a neighborhood parameter. When a node is updated during the SOM training, its neighbooring nodes are also updated with a fraction of the original update amount. The update amount decreases as you move further from the initial node. The neighborhood divider parameter sets the initial neighboorhood amount. During training, the neighborhood is linearly decreasing to 0 as the training progresses.
- (C) Start the training. This will take some time.  
- (D) Save everything by clicking the save button. This step is crucial, as you would loose all training if you don't save it. If all goes accordingly, you should have two more text files in your dataset_folder/data. These files are,
  - SOM-nodes.txt: Each line in this text file is the location of an SOM node in the multidimensional feature space. The first entry, index is the SOM node ID. 
  - stats.txt: This file also contains the feature weights and statistics that are used to train the SOM. The statistics of the dataset allow us to normalize the feature dimensions for SOM training, and mean and standard deviation calculated over the training dataset for each given feature dimension. The feature weights are fixed, and the idea behind the weights is that all MFCC features should affect the training as if they are one timbre feature. 

## 6- Clustering 

After the Self-Organizing Map training, we can assign each audio segment and sample to the SOM node that is the closest to the feature vector of the segment. This is clustering, where each SOM node is an audio cluster with multiple audio segments from the training dataset. 

- Press Start to generate the clusters. 

This section creates clusters.txt in the dataset_folder/data. In the clusters.txt, the first number is the SOM node ID, and the rest of the numbers are the indexes of the audio segments in the data-concatenated.txt file. 

## 7- Analyze recordings in the forms folder  

This section applies segmentation and audio feature extraction to the recordings in the folder "forms". The learning pipeline require this process to create musical structure from these recordings. This algorithm of this step is almost the same as step 2.   

## 8 - Musical Structure

After you complete all previous sections, this step is rather easy. Just click the start button, and press the save button when it is done. This creates the file VMM-training-SOM-seq.txt in the agent_folder/data location. This file is used to train the statistical sequence modelling algorithms, specifically factorOracle and Variable Markov Models. 

This section converts the original songs in the training dataset to a sequence of SOM nodes. This section uses the original order of audio segments, and replace the segments with their associated SOM node numbers. This results in a representation of musical form where each number is a cluster (type) of sounds.   

========================

## Running Factor Oracle version

Let's make some sounds! 

The training until this point allows you to use generative version of the agent with factorOracle. There two types of generative approaches

- **gen-FO**
  This version is monophonic, and plays one sound after another. The sound selection is done automatically. To run,
  
    1- Open the gen-FO-run-example.maxpat
    
    2- Drop your agent_folder to the "drop a folder here" area.
<<<<<<< HEAD
    3- Choose a song from the menu to train the factorOracle. This training is so fast that it doesn't require save & load. The list includes recordings in both the "audio" and "forms" folders.
=======
    
    3- Choose a song from the menu to train the factorOracle. This training is so fast that it doesn't require save & load. 
    
    4- Press the Start button. The toggle under the start button controls a gate that stops sound selection loop. The bang next to the toggle requests Factor Oracle to suggest a new sound. This bang is clickable. The MUTE button mutes all sounds generated by the agent. Congruence is a Factor Oracle parameter that sets the probabilty of forward jumps vs. backwards jumps. 

- **gen-FO-fixed-rhythm**
  The steps to run this version is the same as gen-FO version. The only difference of gen-FO-fixed-rhythm is that the sound selection happens on a regular time interval. There is an additional tempo parameter in the UI to control the time interval of sound selection. This version can play polyphony, if the selected sound samples are longer than the given time interval of sound selection. 
  
=====================

Keep in touch for the Variable Markov Model Max-Order versions for interactive music systems... The guide is upcoming...
  


