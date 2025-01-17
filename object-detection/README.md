# Object detection with Intel® Distribution of OpenVINO™ toolkit 

This tutorial uses a Single Shot MultiBox Detector (SSD) on a trained mobilenet-ssd* model to walk you through the basic steps of using two key components of the Intel® Distribution of OpenVINO™ toolkit: Model Optimizer and Inference Engine. 

Model Optimizer is a cross-platform command-line tool that takes pre-trained deep learning models and optimizes them for performance/space using conservative topology transformations. It performs static model analysis and adjusts deep learning models for optimal execution on end-point target devices. 

Inference is the process of using a trained neural network to interpret data, such as images. This lab feeds a short video of cars, frame-by-frame, to the Inference Engine which subsequently utilizes an optimized trained neural network to detect cars. 

### Download workshop content and set directory path
Please make sure to complete the [Lab Setup](https://github.com/intel-iot-devkit/smart-video-workshop/blob/master/Lab_setup.md). 
	
#### Set short path for the workshop directory

	export SV=/opt/intel/workshop/smart-video-workshop/
    
## Part 1: Optimize a deep-learning model using the Model Optimizer (MO)

In this section, you will use the Model Optimizer to convert a trained model to two Intermediate Representation (IR) files (one .bin and one .xml). The Inference Engine requires this model conversion so that it can use the IR as input and achieve optimum performance on Intel hardware.

#### 1. Create a directory to store IR files
 	
	cd $SV/object-detection/
	mkdir -p mobilenet-ssd/FP32 

#### 2. Navigate to the Intel® Distribution of OpenVINO™ toolkit install directory

	cd /opt/intel/openvino/deployment_tools/model_optimizer

#### 3. Run the Model Optimizer on the pretrained Caffe* model. This step generates one .xml file and one .bin file and place both files in the tutorial samples directory (located here: /object-detection/)

	python3 mo_caffe.py --input_model /opt/intel/openvino/deployment_tools/open_model_zoo/tools/downloader/public/mobilenet-ssd/mobilenet-ssd.caffemodel -o $SV/object-detection/mobilenet-ssd/FP32 --scale 256 --mean_values [127,127,127]

> **Note:** Although this tutorial uses Single Shot MultiBox Detector (SSD) on a trained mobilenet-ssd* model, the Inference Engine is compatible with other neural network architectures, such as AlexNet*, GoogleNet*, MxNet* etc.

<br>

The Model Optimizer converts a pretrained Caffe* model to make it compatible with the Intel Inference Engine and optimizes it for Intel® architecture. These are the files you would include with your C++ application to apply inference to visual data.
	
> **Note:** if you continue to train or make changes to the Caffe* model, you would then need to re-run the Model Optimizer on the updated model.

#### 4. Navigate to the tutorial sample model directory

	cd $SV/object-detection/mobilenet-ssd/FP32

#### 5. Verify creation of the optimized model files (the IR files)

	ls

You should see the following two files listed in this directory: **mobilenet-ssd.xml** and **mobilenet-ssd.bin**


## Part 2: Use the mobilenet-ssd* model and Inference Engine in an object detection application


#### 1. Open the sample app (main.cpp) in the editor of your choice to view the lines that call the Inference Engine.

	cd $SV/object-detection/
	gedit main.cpp

* Line 130 &#8212; loads the Inference Engine plugin for use within the application
* Line 144 &#8212; initializes the network object
* Line 210 &#8212; loads model to the plugin
* Line 228 &#8212; allocate input blobs
* Line 238 &#8212; allocate output blobs
* Line 289 &#8212; runs inference using the optimized model


#### 2. Close the source file

#### 3. Source your environmental variables

	source /opt/intel/openvino/bin/setupvars.sh

#### 4. Build the sample application with make file

 	cd $SV/object-detection/
	make
	
> **Note:** *Please ignore the warnings. They are due to new IE APIs*
#### 5. Download the test video file to the object-detection folder. 
Put the below link in your favorite browser. 

	https://pixabay.com/en/videos/download/video-1900_source.mp4?attachment
	
Cars - 1900.mp4 file will get downloaded. Put that file in the $SV/object-detection folder. 

	mv ~/Downloads/Cars\ -\ 1900.mp4 .

#### 6. Run the sample application to use the Inference Engine on the test video
The below command runs the application 
	 
	./tutorial1 -i $SV/object-detection/Cars\ -\ 1900.mp4 -m $SV/object-detection/mobilenet-ssd/FP32/mobilenet-ssd.xml 
 
> **Note:** If you get an error related to "undefined reference to 'google::FlagRegisterer...", try uninstalling libgflags-dev: sudo apt-get remove libgflags-dev

#### 7. Display output
For simplicity of the code and in order to put more focus on the performance number, video rendering with rectangle boxes for detected objects has been separated from main.cpp. 

	 make -f Makefile_ROIviewer 
	./ROIviewer -i $SV/object-detection/Cars\ -\ 1900.mp4 -l $SV/object-detection/pascal_voc_classes.txt 
	
You should see a video play with cars running on the highway and red bounding boxes around them. 

Here are the parameters used in the above command to run the application:

	./tutorial1 -h

		-h              Print a usage message
		-i <path>       Required. Path to input video file
		-model <path>   Required. Path to model file.
		-b #            Batch size.
		-thresh #       Threshold (0-1: .5=50%)
		-d <device>     Infer target device (CPU or GPU or MYRIAD)
		-fr #           Maximum frames to process
	

## Part 3: Run the example on different hardware

**IT'S BEST TO OPEN A NEW TERMINAL WINDOW SO THAT YOU CAN COMPARE THE RESULTS**

 Make sure that you have sourced the environmental variables for each newly opened terminal window.
 
	source /opt/intel/openvino/bin/setupvars.sh
	
	export SV=/opt/intel/workshop/smart-video-workshop/
	
	cd $SV/object-detection
 
#### 1. CPU
```
./tutorial1 -i $SV/object-detection/Cars\ -\ 1900.mp4 -m $SV/object-detection/mobilenet-ssd/FP32/mobilenet-ssd.xml -d CPU
```
You will see the **total time** it took to run the inference.

#### 2. GPU
Since you installed the OpenCL™ drivers to use the GPU, you can run the inference on GPU and compare the difference.

Set target hardware as GPU with **-d GPU**
```
./tutorial1 -i $SV/object-detection/Cars\ -\ 1900.mp4 -m $SV/object-detection/mobilenet-ssd/FP32/mobilenet-ssd.xml -d GPU
```

The **total time** between CPU and GPU will vary depending on your system.
