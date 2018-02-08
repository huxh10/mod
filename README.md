# Object Detection on Edge

## Platform Setup

[Install Intel Movidius SDK on a Linux server](https://movidius.github.io/ncsdk/).

Clone current Movidius-supported models from [Neural Compute App Zoo](https://github.com/movidius/ncappzoo).

Movidius doesn't support Faster-RCNN for object detection. We use caffe proejct [SSD with Mobilenet as backend](https://github.com/movidius/ncappzoo/tree/master/caffe/SSD_MobileNet).

## Cutomized Object Detection Model

We use this [caffe Mobilenet-SSD implementation](https://github.com/chuanqi305/MobileNet-SSD).

1. Caffe installation and data preparation
	* Follow the README and install [caffe](https://github.com/weiliu89/caffe/tree/ssd). There could be some errors while installing caffe, most of which can be solved by modifying ```Makefile.config```.

	* Prepare our own dataset. Images can be crawled using [BaiduImageSpider](https://github.com/kong36088/BaiduImageSpider). Use [LabelImg](https://github.com/tzutalin/labelImg) to label images and generate VOC type data.

	* Edit ```labelmap_voc.prototxt``` and use scripts ```create_list.sh``` and ```create_data.sh``` ([caffe/data/VOC0712/](https://github.com/weiliu89/caffe/tree/ssd/data/VOC0712)) to create lmdb format data.

2. Back to Mobilenet-SSD and train
	* Edit ```labelmap.prototxt``` and run ```./gen_model.sh class_num```.
	
	* Use the trained weights to start (transfer learning). Run ```./train.sh```. (Even after 30,000 iterations, the loss is still around 1.0, around 1500 training images, three classes)

	* Run ```python merge_bn.py``` to generate deploy caffemodel.
	
	* Run ```python demo.py```. We have another python file ```demo_video.py``` to process video.

3. Compile caffemodel to Movidius graph and run
	* copy ```MobileNetSSD_deploy.caffemodel``` and ```MobileNetSSD_deploy.prototxt``` to ```ncappzoo/caffe/SSD_Mobilenet```

	* run ```mvNCCompile -w MobileNetSSD_deploy.caffemodel -s 12 MobileNetSSD_deploy.prototxt``` to get graph file, which is exectuted in Movidius Neural Compute Stick. Current mvNCCompile has a bug while compile the customized prototxt, [to fix it](https://ncsforum.movidius.com/discussion/572/converting-ssd-mobilenet#latest).

	* run ```python3 run.py```. We have another python file ```run_video.py``` to process video.
