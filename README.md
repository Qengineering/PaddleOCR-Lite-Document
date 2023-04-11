# PaddleOCR Lite document scanner
![output image]( https://qengineering.eu/github/WillekePassOut.webp )<br>
_Image source: [Dutch  government](https://www.rijksoverheid.nl/actueel/nieuws/2021/08/02/nieuw-model-nederlandse-identiteitskaart-per-2-augustus-2021)._<br><br>
[![License](https://img.shields.io/badge/License-BSD%203--Clause-blue.svg)](https://opensource.org/licenses/BSD-3-Clause)<br/><br/>
Paper: [E-book: _Dive Into OCR_ (PDF)](https://paddleocr.bj.bcebos.com/ebook/Dive_into_OCR.pdf)<br/><br/>
PaddleOCR is a text engine. It can detect text in a scene, determine the orientation, and recognize the characters (Chinese or Latin).<br>
To do so, it needs a deep learning framework, like PaddlePaddle or Paddle-Lite.<br>
This application uses Paddle-Lite because, as the name already suggests, it is a lightweight version of PaddlePaddle, it can deploy small quantized models (INT8), and its architecture is well suited for the ARMv8 processors found on a Raspberry Pi.<br>
Once all the software is installed, it can run on a bare Raspberry Pi without needing cloud services or any (expensive) license.<br><br>
Frame rate (RPi 4 @ 1925 MHz - 64 bits Bullseye OS):<br>
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
------------

## Running the app.

------------

[![paypal](https://qengineering.eu/images/TipJarSmall4.png)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=CPZTM5BB3FCYL) 


