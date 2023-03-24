# application_project

Application Project: Dangerous goods label classification for Lübecker Hafen-Gesellschaft mbH (LHG)

Task description:
The implementation of an ML solution for the identification and classification of dangerous goods labels on trailers is the central focus of this project. Beyond that optionally additional functionalities can be added, which will be implemented depending on the time available:
·   	Integration and comparison with existing booking data from the LHG logistics system
·   	Display of deviations in a web frontend
·   	Implementation of a push mechanism (e-mail) for notification of deviations
The language and technology used to implement the requirements can be freely selected. For a possible integration into the IT system landscape of the LHG a Docker container is suitable.

The LHG provides the following data:
·   	Image material of the scan system of all units for one year.
·   	Booking data for the units for one year as an export from the LHG logistics system in Excel format


Implementation plan: 

1.	Collecting Data: 
Examples (pictures of 7 trucks) of data already received. We found out that pictures of different trucks have the same names. All pictures of one truck are inside one archived folder. Names of pictures that can be used during the project:
<img width="551" alt="image" src="https://user-images.githubusercontent.com/101496738/227512259-32dc0d38-3443-4719-a528-aaa600b4ff4e.png">


This is part of the photo from the upper left corner of “snapshot-chassis0-unit0-Isright-ocr-dcocr1.jpg” file:  
![image](https://user-images.githubusercontent.com/101496738/227512318-b4976e9c-416b-444a-9ab5-f96194b4925e.png)
Based on this, the company may already have a system for recognizing signs of hazardous materials HOG - Histogram of Oriented Gradients. HOG  is a feature descriptor for images and is widely used in vision and image processing tasks for object detection and recognition. It is accurate and fast. Although it may not be as good as today’s deep learning object detection techniques.
https://towardsdatascience.com/hog-histogram-of-oriented-gradients-67ecd887675f.
https://debuggercafe.com/image-recognition-using-histogram-of-oriented-gradients-hog-descriptor/

TODO:
●	Receive from the LHG all the pictures of trucks. 
●	Ask the company about object detection system already implemented (if exist)


2.	Preprocessing the Data
There is a Python script that should extract the specific pictures you need from your archived folders and move them to a new folder with the same name as the original folder: ‘script.ipynb’

TODO:
●	Clean the unnecessary data. In each archive folder there are about 40 files (images of the truck from different sides and different resolutions). We need only a few of them. So doing this we can save storage space, time for labeling. So we can create a script to extract only photos we need.
●	Segmenting and labeling the hazardous material signs on the pictures of trucks can be done in several ways. Here are some common methods:
○	Manual Annotation (LabelImg, VIA Annotation, Kili)
○	Object Detection (object detection algorithms YOLO or Faster R-CNN)


3.	Creating Classifiers: 

TODO:
●	Firstly we can try to find pretrained model for hazmat signs classification (arXiv.org)
●	Create neural network models in PyTorch. We can choose from several architectures, such as Convolutional Neural Network (CNN) or Residual Network (ResNet).
●	We think about creating two models. First model should be used to determine if there is a sign or not. If not, then the process terminated, otherwise the second model will classify the sign.
●	During the process of creating NN models following aspects should be discussed:
○	Defining the Loss Function and Optimizer:
○	During how many epochs train the model
○	What metrics to use for evaluating (accuracy, precision, recall, F1-score) to measure effectiveness
○	Fine-Tuning the Model - adjusting the hyperparameters (learning rate, number of layers, and activation functions) to improve the model's performance.


4.	Deploying the Model: 

TODO:
●	Deploy the model to a production environment making a docker container.

