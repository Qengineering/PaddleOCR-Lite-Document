# PaddleOCR Lite document scanner
![output image]( https://qengineering.eu/github/WillekePassOut.webp )<br>
_Image source: [Dutch  government](https://www.rijksoverheid.nl/actueel/nieuws/2021/08/02/nieuw-model-nederlandse-identiteitskaart-per-2-augustus-2021)._<br><br>
[![License](https://img.shields.io/badge/License-BSD%203--Clause-blue.svg)](https://opensource.org/licenses/BSD-3-Clause)<br/><br/>
Paper: [E-book: _Dive Into OCR_ (PDF)](https://paddleocr.bj.bcebos.com/ebook/Dive_into_OCR.pdf)<br/><br/>
PaddleOCR is a text engine. It can detect text in a scene, determine the orientation, and recognize the characters (Chinese or Latin).<br>
To do so, it needs a deep learning framework, like PaddlePaddle or Paddle-Lite.<br>
This application uses Paddle-Lite because, as the name already suggests, it is a lightweight version of PaddlePaddle, it can deploy small quantized models (INT8), and its architecture is well suited for the ARMv8 processors found on a Raspberry Pi.<br>
Once all the software is installed, it can run on a bare Raspberry Pi without needing cloud services or any (expensive) license.<br><br>
INference time (RPi 4 @ 1925 MHz - 64 bits Bullseye OS):<br>
Detect text: 366 mSec.<br>
Recognize text: 134 mSec per line.<br><br>
Special made for a bare Raspberry Pi 4, see [Q-engineering deep learning examples](https://qengineering.eu/deep-learning-examples-on-raspberry-32-64-os.html)

------------

## Dependencies.
To run the application, you have to:
- A raspberry Pi 4 with a 32 or 64-bit operating system. It can be the Raspberry 64-bit OS, or Ubuntu 18.04 / 20.04. [Install 64-bit OS](https://qengineering.eu/install-raspberry-64-os.html) <br/>
- PaddleOCR installed.
- Paddle-Lite optimizer. [Install Paddle-Lite](https://qengineering.eu/install-paddle-lite-on-raspberry-pi-4.html) <br/>
- Paddle-Lite optimizer. [Install Paddle-Lite](https://qengineering.eu/install-paddle-lite-on-raspberry-pi-4.html) <br/>
- Clipper installed.
- OpenCV 64 bit installed. [Install OpenCV 4.5](https://qengineering.eu/install-opencv-4.5-on-raspberry-64-os.html) <br/>
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
#### Clipper
PaddleOCR uses Clipper, a library for clipping and shifting polygons. You can see it in action in the photo above. Clipper2 is currently the latest version. However, PaddleOCR uses the first version, which fortunately is still available from [SourceForge](https://sourceforge.net/projects/polyclipping/).
```
Download clipper and unzip in a folder named <clipper>
.
├── clipper
│   ├── C#
│   ├── cpp
│   ├── Delphi
│   ├── Documentation
│   ├── License.txt
│   ├── README
│   └── Third Party

$ cd clipper/cpp
$ mkdir build && cd build
$ cmake ..
$ make -j4
$ sudo cp -r ./libpolyclipping.so.22.0.0 /usr/lib
$ sudo ln -s /usr/lib/libpolyclipping.so.22.0.0 /usr/lib/libpolyclipping.so.22
$ sudo ln -s /usr/lib/libpolyclipping.so /usr/lib/libpolyclipping.so.22
$ cd ..
$ sudo mkdir /usr/local/include/polyclipping
$ sudo cp -r ./clipper.hpp /usr/local/include/polyclipping
```
#### Paddle-Lite framework
With Clipper up and running, it's now time to install the Paddle-Lite framework. See also our [guide](https://qengineering.eu/install-paddle-lite-on-raspberry-pi-4.html).
```
# install dependencies
$ sudo apt-get install cmake wget
# download Paddle Lite
$ git clone --depth=1 https://github.com/PaddlePaddle/Paddle-Lite.git
$ cd Paddle-Lite
# build 64-bit Paddle Lite (±1 hour)
$ ./lite/tools/build_linux.sh --arch=armv8 --with_extra=ON --with_cv=ON --with_static_lib=ON
# Once build, you will have the following directories:
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
$ tar xf  ch_PP-OCRv2_rec_slim_quant_infer.tar
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

```
$git clone --depth=1 https://github.com/PaddlePaddle/PaddleOCR.git
```

------------

## Running the app.

------------

[![paypal](https://qengineering.eu/images/TipJarSmall4.png)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=CPZTM5BB3FCYL) 


