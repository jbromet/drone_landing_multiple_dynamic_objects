# Dynamic Object Detection for Drone Delivery 

**Version 1.0.0**

This project aims at improving the detection of dynamic objects from a UAV in a delivery area, working with different heights. 

--- 

## Table of Contents
* [Building a simulated drone scenario with Unity ](#building-a-simulated-drone-scenario-with-Unity)
    * [First steps with Unity](#first-steps-with-Unity)
    * [Convert Unity annotations to VOC format txt](#convert-Unity-annotations-to-VOC-format-txt)
* [Improve YOLO object detector using public data](#improve-YOLO-object-detector-using-public-data)
    * [Dataset Preparation](#dataset-preparation)
    * [Installation of YOLO and Darknet](#installation-of-yolo-and-darknet)
    * [Training and testing](#traning-and-testing)
    * [Evaluation](#evaluation)
* [Tracking](#tracking)
  
 


---

## Building a simulated drone scenario with Unity 

### First steps with Unity

Unity allows to create scenes together with 3D models such as people and vehicles and to animate these objects with a desired speed and direction. 
The _Perception package_ provides a toolkit for generating large-scale datasets for computer vision training and validation.
The images are delivered in the form of frames annotated with ground truth. 

1. Install the latest version of 2019.4.x or 2020.1.x Unity Editor from [here](https://unity3d.com/get-unity/download/archive). 
2. Follow the tutorial to use the [Perception Package](https://github.com/Unity-Technologies/com.unity.perception) of Unity.
3. For our purposes: 
- Add a __Perception Camera__ and __Animator__ to the main camera. The animator component will allow to move the camera and simulate a landing scenario. 
- Enable only the __BoundingBox2DLabeller__ in the Perception Camera. 
- Create a __Simulation Scenario__ with fixed lenght of 500 or 1000 iterations. 
- Create a World with subassets, including pedestrians and bicycles. Label each of your objects by clicking on __Add a component__ and then __Labelling (Script)__. Each of your object should have a different label. Add __Animations__ to your models by writing a Controller script and set up speed.
> My scenario and the assets of my scene can be found here: _drone_landing_multiple_dynamic_objects/Unity/Delivery Scenario_. 
> Download 3D models from the Unity Asset store to create your own scenario. 
- To randomize over several features, add __Randomizers__ (texture or rotation) in the Simulation Scenario and apply them to the objects that you like. 
> Ex : To randomize the texture of the terrain, add a TextureRandomizer in the Simulation Scenario and add a randomizer tag to the terrain. 
- Click on __Play__ and access you files via the link in the console. 

4. Record by installing the __Recorder__ package on Unity. 

### Convert Unity annotations to VOC format txt 

The annotations from Unity are gathered in different .json files called "captures.json". To convert these anotations to a VOC normalized format, run the script drone_landing_multiple_dynamic_objects/Convertors/Unity_to_VOC.py.

---

## Improve YOLO object detector using public data 

### Dataset 

The UAV123 Dataset was used to fine-tune YOLO using images taken at different altitudes. 

1. Download the complete UAV123 & UAV20L datasets [here](https://cemse.kaust.edu.sa/ivul/uav123).
2. For "low images" and pedestrians only: select the 8th and 19th folder. For "high" images and pedestrians only: select the 4th, 7th and 10th folder. Any other folders that match a low altitude or high altitude point of view can be chosen. 
3. Run the script _drone_landing_multiple_dynamic_objects/UAV Dataset/data_preparation.py script to classify, rename and merge folders for the training. 
4. Create a Roboflow account and follow this [tutorial](https://blog.roboflow.com/training-yolov4-on-a-custom-dataset/) to prepare the data for training. 
- Create three datasets on Roboflow (low, high, all heights) and upload the annotations as well. The previous data_preparation.py script takes care of converting the annotations format already.
- Split dataset into 25% Validation, 75% Training, testing beind done using Unity pictures. 
- Make sure every object is annotated. 
> If there are 2 people on the picture, only one is annotated. Annotate the other one manually on Roboflow, as well as delete images where no objects are present. 
- Preprocessing step : 
  * Resize : add 2 black bands to fit the 416x416 format of Darknet. 
  * Blur and brightness addionnal. 
- Augmentation step : any that you like. 
- Create your dataset and copy the zipping link. We will use this link later with Darknet. 

N.B: Step 4 is optional but Roboflow provides easy to implement prepocessing, augmentation and annotation tools. This can be done manually if desired. 

### Installation of YOLO and Darknet 

Relevant and important information about Darknet can be found [here](https://github.com/AlexeyAB/darknet]). Read carefully points [6](https://github.com/AlexeyAB/darknet#how-to-train-to-detect-your-custom-objects), [8](https://github.com/AlexeyAB/darknet#when-should-i-stop-training), [9](https://github.com/AlexeyAB/darknet#how-to-improve-object-detection) and [10](https://github.com/AlexeyAB/darknet#how-to-mark-bounded-boxes-of-objects-and-create-annotation-files).

When using Google Colab, you don't need to install anything. All the packages are already included. To run Darknet locally, follow the link above. 

### Testing and training 

See my [Google Colab](https://colab.research.google.com/drive/16brlfnQlRp286mXA2jOtmoqpGB48QOzO?usp=sharing) for training and testing. 
My google Colab follows the tutorial and the [Colab](https://colab.research.google.com/drive/1mzL6WyY9BRx4xX476eQdhKDnd_eixBlG#scrollTo=GNVU7eu9CQj3) of Roboflow, check it out for more information. 

1. __Preparation__ : Mount your drive and enable an environment with GPU. 
2. __Installing Darknet for YOLOv4 on Collab__ : Ignore the errors when running "!make". Download the pre-trained weights yolo.conv.137 as well as the yolov4.weights which are the weights obtained when training on COCO dataset.
3. __Define helper functions__ 
4. __Create 3 folders__ : "low", "high" and "all" in the same folder where you have cloned Darknet. 
5. Do these steps for __each of the training set__ : 
  *5a. Set up Custom Dataset for YOLOv4 : here copy the zip link from Roboflow. 
  *5b. Change name of files according to wich dataset. Example: %cp train/*.jpg data/obj/_high_/
  *5c. Change number of classes in obj.data if needed
  *5d. Change backup folder in obj.data accordingly
6. __Write custom cfg file__: Modify the darknet/cfg/yolov4-obj.cfg following [this](https://github.com/AlexeyAB/darknet#how-to-train-to-detect-your-custom-objects) or this [video](https://www.youtube.com/watch?v=mmj3nxGT2YQ&t=1121s&ab_channel=TheAIGuy) from minute 20:00. 
7. __Train__ with a learning rate of 0.001. Set up a smaller one once you reach overfitting. Train three times. Weights are saved in the backup folder. 
8. __Infer Custom Objects with Saved YOLOv4 Weights__: 
  *8a. Infer on pictures: for each testing set, detect using the 4 models (3 saved weights + yolov4.weights for COCO module). Then merge each individual .json result into one global .json results file. 
  *8b. Infer on videos: same process. Download the videos afterwards locally. 
9. Evaluate Performance with mAP: 

  





