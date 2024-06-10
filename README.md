# PaddleOCR Lite document scanner
![output image]( https://qengineering.eu/github/WillekePassOut.webp )<br>
_Image source: [Dutch  government](https://www.rijksoverheid.nl/actueel/nieuws/2021/08/02/nieuw-model-nederlandse-identiteitskaart-per-2-augustus-2021)._<br><br>
[![License](https://img.shields.io/badge/License-BSD%203--Clause-blue.svg)](https://opensource.org/licenses/BSD-3-Clause)<br/><br/>
Paper: [E-book: _Dive Into OCR_ (PDF)](https://paddleocr.bj.bcebos.com/ebook/Dive_into_OCR.pdf)<br/><br/>
PaddleOCR is a text engine. It can detect text in a scene, determine the orientation, and recognize the characters (Chinese or Latin).<br>
To do so, it needs a deep learning framework, like PaddlePaddle or Paddle-Lite.<br>
This application uses Paddle-Lite because, as the name already suggests, it is a lightweight version of PaddlePaddle, it can deploy small quantized models (INT8), and its architecture is well suited for the ARMv8 processors found on a Raspberry Pi.<br>
Once all the software is installed, it can run on a bare Raspberry Pi without needing cloud services or any (expensive) license.<br><br>
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
- A raspberry Pi 4 with a 32 or 64-bit operating system. It can be the Raspberry 64-bit OS, or Ubuntu 18.04 / 20.04. [Install 64-bit OS](https://qengineering.eu/install-raspberry-64-os.html) <br/>
- PaddleOCR installed.
- Paddle-Lite optimizer. [Install Paddle-Lite](https://qengineering.eu/install-paddle-lite-on-raspberry-pi-4.html) <br/>
- OpenCV 64-bit installed. [Install OpenCV 4.5](https://qengineering.eu/install-opencv-4.5-on-raspberry-64-os.html) <br/>
- Code::Blocks installed. (```$ sudo apt-get install codeblocks```)

------------

## Installing the app.
Before you can run the application, you need to install quite a lot of software. Let's start.
#### OpenCV
The first package you need is OpenCV. On our website, there are [several guides](https://qengineering.eu/install-opencv-4.5-on-raspberry-64-os.html) on how to install OpenCV on your Raspberry Pi.<br>
It shouldn't be a problem if you follow the instructions carefully.
#### Code::Blocks
For compilation, you can choose between CMake or Code::Blocks.
We find Code::Blocks more comfortable to work with than CMake because of its IDE, which allows quick changes.
```
$ sudo apt-get install codeblocks
```
#### Paddle-Lite framework
It's time to install the Paddle-Lite framework. See also our [guide](https://qengineering.eu/install-paddle-lite-on-raspberry-pi-4.html).
```
# install dependencies
$ sudo apt-get install cmake wget
# download Paddle Lite
$ git clone --depth=1 https://github.com/PaddlePaddle/Paddle-Lite.git
$ cd Paddle-Lite
# build 64-bit Paddle Lite (±1 hour)
$ ./lite/tools/build_linux.sh --arch=armv8 --with_extra=ON --with_cv=ON --with_static_lib=ON
# Once built, you will have the following directories:
Paddle-Lite/inference_lite_lib.android.armv8/
├── cxx                                        C++ prebuild library
|   ├── include                                C++
|   |   ├── paddle_api.h
|   |   ├── paddle_image_preprocess.h
|   |   ├── paddle_lite_factory_helper.h
|   |   ├── paddle_place.h
|   |   ├── paddle_use_kernels.h
|   |   ├── paddle_use_ops.h
|   |   └── paddle_use_passes.h
|   └── lib                                           C++ library
|       ├── libpaddle_api_light_bundled.a             C++ static library
|       └── libpaddle_light_api_shared.so             C++ dynamic library

# copy the headers and library to /usr/local/
$ sudo mkdir -p /usr/local/include/paddle-lite
$ sudo cp -r build.lite.linux.armv8.gcc/inference_lite_lib.armlinux.armv8/cxx/include/*.* /usr/local/include/paddle-lite
$ sudo mkdir -p /usr/local/lib/paddle-lite
$ sudo cp -r build.lite.linux.armv8.gcc/inference_lite_lib.armlinux.armv8/cxx/lib/*.* /usr/local/lib/paddle-lite
```
#### Paddle-Lite optimizer
In addition to the Paddle-Lite library, we also need the optimization tool `opt`.<br>
The models used by PaddleOCR must match the version of the Paddle-Lite framework. No doubt Paddle-Lite will evolve, and the models provided here will no longer match the newer version.<br><br>
**👉 Please check issue https://github.com/Qengineering/PaddleOCR-Lite-Document/issues/1 for more info and recent solutions! 👈** <br><br>
Before you can compile the optimizer, please check [pull requist #10164](https://github.com/PaddlePaddle/Paddle-Lite/pull/10164).<br>
The `Paddle-Lite/lite/tools/build_linux.sh` need to be adapted for the aarch64 OS.<br><br>
![output image](https://qengineering.eu/github/Paddle_issue_10102.png)<br><br>
Perhaps, in the near future, the pull request was granted. Until that moment, please alter the `Paddle-Lite/lite/tools/build_linux.sh` file according to the suggestions above.<br>
With the `build_linux.sh` script up to date, you can buid the optimizer with the following commands.
```
$ cd Paddle-Lite
$ ./lite/tools/build_linux.sh --arch=armv8 --with_extra=ON --with_cv=ON build_optimize_tool
# Once build, you will find the tool here: Paddle-Lite/build.opt/lite/api/opt
```
Introduction to paddle_lite_opt parameters:

|Options|Description|
|---|---|
|--model_dir|The path of the PaddlePaddle model to be optimized (non-combined form)|
|--model_file|The network structure file path of the PaddlePaddle model (combined form) to be optimized|
|--param_file|The weight file path of the PaddlePaddle model (combined form) to be optimized|
|--optimize_out_type|Output model type, currently supports two types: protobuf and naive_buffer, among which naive_buffer is a more lightweight serialization/deserialization implementation. If you need to perform model prediction on the mobile side, please set this option to naive_buffer. The default is protobuf|
|--optimize_out|The output path of the optimized model|
|--valid_targets|The executable backend of the model, the default is arm. Currently it supports x86, arm, opencl, npu, xpu, multiple backends can be specified at the same time (separated by spaces), and Model Optimize Tool will automatically select the best method. If you need to support Huawei NPU (DaVinci architecture NPU equipped with Kirin 810/990 Soc), it should be set to npu, arm|
|--record_tailoring_info|When using the function of cutting library files according to the model, set this option to true to record the kernel and OP information contained in the optimized model. The default is false|

#### PaddleOCR models
The next step is getting the deep learning models PaddleOCR will be using.<br>
Mostly you use three models: one for detecting the text, one for the orientation of the text and one for character recognition.<br>

```
$ cd Paddle-Lite/build.opt/lite/api

# Download the Chinese and English inference model of PP-OCRv3
$ wget  https://paddleocr.bj.bcebos.com/PP-OCRv3/chinese/ch_PP-OCRv3_det_slim_infer.tar
$ wget  https://paddleocr.bj.bcebos.com/PP-OCRv3/chinese/ch_PP-OCRv3_rec_slim_infer.tar
$ wget  https://paddleocr.bj.bcebos.com/dygraph_v2.0/slim/ch_ppocr_mobile_v2.0_cls_slim_infer.tar

#unzip the models
$ tar xf  ch_PP-OCRv3_det_slim_infer.tar
$ tar xf  ch_PP-OCRv3_rec_slim_infer.tar
$ tar xf  ch_ppocr_mobile_v2.0_cls_slim_infer.tar

# Convert detection model
./opt --model_file=./ch_PP-OCRv3_det_slim_infer/inference.pdmodel  --param_file=./ch_PP-OCRv3_det_slim_infer/inference.pdiparams  --optimize_out=./ch_PP-OCRv3_det_slim_opt --valid_targets=arm  --optimize_out_type=naive_buffer
# Convert recognition model
./opt --model_file=./ch_PP-OCRv3_rec_slim_infer/inference.pdmodel  --param_file=./ch_PP-OCRv3_rec_slim_infer/inference.pdiparams  --optimize_out=./ch_PP-OCRv3_rec_slim_opt --valid_targets=arm  --optimize_out_type=naive_buffer
# Convert angle classifier model
./opt --model_file=./ch_ppocr_mobile_v2.0_cls_slim_infer/inference.pdmodel  --param_file=./ch_ppocr_mobile_v2.0_cls_slim_infer/inference.pdiparams  --optimize_out=./ch_ppocr_mobile_v2.0_cls_slim_opt --valid_targets=arm  --optimize_out_type=naive_buffer
```
![output image](https://qengineering.eu/github/PaddleOCRoptimizer.png)<br>
The PaddleOCR engine needs the `*.nd` files later on.
#### PaddleOCR
The last action is to download the PaddleOCR software. PaddleOCR is basically a large package of algorithms written in various computer languages, such as C++ and Python. It is not a deep learning framework like PaddlePaddle or PyTorch.
```
$ git clone --depth=1 https://github.com/PaddlePaddle/PaddleOCR.git
```
![image](https://user-images.githubusercontent.com/44409029/231420269-3ef55dd6-4a2e-4ef0-bab6-6d75d72de767.png)


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
├── include                       (copy of PaddleOCR/deploy/lite/)
│   ├── clipper.h                   
│   ├── cls_process.h
│   ├── crnn_process.h
│   └── db_post_process.h
├── src                           (copy of PaddleOCR/deploy/lite/)
│   ├── clipper.cpp
│   ├── cls_process.cc
│   ├── crnn_process.cc
│   ├── db_post_process.cc
│   └── ocr_db_crnn.cc
├── models                        (found in Paddle-Lite/build.opt/lite/api)
│   ├── ch_ppocr_mobile_v2.0_cls_slim_opt.nb
│   ├── ch_PP-OCRv3_det_slim_opt.nb
│   ├── ch_PP-OCRv3_rec_slim_opt.nb
│   ├── config.txt
│   └── ppocr_keys_v1.txt
├── PaddleOCR-Lite-Doc.cbp
└── WillekePass.jpg
```
As you can see, many files are just copies of the original ones found in the Paddle-Lite example.<br>
The models are the ones you just generated with the optimizer.<br>
As said before, your models must match the Paddle-Lite version. Meaning the supplied models here will have only a short lifespan, as Paddle-Lite will evolve further.<br><br>
Load the `PaddleOCR-Lite-Doc.cbp` project file in Code::Blocks and build the app.<br>
Building the application will take a **very** long time,  more than 4 minutes. Don't despair; just be patient.<br>

------------

## Running the app.
Once successfully built you will find the executable `./ocr_db_crnn` in the `bin/Release` folder.
The parameter list:
```
./ocr_db_crnn Mode Detection model file Orientation classifier model file Recognition model file  Hardware  Precision Threads Batchsize  Test image path Dictionary  
```
Some examples. To run the app with the given passport:
```
./ocr_db_crnn system ../../models/ch_PP-OCRv3_det_slim_opt.nb  ../../models/ch_PP-OCRv3_rec_slim_opt.nb  ../../models/ch_ppocr_mobile_v2.0_cls_slim_opt.nb  arm8 INT8 4 1  ../../WillekePass.jpg  ../../models/config.txt  ../../models/ppocr_keys_v1.txt  True
```
Only using detection model
```
./ocr_db_crnn  det ../../models/ch_PP-OCRv3_det_slim_opt.nb arm8 INT8 4 1 ../../11.jpg  ../../models/config.txt
```
Only using recognition model
```
./ocr_db_crnn  rec ../../models/ch_PP-OCRv3_rec_slim_opt.nb arm8 INT8 4 1 ../../word_1.jpg ../../models/ppocr_keys_v1.txt ../../models/config.txt
```

------------

## CMake.
Instead of Code::Blocks, you can also use CMake to build the application.<br>
Please follow the next instructions. Assuming your in the 'main' directory.<br>
```
$ mkdir build
$ cd build
$ cmake ..
$ make  -j4

Scanning dependencies of target ocr_db_crnn
[ 16%] Building CXX object CMakeFiles/ocr_db_crnn.dir/src/ocr_db_crnn.cc.o
[ 33%] Building CXX object CMakeFiles/ocr_db_crnn.dir/src/db_post_process.cc.o
[ 50%] Linking CXX executable ../ocr_db_crnn
[100%] Built target ocr_db_crnn

$ cd ..
$ tree -L 2
.
├── build
│   ├── CMakeCache.txt
│   ├── CMakeFiles
│   ├── cmake_install.cmake
│   └── Makefile
├── CMakeLists.txt
├── include
│   ├── clipper.h
│   ├── cls_process.h
│   ├── crnn_process.h
│   └── db_post_process.h
├── models
│   ├── ch_ppocr_mobile_v2.0_cls_slim_opt.nb
│   ├── ch_PP-OCRv3_det_slim_opt.nb
│   ├── ch_PP-OCRv3_rec_slim_opt.nb
│   ├── config.txt
│   └── ppocr_keys_v1.txt
├── ocr_db_crnn
├── PaddleOCR-Lite-Doc.cbp
├── PaddleOCR-Lite-Doc.depend
├── PaddleOCR-Lite-Doc.layout
├── src
│   ├── clipper.cpp
│   ├── cls_process.cc
│   ├── crnn_process.cc
│   ├── db_post_process.cc
│   └── ocr_db_crnn.cc
└── WillekePass.jpg
```
Run. Note, you are now in the 'main' directory, not in the 'bin/Release' created by Code::Blocks.
```
$ ./ocr_db_crnn system ./models/ch_PP-OCRv3_det_slim_opt.nb  ./models/ch_PP-OCRv3_rec_slim_opt.nb  ./models/ch_ppocr_mobile_v2.0_cls_slim_opt.nb  arm8 INT8 4 1  ./WillekePass.jpg  ./models/config.txt  ./models/ppocr_keys_v1.txt  True
```
Once ocr_db_crnn works, you may remove the build directory, since we don't need it any more.
```
$ sudo rm -rf build
```

------------

## Notes.
1. `ppocr_keys_v1.txt` is a Chinese dictionary file. If the nb model is used for English recognition or other language recognition, dictionary file should be replaced with a dictionary of the corresponding language. PaddleOCR provides a variety of dictionaries under ppocr/utils/, including:
```
dict/french_dict.txt     # french
dict/german_dict.txt     # german
ic15_dict.txt            # english
dict/japan_dict.txt      # japan
dict/korean_dict.txt     # korean
ppocr_keys_v1.txt        # chinese
```

2.  `config.txt` of the detector and classifier, as shown below:
```
max_side_len  960          #  Limit the maximum image height and width to 960
det_db_thresh  0.3         # Used to filter the binarized image of DB prediction, setting 0.-0.3 has no obvious effect on the result
det_db_box_thresh  0.5     # DDB post-processing filter box threshold, if there is a missing box detected, it can be reduced as appropriate
det_db_unclip_ratio  1.6   # Indicates the compactness of the text box, the smaller the value, the closer the text box to the text
use_direction_classify  0  # Whether to use the direction classifier, 0 means not to use, 1 means to use
rec_image_height  48       # The height of the input image of the recognition model, the PP-OCRv3 model needs to be set to 48, and the PP-OCRv2 model needs to be set to 32
```
Please note the importance of the **rec_image_height** parameter.<br> 
When using the PaddleOCRv2 models, the height must be 32, otherwise the recognition will be very bad.<br>
The same applies for the PaddleOCRv3 models when running with the wrong height.<br>
So, OCRv2 = 32 | OCRv3 = 48

[![paypal](https://qengineering.eu/images/TipJarSmall4.png)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=CPZTM5BB3FCYL) 


