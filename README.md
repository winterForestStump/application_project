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
* after discussion and analysis, we decided that the most appropriate photos for training would be those taken from the back of the vehicle and titled “snapshot-chassis0-Isback-full.jpg”
* the company also provided an excel file with the tableu data. We analyzed the file and came to the conclusion that it is difficult to use it in the process of processing and obtaining the necessary photos. So the only identifier of the vehicle in the excel file, which can be used in the identification and filtering of photos is datestamp. In addition, some of the vehicles in the file do not have a mark on the presence of dangerous goods and, accordingly, the sign, while in the photo taken at the specified time at the specified gate, it is present.

We have prepared a code [Script for data processing](script_for_extraction_ver2.ipynb) for extracting and manipulating the necessary photos from zip folders. As a result, we were able to extract 574K photos taken from the rear. In order to reduce the dataset, we decided to further filter the photos and exclude non-trucks from the set. As a result the dataset was reduced to 338K photos. 


TODO TASKS:
- [x] Receive from the LHG all the pictures of trucks. 
- [X] Ask the company about object detection system already implemented (if exist)
- [x] Prepare the code to retrive data
- [X] Prepare code for filtering photos


### 2. Labeling the data: 
In order for the model to detect and recognize the sign on the photo, we were needed to mark up and assign labels to each sign. For these purposes we used [MakeSense.ai](https://www.makesense.ai/). On their website they are declaring that: 
* Open source and free to use under GPLv3 license
* No advanced installation required, browser usage
* They don't store the images, because they don't send them anywhere
* Support multiple label types - rects, lines, points and polygons (we are using polygons)
* Support output file formats like YOLO, VOC XML, VGG JSON, CSV (we are using YOLO)

So far we have analyzed about 70K photos and received 2,470 labeled photos. We used photos from only one gate (ICG02) taken during the throughout the year, so as to get pictures of different day time (day, evening, night, morning) and time of the year.  We have 12 classes of very uneven distributed signs:
Labels distributions for the previous dataset (41K photos analyzed, 1,400 labeled photos):

<img width="326" alt="2023-05-15 19_46_15-233804454-c8aeaf04-4dc8-44d3-b63e-d626f3a6fc9a png (359×529)" src="https://github.com/Stump-rus/application_project/assets/101496738/8a6bda65-57dc-43c3-a300-edc9905a6749">

Labels distributions for the new dataset (70K photos analyzed, 2,470 labeled photos):
<img width="326" alt="2023-05-15 19_44_25-labels jpg ‎- Photos" src="https://github.com/Stump-rus/application_project/assets/101496738/dba98943-75b7-4905-a4bb-ef59c350fdb5">

We still have unbalansed data, but as the results of the model test indicate that this is not a significant problem.

TODO TASKS:
- [X] Choose the labeling tool and start labeling
- [X] Proceed with labeling
- [X] Find a way to balance the labels distribution


### 3. Working with DL model: 
For the training we have chosen [YOLOv5](https://pytorch.org/hub/ultralytics_yolov5/) model. It is a compound-scaled object detection model trained on the COCO dataset, and includes simple functionality for Test Time Augmentation (TTA), model ensembling, hyperparameter evolution, and export to ONNX, CoreML and TFLite.

We decided to use medium size YOLOv5m model:
<img width="528" alt="2023-05-15 19_53_02-ultralytics_yolov5_ YOLOv5" src="https://github.com/Stump-rus/application_project/assets/101496738/c8a937bc-d3e8-4a57-908d-f503b09d2251">

We used the currently available marked-up photos in the following proportions: 2,270 for the training session and 200 for the test session. We also added 265 background images. Background images are images with no objects that are added to a dataset to reduce False Positives (FP). It is recommended about 0-10% background images to help reduce FPs. No labels are required for background images. The [Notebook](YOLOv5m_model.ipynb) with the code,training and test results.

We trained the model on Google Colab free account using the GPU computing power. Training process (100 epochs, 16 batches) took about 3.5 hours. After the training we have received following results:
![results](https://github.com/Stump-rus/application_project/assets/101496738/f6944082-51fa-4bba-888a-0040e04a6bf7)

***Test results***:
We tested the model on the test dataset of 200 pictures (different classes and background pictures). The results are the following:
Once the Bento is built, containerize it as a Docker image for deployment:

| Class | Images| Instances | P | R | mAP50 | mAP50-95|
| ----- | ------| --------- | - | - | ----- | --------|
| all | 200 | 135 | 0.902 | 0.915 | 0.951 | 0.793 |
| 2 Gases | 200 | 8 | 0.807 | 0.75 | 0.781 | 0.723 |
| 3 Flammable liquids | 200 | 23 | 0.876 | 1 | 0.976 | 0.782 |
| 5 Oxidizing substances | 200 | 5 | 0.962 | 1 | 0.995 | 0.906 |
| 6 Toxic substances | 200 | 3 | 1 | 0.778 | 0.995 | 0.764 |
| 8 Corrosive substances | 200 | 19 | 0.896 | 0.905 | 0.964 | 0.771 |
| 9 Miscellaneous dangerous | 200 | 28 | 0.923 | 1 | 0.988 | 0.815 |
| MP Marine pollutant | 200 | 31 | 0.935 | 1 | 0.995 | 0.844 |
| LQ | 200 | 18 | 0.82 | 0.889 | 0.917 | 0.741 |

Confusion matrix after testing:
![confusion_matrix](https://github.com/Stump-rus/application_project/assets/101496738/f5a47ffd-6518-4bd4-8087-d638158577ae)

Also we have made some recognition and classification with our model. Some examples of the recognized signs:
![val_batch0_labels](https://github.com/Stump-rus/application_project/assets/101496738/7cab6a84-8410-49f9-b27a-f4a430a706db)
![val_batch2_labels](https://github.com/Stump-rus/application_project/assets/101496738/1e4f79e5-508b-47b2-9795-5ca750f7a80a)



YOLOv5 LICENSE: https://github.com/ultralytics/yolov5/blob/master/LICENSE


TODO TASKS:
- [X] Select a pre-trained model
- [X] Conduct first experiment with existing labeled data
- [X] Conduct more experiments with more data
- [X] Check the evaluating metrics of the model (accuracy, precision, recall, F1-score) to measure effectiveness
- [X] Conduct test experiments with balanced data
- [X] Train the model with pictures without signs and no-trucks pictures
- [ ] Fine-tune the model if necessary



### 4. Deploying the model: 

**UPDATE:**

For deploying the model we used [BentoML](https://www.bentoml.com/) platform. Containerizing as Docker image allows users to easily test out environment and dependency configurations locally. Once Bentos are built and saved to the bento store, we can containerize saved bentos with the CLI command: bentoml containerize.
We created a docker image **yolo_v5_demo:owwlnyx5fkcp2cdk** (5.92 GB) which can be run locally.

Run the command **docker run -it --rm -p 3000:3000 yolo_v5_demo:owwlnyx5fkcp2cdk serve** and visit **http://127.0.0.1:3000** (for me port 3000 didn't work and I changed it to the 5000, which worked fine)

The sample service provides two different endpoints:
* `/invocation` - takes an image input and returns predictions in a tabular data format
* `/render` - takes an image input and returns the input image with boxes and labels rendered on top
<img width="660" alt="01_BentoML Prediction Service" src="https://github.com/Stump-rus/application_project/assets/101496738/0d01dc79-62ed-4281-958e-66ccc7d450bb">

* For /invocation option "Try it out" button. Choose file for sign detection. Push "Execute" button.On response: code 200 and json file with boundaries coordinates, confidence of classification, class number (from the class_names.txt file) and class names.
* For /render option push "Try it out" button. Choose file for sign detection. Push "Execute" button. On response: code 200 and link to "Download file" of the picture with detected signs

There are some [pictures of the trucks](test_pictures) hich you can use for model and service testing.

TODO TASKS:
- [X] Use BentoML to deploy the model
- [X] Create a Docker container
- [ ] (optional) Integrate and compare with existing booking data from the LHG logistics system
- [ ] (optional) Display deviations in a web frontend
- [ ] (optional) Implement a push mechanism (e-mail) for notification of deviations