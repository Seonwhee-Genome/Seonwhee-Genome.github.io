# Recognizing Music in Video Clips  
### 1. Overview  
![Overview](/assets/img_music/MusicRecognition_01.png) 
Many people would like to know what music they are hearing while watching a video. IPTV and OTT media service providers could provide such a music identification service to increase user satisfaction.  
![Overall2](/assets/img_music/MusicRecognition_02.png)  
To identify the music in a video clip, we sent the audio extracted from the video to Houndify, the AI platform developed by SoundHound, Inc. However, given that SoundHound charges by the minute to identify audio tracks, it would not be financially efficient to simply feed in entire videos. Thus, I trained an algorithm to differentiate between music and other types of sounds. Then only the music files would be sent for identification.   
### 2. FFT, MFCC, and VGGish  
The waveforms of sound vary significantly, so it is difficult to generalize the waveforms of music and other types of sounds.    
<img src="/assets/img_music/Waves.png" alt="Waves" width="450" />   
The frequency space of all sinusoidal waves can be identified by fast Fourier transformations.  
<img src="/assets/img_music/FFTs.png" alt="FFTs" width="450" />   
The sample in the figure above shows that the frequency space for non_music/101729_1.wav was significantly different from the other three sample files while that for non_music/130961_46.wav was not.  
Another approach to differentiating music from other sounds could be to implement different type of spectrogram, MFCC. The figure below shows the log Mel spectrums of each sample.  
<img src="/assets/img_music/mel_01.png" alt="mel1" width="250" /><img src="/assets/img_music/mel_02.png" alt="mel2" width="250" />   
<img src="/assets/img_music/mel_03.png" alt="mel3" width="250" /><img src="/assets/img_music/mel_04.png" alt="mel4" width="250" />  
Convolutional neural networks could be run on these Mel-spectrum features, which can be represented as two-dimensional matrices, to extract more complex features of the sounds’ characteristics.  
Below is a visualization of the inference results of a pre-trained VGGish model(Hershey *et al.*, 2017) run on the samples.  
<img src="/assets/img_music/VGGish.png" alt="VGGish" width="500" />  
The VGGish features above are relatively simple and produce clearer patterns that I thought could be used to differentiate between music and other types of sounds.  
### 3. Experiments  
I found that VGGish(Hershey *et al.*, 2017) was effective at distinguishing between music and other types of sounds. Instead of giving more detail about the classifier types and their model architectures, I will present some fundamental experimental results.   
I used the mixed FMA and UrbanSound dataset to train custom classifiers and test their performance.   
Without VGGish features, the classifiers could not effectively distinguish between music and other types of sounds.  
<img src="/assets/img_music/ControlExp.PNG" alt="control" width="200" />  
The client requested that the model maximize the true positive classification rate, which is the rate at which music is properly identified as music.   
<img src="/assets/img_music/ConfusionMat.PNG" alt="confusionmat" width="250" />   
The results showed that the VGGish feature extraction was powerful and applicable to this task.   
   
---------------------  
Hershey, S., Chaudhuri, S., Ellis, D. P. W., Gemmeke, J. F., Jansen, A., Moore, R. C., Plakal, M., Platt, D., Saurous, R. A., Seybold, B., Slaney, M., Weiss, R. J., & Wilson, K. (2017). CNN architectures for large-scale audio classification. *2017 IEEE International Conference on Acoustics, Speech and Signal Processing (ICASSP)*, 131–135. https://doi.org/10.1109/ICASSP.2017.7952132