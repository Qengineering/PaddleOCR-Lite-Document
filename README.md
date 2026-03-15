# ncnn - PaddleOCR document scanner
![output image]( https://qengineering.eu/github/WillekePassOut.webp )<br>
_Image source: [Dutch  government](https://www.rijksoverheid.nl/actueel/nieuws/2021/08/02/nieuw-model-nederlandse-identiteitskaart-per-2-augustus-2021)._<br><br>
[![License](https://img.shields.io/badge/License-BSD%203--Clause-blue.svg)](https://opensource.org/licenses/BSD-3-Clause)<br/><br/>
Paper: [E-book: _Dive Into OCR_ (PDF)](https://paddleocr.bj.bcebos.com/ebook/Dive_into_OCR.pdf)<br/><br/>
PaddleOCR is a text engine. It can detect text in a scene, determine the orientation, and recognise the characters (Chinese or Latin).<br>
This application uses an [**ncnn**](https://github.com/Tencent/ncnn) version of PaddleOCR.<br><br>
In the early days, the PaddleOCR version had to match its corresponding Paddle-Lite version to run properly on a Raspberry Pi. However, since Paddle-Lite has been largely abandoned on GitHub, while the main PaddleOCR project has continued advancing with more powerful models (such as PP-OCRv4 and v5), that compatibility is no longer practical.<br>
For this reason, we use the ncnn port of the latest PaddleOCR version (**v5**).<br>
It eliminates the need for installing PaddlePaddle or Paddle-Lite, making setup much simpler.<br>
Once everything is installed, it runs entirely on a Raspberry Pi — no cloud services or expensive licenses required.<br><br>
Inference time (RPi 4 @ 1925 MHz - 64 bits Bullseye OS):<br>
Detect text: 366 mSec.<br>
Recognize text: 134 mSec per line.<br><br>
Inference time (**RPi 5** @ 2450 MHz - 64 bits Bookworm OS):<br>
Detect text: 183 mSec.<br>
Recognize text: 67 mSec per line.<br><br>
Special made for a bare Raspberry Pi 4, see [Q-engineering deep learning examples](https://qengineering.eu/deep-learning-examples-on-raspberry-32-64-os.html)

------------

## Dependencies.
To run the application, you have to:
- A Raspberry Pi 4 with a 32 or 64-bit operating system. It can be the Raspberry 64-bit OS, or Ubuntu 18.04 / 20.04. [Install 64-bit OS](https://qengineering.eu/install-raspberry-64-os.html) <br/>
- ncnn. [Install ncnn](https://qengineering.eu/install-ncnn-on-raspberry-pi-4.html) <br/>
- OpenCV 64-bit installed. [Install OpenCV 4.5](https://qengineering.eu/install-opencv-4.5-on-raspberry-64-os.html) <br/>
- Optional Code::Blocks installed. (```$ sudo apt-get install codeblocks```)

------------

## Installing the app.
Before you can run the application, you need to install quite a lot of software. Let's start.
#### OpenCV
The first package you need is OpenCV. On our website, there are [several guides](https://qengineering.eu/install-opencv-4.5-on-raspberry-64-os.html) on how to install OpenCV on your Raspberry Pi.<br>
It shouldn't be a problem if you follow the instructions carefully.
#### ncnn
The next package you need is the ncnn framework. On our website, there is a [guide](https://qengineering.eu/install-ncnn-on-raspberry-pi-4.html) on how to install ncnn.<br>
Again, an easy installation.
#### Code::Blocks
For compilation, you can choose between CMake or Code::Blocks.
If you want to write code, Code::Blocks can be more comfortable to work with due to its IDE.
```
$ sudo apt-get install codeblocks
```
------------

## Installing the app.
With all the tools in place, it is time to build the application.
```
$ mkdir MyDir
$ cd MyDir
$ git clone https://github.com/Qengineering/PaddleOCR-Lite-Document.git
```
Your MyDir folder must now look like this:
```
.
|-- 11.jpg
|-- CMakeLists.txt
|-- include
|   |-- clipper.h
|   |-- cls_process.h
|   |-- crnn_process.h
|   `-- db_post_process.h
|-- LICENSE
|-- models
|   |-- cls-sim-op.bin
|   |-- cls-sim-op.param
|   |-- config.txt
|   |-- PP_OCRv5_mobile_det.bin
|   |-- PP_OCRv5_mobile_det.param
|   |-- PP_OCRv5_mobile_rec.bin
|   |-- PP_OCRv5_mobile_rec.param
|   |-- PP_OCRv5_server_det.bin
|   |-- PP_OCRv5_server_det.param
|   |-- PP_OCRv5_server_rec.bin
|   |-- PP_OCRv5_server_rec.param
|   `-- PP_OCRv5_vocab.txt
|-- PaddleOCR-Lite-Doc.cbp
|-- PaddleOCR-Lite-Doc.depend
|-- PaddleOCR-Lite-Doc.layout
|-- README.md
|-- src
|   |-- clipper.cpp
|   |-- cls_process.cc
|   |-- crnn_process.cc
|   |-- db_post_process.cc
|   `-- main.cpp
|-- WillekePass.jpg
`-- word_1.jpg
```
Load the `PaddleOCR-Lite-Doc.cbp` project file in Code::Blocks and build the app.<br>

------------

## CMake.
Instead of Code::Blocks, you can also use CMake to build the application.<br>
Please follow the next instructions. Assuming you're in the 'main' directory.<br>
```bash
$ mkdir build
$ cd build
$ cmake ..
$ make  -j4
$ cd ..
```
------------

## Running the app.
Once successfully built, you will find the executable **ocr**.

The parameter list:
```
./ocr Mode Detection model file Orientation classifier model file Recognition model file  Hardware  Precision Threads Batchsize  Test image path Dictionary  
```
To detect the locations of text in an image:
```bash
./ocr det ./models/PP_OCRv5_mobile_det arm8 FP32 4 1 ./11.jpg ./models/config.txt
```
To ocr a single word:
```bash
./ocr rec ./models/PP_OCRv5_mobile_rec arm8 FP32 4 1 ./word_1.jpg ./models/PP_OCRv5_vocab.txt ./models/config.txt
```
To process a document word:
```bash
./ocr system ./models/PP_OCRv5_mobile_det ./models/PP_OCRv5_mobile_rec ./models/cls-sim-op arm8 FP32 4 1 ./WillekePass.jpg ./models/config.txt ./models/PP_OCRv5_vocab.txt
```
------------

## Notes.

1.  `config.txt` of the detector and classifier, as shown below:
```
max_side_len  960          #  Limit the maximum image height and width to 960
det_db_thresh  0.3         # Used to filter the binarised image of DB prediction, setting 0.-0.3 has no obvious effect on the result
det_db_box_thresh  0.5     # DDB post-processing filter box threshold, if there is a missing box is detected, it can be reduced as appropriate
det_db_unclip_ratio  1.6   # Indicates the compactness of the text box; the smaller the value, the closer the text box to the text
use_direction_classify  0  # Whether to use the direction classifier, 0 means not to use, 1 means to use
rec_image_height  48       # The height of the input image of the recognition model: **must be 48**.
```


