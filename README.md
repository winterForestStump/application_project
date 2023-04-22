# Application Project: Dangerous goods label classification for Lübecker Hafen-Gesellschaft mbH (LHG)

## Task description:
The central focus of this project is **the implementation of an ML solution for the identification and classification of dangerous goods labels on trailers**. Beyond that optionally additional functionalities can be added, which will be implemented depending on the time available:
* Integration and comparison with existing booking data from the LHG logistics system
* Display of deviations in a web frontend
* Implementation of a push mechanism (e-mail) for notification of deviations
The language and technology used to implement the requirements can be freely selected. For a possible integration into the IT system landscape of the LHG a Docker container is suitable.

The LHG provides the following data:
·   	Image material of the scan system of all units for one year.
·   	Booking data for the units for one year as an export from the LHG logistics system in Excel format


## Implementation plan: 

### 1. Collecting and preprocessing the data: 
From the LHG we received HDD with more than 6 TB of pictures of trucks on it. The total number of photos is about 2.3 million. The directory structure is as follows:
- Folder with the name of the gate (since there are 4 gates (two at the entrance and two at the exit) 4 folders in total)
  - Folders for each day (365 folders)
    - Zip folders for each vehicle (the number of archived folders corresponds to the number of vehicles passed through the gate per day)
      - Pictures of the vehicles (in average of about 40 photos: photos of tractor unit and trailer from different angles, photos of license plates and distinguishing marks, etc.)

We analyzed the data and came to the following conclusions:

* each zip folder has a unique name, including the timestamp with the YYYYMMDD-HHMMSS format
* inside the zip folder specific photos have unique names
* the specific photos have the same names in all zip folders
* the most suitable photos for use in ML and DL models have the following names and characteristics

<img width="400" alt="image" src="https://user-images.githubusercontent.com/101496738/227512259-32dc0d38-3443-4719-a528-aaa600b4ff4e.png">

* we also discussed with the company the accuracy of the current system of recognition of dangerous signs and received the answer that the accuracy is not known because the system is not used in current activities
* after discussion and analysis, we decided that the most appropriate photos for teaching would be those taken from the back of the vehicle and titled “snapshot-chassis0-Isback-full.jpg”
* the company also provided an excel file with the tableu data. We analyzed the file and came to the conclusion that it is difficult to use it in the process of processing and obtaining the necessary photos. So the only identifier of the vehicle in the excel file, which can be used in the identification and filtering of photos is datestamp. In addition, some of the vehicles in the file do not have a mark on the presence of dangerous goods and, accordingly, the sign, while in the photo taken at the specified time at the specified gate, it is present.

We have prepared a code [Script for data processing](script_for_extraction_ver2.ipynb) for extracting the necessary photos from zip folders. As a result, we were able to extract 574K photos taken from the rear. In order to reduce the dataset, we decided to further filter the photos and exclude non-trucks from the set. As a result the dataset was reduced to 338K photos. 


TODO TASKS:
- [x] Receive from the LHG all the pictures of trucks. 
- [X] Ask the company about object detection system already implemented (if exist)
- [x] Prepare the code to retrive data
- [X] Prepare code for filtering photos




### 2. Labeling the data: 
In order for the model to detect and recognize the sign on the photo, we need to mark up and assign labels to each sign. For these purposes we used [MakeSense.ai](https://www.makesense.ai/). On their website they are declaring that: 
* Open source and free to use under GPLv3 license
* No advanced installation required, browser usage
* They don't store the images, because they don't send them anywhere
* Support multiple label types - rects, lines, points and polygons (we are using polygons)
* Support output file formats like YOLO, VOC XML, VGG JSON, CSV (we are using YOLO)

So far we have analyzed about 41 photos and received 1,456 labeled photos. We have 12 classes of very uneven distributed signs:
<img width="180" alt="image" src="https://user-images.githubusercontent.com/101496738/233804454-c8aeaf04-4dc8-44d3-b63e-d626f3a6fc9a.png">


TODO TASKS:
- [X] Choose the labeling tool and start labeling
- [ ] Proceed with labeling
- [ ] Find a way to balance the labels distribution




### 3. Working with DL model: 
For the training we have chosen [YOLOv5](https://pytorch.org/hub/ultralytics_yolov5/) model. It is a compound-scaled object detection model trained on the COCO dataset, and includes simple functionality for Test Time Augmentation (TTA), model ensembling, hyperparameter evolution, and export to ONNX, CoreML and TFLite.

We used the currently available marked-up photos in the following proportions: 900 for the training session and 100 for the test session. The [Notebook](YOLOv5_project_.ipynb) with the code and training and test results.

So far we have received some preview results. Confusion matrix after testing on unbalanced test data is following:
<img width="390" alt="image" src="https://user-images.githubusercontent.com/101496738/233805486-c3953d77-667b-486d-81c7-8f8e183bf437.png">

Also we have made some recognition and classification with our model. Some examples of the recognized signs:
<img width="304" alt="image" src="https://user-images.githubusercontent.com/101496738/233805893-3520e516-3d79-42c9-9f81-2d9842196662.png">
<img width="301" alt="image" src="https://user-images.githubusercontent.com/101496738/233805919-4efc41df-b34b-4314-af38-8aa31102c699.png">
<img width="302" alt="image" src="https://user-images.githubusercontent.com/101496738/233805968-02e3aafd-dfe7-4b8e-81b7-a96cd1a2c26d.png">


TODO TASKS:
- [X] Select a pre-trained model
- [X] Conduct first experiment with existing labeled data
- [ ] Conduct more experiments with more data
- [ ] Check the evaluating metrics of the model (accuracy, precision, recall, F1-score) to measure effectiveness
- [ ] Conduct test experiments with balanced data
- [ ] Train the model with pictures without signs and no-trucks pictures
- [ ] Fine-tune the model if necessary



### 4. Deploying the model: 

TODO TASKS:
- [ ] Create a Docker container
- [ ] Integrate and compare with existing booking data from the LHG logistics system
- [ ] Display deviations in a web frontend
- [ ] Implement a push mechanism (e-mail) for notification of deviations
