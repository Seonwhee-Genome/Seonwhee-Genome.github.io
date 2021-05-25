---
title: "Natural Language Processing after AI OCR"
tags: [Natural Language Processing, AI OCR]
categories:
  - Portfolio
date: 2021-04-10
---
  
### 1. Introduction    
This process is the final step in recognizing processed food. To understand the whole food recognition pipeline and its use case, I recommend you to read the [previous post](https://seonwhee-genome.github.io/portfolio/Food_Recognition/) first.<br>
![Overall](/assets/img_foods/OverallAlgorithm_v7.png)  
<br>
In the previous post, I discussed why AI OCR, a combination of deep learning-based text detection and recognition for images, was implemented in our food recognition pipeline. In this article, I would like to elaborate more on AI OCR post-processing, which is depicted in the bottom-right of the diagram above.<br><br>

### 2. Scanning DBs and matching food names recognized by an AI model  
![Embedding](/assets/img_foods/Embeddings.png)<br>  
The goal of the system is to match a query to the most similar food name in the DB.<br> 
To calculate the similarity between two words, namely a query and the reference in the DB, I proposed using word-embedding because I hypothesized that, if I embedded food names in a vector space, similar words embedded in vectors would have high cosine similarities.<br>
![blossom](/assets/img_foods/recog_example3.png)
<br>
In the example picture above, I only need “아몬드블라썸” to find the closest food name in the DB. However, the text recognized by our model often included additional writing, such as “100 ml,” “500 g,” and “ALMONDBLOSSOM.” Moreover, some products have names that are a combination of the Korean Hangeul alphabet, Chinese characters, English letters, and numbers. It seemed to be inefficient to segregate and filter out additional letters by using rule-based algorithms. Thus, we examined words’ contexts to determine which were essential.<br>
![mont](/assets/img_foods/recog_example2.jpg)<br>
For example, the embedding model could learn “몽쉘,” “몽셀,” “몽쉘_딸기_생크림케이크,” and “몽쉘_생크림케이크” and embed them in vectors. Their vectors would be more similar to each other than to other word vectors.<br> 
My colleague suggested using the minimum edit distance.<br>
We assessed the feasibility of the two strategies.<br>
![two strategies](/assets/img_foods/OCR_NLP.png)<br>
Using word-embedding to identify context seemed like it would complement calculating edit distances alone. However, in practice, it was very complicated because the corpus in the DB consisted only of food names and so was insufficient for us to test our hypothesis. We needed to design the corpus to be able to learn the context of food names successfully.<br>
We found that the minimum edit distance algorithm was robust to misrecognized letters. When we decomposed Korean writing into consonants and vowels, we were able to calculate the edit distance between a query and its corresponding reference more precisely.<br><br>
![edit distance](/assets/img_foods/Edit_distance.png)  
As shown in the figure above, “되치머” seems to be more similar to “돼지바” than “쵸코렛.” However, both “되치머” and “쵸코렛” had an edit distance of 3, indicating that the word-level edit distance cannot represent vowel- and consonant-level differences between two words written in Hangeul. Therefore, they have to be decomposed.<br>
I made an iOS app using Swift to test the AI OCR model and the minimum edit distance algorithm. A commercial Android app was developed by another developer. The images below show the client-side UIs of the apps.<br>
 <img src="/assets/img_foods/OCR_example1.gif" alt="food" width="250" />  


