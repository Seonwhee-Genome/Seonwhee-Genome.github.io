---
title: "Vehicle Automatic Speech Recognition"
tags: [Natural Language Processing, ASR]
categories:
  - Portfolio
date: 2022-07-15
---

### 1. Dataset  
We received three datasets: training, noise, and test sets. The training set was noiseless Korean utterance WAV files and scripts. The noise set was various car engine sounds. The test set was utterances with noisy backgrounds and without scripts. Figs. 1–3 show mel spectrograms of sound files from each set.
![Figure 1](/assets/img_asr/train_set.png)  
*Figure 1. The mel spectrogram of a WAV file from the training set.*
![Figure 2](/assets/img_asr/noise_set.png)  
*Figure 2. The mel spectrogram of a WAV file from the noise set.*
![Figure 3](/assets/img_asr/test_set.png)  
*Figure 3. The mel spectrogram of a WAV file from the test set.*  
### 2. Metric

The metric used to evaluate our system was the character error rate (CER), but it did not include errors in punctuation and spacing. All recognized output from the test set had to be written in CSV format. Our CER was determined by uploading our CSV file to the testbed. All entries were ranked on a leaderboard.

### 3. Fine-tuning the pre-trained wav2vec 2.0-base

The wav2vec 2.0-base(Baevski et al., 2020) model was trained with the LibriSpeech(Panayotov et al., 2015) dataset before the competition. This dataset was a corpus of English speech, so I was not sure whether it would recognize Korean speech. Surprisingly, however, it successfully classified Korean phonemes after being fine-tuned with the training set and had a CER of 0.038688. It even outperformed the DeepSpeech2(Amodei et al., 2016) model that was fine-tuned with the same dataset, which had a CER of 0.474385. These results indicate that high-quality representations learned from English utterances also contained the linguistical features of Korean to some extent.
	
|  | DeepSpeech2 | wav2vec-pf |
| --- | --- | --- |
| CER | 0.474385 | 0.0386878663 |







### 4. How background noise affected speech recognition performance

The test set contained noise, as mentioned above. However, it was impossible to learn representations that were robust to noise by fine-tuning the noiseless training dataset. To address this issue, we created two datasets with noise overlayed on clean utterances with 0.48:0.52 and 1:0 noise-to-clean utterance ratios. We determined optimization progress in terms of how validation losses and validation CERs differed by noise ratios.

As the table below shows, the model fine-tuned with a 1:0 ratio (wav2vec-pf@noise_100%) completely failed. 
  
|  | wav2vec-pf | wav2vec-pf@noise_48% | wav2vec-pf@noise_100% |
| --- | --- | --- | --- |
| validation CTC loss | 0.0293304 | 0.0351575 | 4.4560456 |
| validation CER | 0.11237090 | 0.1133904 | 0.9982381 |
| training epoch | 30 | 30 | 15 |
  

The figure below shows visualizations of the attention map of wav2vec transformer layers. The strength of the check-like patterns in the visualizations indicates the degree to which wav2vec-pf attended to the information in that area of the map. Thus, the left image shows particularly strong attention, while the middle image shows some attention and the right image shows no attention.
![Figure 4](/assets/img_asr/noise_primary.png)

Transformer layer optimization was conducted before the final linear classification test. While wav2vec-pf successfully predicted the class of each audio frame, wav2vec-pf@noise_100% failed to predict any audio frame. These results indicate that fine-tuning with the 1:0 noise-overlayed dataset did not help to learn noise-robust representations.
![Figure 5](/assets/img_asr/linear_layer1.png)
![Figure 6](/assets/img_asr/linear_layer2.png)

Thus, we next used a two-step fine-tuning approach to increase model robustness. First, we fine-tuned the wav2vec 2.0-base with the clean training set and then fine-tuned wav2vec-pf again with the 0.48:0.52 noise-overlayed dataset.

As the table below shows, the two-step fine-tuning process using the 0.48:0.52 noise dataset was promising. Fine-tuning using the 0.48:0.52 noise dataset (wav2vec-pf_sf@noise_48%) resulted in better performance than wav2vec-pf_sf@noise_100%, but it was still difficult to optimize it. 

