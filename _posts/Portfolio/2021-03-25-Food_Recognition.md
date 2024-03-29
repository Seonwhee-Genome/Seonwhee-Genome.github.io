---
title: "Food Recognition (Object Detection and AI OCR)"
tags: [Object Detection, AI OCR]
categories:
  - Portfolio
date: 2021-03-25
---
 
### 1. Project overview  
I designed a food recognition system that is able to identify more than 3,000 types of food, including both generic types of food and food produced by particular companies. The client this project was being done for required that the system be able to distinguish between foods produced by specific companies and to get nutritional information from the client’s database. For example, the system would have to be able to distinguish between pepperoni pizzas made by me and by Domino’s.  
### 2. Obstacles  
<img src="/assets/img_foods/opensource_pizza.jpg" alt="peperroni" width="450" />    
There are an infinite number of possible pepperoni pizzas, so it was almost impossible to recognize which were made by Domino’s based solely on their shape. Therefore, object detection models like YOLO (Redmon *et al.*, 2016) were of limited value in distinguishing between who produced a particular food product.  
To meet the client’s requirements, I re-defined the task. First, the object detection model would determine what the food in question was but not try to determine who made it. The brand and product name were required to identify who produced the food in question. Thus, if the photo of the food contained a package label, then the system could identify the product’s name and who made it. It was for this reason that I included the AI OCR model in the system.    
### 3. Food recognition machine learning pipeline  
![Overall](/assets/img_foods/OverallAlgorithm_v7.png)  
First, my colleague and I trained a model which classified images according to whether they were natural or processed foods. Next, the image was subject to either object detection or AI OCR. The former was used to determine what the food was and where it was located. we re-trained well-known pre-trained object detection models, like SSD (Liu _et al._, 2016) and YOLO(Redmon *et al.*,2016). The latter was used to detect and recognize the text on processed food packages to identify the brand and product name.  
The reason that the classifier was placed at the front and object detection and OCR occurred separately was to overcome the system’s limitations which were that natural foods and processed food packages could not be in the same picture and that only one packaged processed food could be identified at a time.  
In order to generate a sufficiently large dataset, we scraped a large number of natural food thumbnail images that also contained watermarks, banners, and other text. This extraneous text was an obstacle to overcoming the first challenge.    
![Alt1](/assets/img_foods/OverallAlgorithm_v9.png)  
Bottles, cans, and other types of packages for beverages, like sodas and beer, were detectable by object detection techniques, not OCR, when they were with other natural foods. Thus, the model overcame its limitations to some extent. This beverage recognition modeling was based on the uniqueness of the package design and the likelihood of particular combinations of foods and beverages. For example, sodas would likely be paired with hamburger combos. Therefore, OCR was not needed to identify packages.  
![Alt2](/assets/img_foods/OverallAlgorithm_v8.png)  
The alternative model was based on an object detection model. This model could differentiate between natural and processed foods and localize and classify the natural foods. Processed foods were cropped and subject to OCR. The advantage of this model was that, in theory, it did not face the limitations mentioned above that required users to take photos of individual food items. However, the effectiveness of the AI OCR process was limited by the resolution of the photo. Text had to be at least 38 pixels tall (Wang *et al.*, 2020).  
### 4. Model inference server  
![WebServer](/assets/img_foods/FoodRecognition_Serve.png)  
This diagram shows how the client makes a REST API call and the server responds with the food location and identification results.  
Although both TensorFlow and PyTorch provide their own server systems, TensorFlow Serving and TorchServe, respectively, they had intractable compatibility issues. I built a web API server using Django and defined an OpenAPI to make inferences. After clients submitted POST requests with uploaded food images as multipart/form-data, the web server performed inferences on uploaded images and returned the results as JSONs.   
I made an iOS app using Swift for testing purposes. A commercial Android app was developed by another developer. The gif below show the client-side UI of how inferences were made.   
 <img src="/assets/img_foods/OCR_example2.gif" alt="food" width="250" />    
   
-------------------
Liu, W., Anguelov, D., Erhan, D., Szegedy, C., Reed, S., Fu, C.-Y., & Berg, A. C. (2016). SSD: Single Shot MultiBox Detector. In B. Leibe, J. Matas, N. Sebe, & M. Welling (Eds.), *The 14th European Conference on Computer Vision (ECCV)* (pp. 21–37). Springer. https://doi.org/10.1007/978-3-319-46448-0_2    
  
Redmon, J., Divvala, S., Girshick, R., & Farhadi, A. (2016). You Only Look Once: Unified, Real-Time Object Detection. In Z. Akata, R. Arandjelovic, A. Argyros, T. Avraham, S. Branson, J. Choi, & A. Davison (Eds.), *2016 IEEE Conference on Computer Vision and Pattern Recognition (CVPR)* (pp. 779–788). IEEE. https://doi.org/10.1109/CVPR.2016.91    
  
Wang, W., Xie, E., Liu, X., Wang, W., Liang, D., Shen, C., & Bai, X. (2020). Scene Text Image Super-resolution in the Wild. In A. Vedaldi, H. Bischof, T. Brox, & J.-M. Frahm (Eds.), *Computer Vision – ECCV 2020* (Vol. 12355). Springer, Cham. https://doi.org/10.1007/978-3-030-58607-2_38  