|  | wav2vec-pf | wav2vec-pf_sf@noise_48% | wav2vec-pf_sf@noise_100% |
| --- | --- | --- | --- |
| validation loss | 0.0293304 | 0.013009 | 3.44943 |
| validation CER | 0.11237090 | 0.108900 | 0.81135 |
| training epoch | 30 | 15 | 6 |

The attention maps below show how the proportion of noise affected attendance to utterance information.
![Figure 7](/assets/img_asr/noise_secondary.png)
The model trained using the 1:0 noise dataset (wav2vec-pf_sf@noise_100%) likely failed because of the difficulty of adapting to data distribution shifts that occurred as a result of the differences between the pre-train and fine-tuning datasets, not from the language difference between the two sets.



### 5. How language affected representation quality
We also conducted an experiment to determine whether language affected representation quality. We compared the fine-tuning results of wav2vec 2.0-base to those of K-Wav2vec 2.0-base(Kim & Kang, 2021), which was wav2vec 2.0-base that was additionally pre-trained with the Korean ASR Ksponspeech dataset. We hypothesized that K-Wav2vec 2.0 would outperform wav2vec-pf and be more robust to noise because the Ksponspeech dataset consisted of Korean utterances and the K-Wav2vec 2.0 authors eliminated silence in the Ksponspeech dataset.  

|  | wav2vec-pf | kwav2vec-pf |
| --- | --- | --- |
| validation loss | 0.03389504 | 4.47127723 |
| validation CER | 0.11209705 | 0.9983875 |
| training epoch | 30 | 7 |


However, as shown in the table below, fine-tuning K-Wav2vec 2.0 with our training set was not successful, which indicates that the effect of language on model performance was minimal.  

### 6. Conclusion  


|  | wav2vec-pf | wav2vec-pf @noise_100% | wav2vec-pf_sf@noise_48% |
| --- | --- | --- | --- |
| CER | 0.038688 | 1 | 0.012369 |

The performance evaluation results reflected the effectiveness of secondary fine-tuning with a moderate ratio of noise data. The primary fine-tuning with clean Korean utterances appeared to cause the models to better learn representations of Korean linguistical features while secondary fine-tuning with 0.48:0.52 noise-overlayed utterances (wav2vec-pf_sf@noise_48%) appeared to increase their robustness to noise.  


### 7. References

Amodei, D., Ananthanarayanan, S., Anubhai, R., Bai, J., Battenberg, E., Case, C., Casper, J., Catanzaro, B., Cheng, Q., Chen, G., Chen, J., Chen, J., Chen, Z., Chrzanowski, M., Coates, A., Diamos, G., Ding, K., Du, N., Elsen, E., … Zhu, Z. (2016). Deep Speech 2: End-to-End Speech Recognition in English and Mandarin. *Proceedings of the 33rd International Conference on International Conference on Machine Learning - Volume 48*, 173–182.  

Baevski, A., Zhou, Y., Mohamed, A., & Auli, M. (2020). wav2vec 2.0: A Framework for Self-Supervised Learning of Speech Representations. In H. Larochelle, M. Ranzato, R. Hadsell, M. F. Balcan, & H. Lin (Eds.), *Advances in Neural Information Processing Systems* (Vol. 33, pp. 12449–12460). Curran Associates, Inc. [https://proceedings.neurips.cc/paper/2020/file/92d1e1eb1cd6f9fba3227870bb6d7f07-Paper.pdf](https://proceedings.neurips.cc/paper/2020/file/92d1e1eb1cd6f9fba3227870bb6d7f07-Paper.pdf)  

Panayotov, V., Chen, G., Povey, D., & Khudanpur, S. (2015). Librispeech: An ASR corpus based on public domain audio books. *2015 IEEE International Conference on Acoustics, Speech and Signal Processing (ICASSP)*, 5206–5210. [https://doi.org/10.1109/ICASSP.2015.7178964](https://doi.org/10.1109/ICASSP.2015.7178964)

Kim, J., & Kang, P. (2021). *K-Wav2vec 2.0: Automatic Speech Recognition based on Joint Decoding of Graphemes and Syllables*. arXiv. [https://doi.org/10.48550/ARXIV.2110.05172](https://doi.org/10.48550/ARXIV.2110.05172)
