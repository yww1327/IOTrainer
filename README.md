# IOTrainer
IOTrainer is an application working on edge devices, such as Raspberry Pi, to check poses during your workout process. It contains the functions of drawing body skeletons and showing the right position point of the workout poses. With PiCamera, IOTrainer could capture and stream in the browser. Other devices can browse the webpage by connecting on same LAN. (The method will be mentioned [below](https://github.com/yww1327/IOTrainer#4-browse-the-webpage-on-other-devices).)

## Demo
| Show Skeletons When Human Body Detected | Show Error and Right Pose Point |
| ---- | ---- |
| ![](https://github.com/yww1327/IOTrainer/blob/main/readme/skeletons.png) | ![](https://github.com/yww1327/IOTrainer/blob/main/readme/error.png) |

Check video demo [here](https://youtu.be/ZAnGCJuLjmA)!

## Hardware Required
* Raspberry Pi 3 (Check the setup manual [here](https://github.com/yww1327/IOTrainer/blob/main/readme/setupManual.pdf))
* Pi Camera
* 5V Transformer

| Needed Components | Configuration of the components |
| ---- | ---- |
| ![](https://github.com/yww1327/IOTrainer/blob/main/readme/neededHardware.jpg) | ![](https://github.com/yww1327/IOTrainer/blob/main/readme/hardwardConfig.jpg) | 

**NOTE**: Remember to enable camera configuration!!
 
![](https://github.com/yww1327/IOTrainer/blob/main/readme/cameraConfigSetup.png)

## Dependencies
* Python 3
* OpenCV (3.4.1+ to use the function ```cv2.dnn.readNetFromTensorflow```)
* imutils (for video streaming)
* Flask (to build webpage)

## Deployment
(**NOTE**: The delay in streaming could be severe due to the operation of OpenCV. The possible solutions will be listed [below](https://github.com/yww1327/IOTrainer#opencv-installation-record).)

Check tutorial video [here](https://www.youtube.com/watch?v=0wJZhFzkVaE)!


### 0. Set up all dependencies

After installing Python and OpenCV, the installation of the following package is needed. You can choose to install in base or your own virtual environment.
```
$ pip3 install imutils
$ pip3 install Flask
```

### 1. Clone this repo

Open your Terminal and go to the path where you would like to store the files.
```
pi@raspberrypi:~ $ git clone https://github.com/yww1327/IOTrainer.git
``` 
### 2. Run main.py

Run ```main.py``` to start web server so that the captured and computed streaming video would shown in the browser.
```
pi@raspberrypi:~ $ cd IOTrainer
pi@raspberrypi:~/IOTrainer $python3 main.py
```

### 3. Browse the webpage

Enter address ```0.0.0.0:5000``` in the browser. You can see the streaming video captured by PiCamera.

### 4. Browse the webpage on other devices

Find your Pi ip address first by entering the following command in terminal.
```
$ ip addr
```
Then you can see the response like the picture shown below:

![](https://github.com/yww1327/IOTrainer/blob/main/readme/ip_addr.PNG)

The number string after "inet" is your Pi ip address. In this case, the Pi ip address is ```172.20.10.5```.
Connect other devices in the same LAN that Pi is in and enter ```172.20.10.5:5000``` in address bar of your browser.

## OpenCV Installation Record
* I first installed OpenCV by pip.
```
$ sudo pip3 install opencv-python
```
However, when I ran my project with it, there was a severe delay in streaming. And I found [this solution](https://stackoverflow.com/questions/34276070/why-compile-opencv-with-qt) on stack overflow, which is compiling OpenCV with QT.

* I have tried to compile 3.2.0 version of OpenCV. There's a small problem came up, which is an error occured when running the command ```$ make -j4```. But this problem can be solved like [this](https://blog.csdn.net/qq_44357371/article/details/105966714). After compiling, it did work, but the function ```cv2.dnn.readNetFromTensorflow``` is not usable in this version. The function is availble in OpenCV 3.4.1+.

* Then I tried 4.2.0 version, but it stuck when I run the command ```$ make -j4```. I didn't find any solution to the problem, so I try another version.

* I tried 4.1.0 instead and succeed. The following steps are the way I compiled OpenCV 4.1.0 with QT by modifying the a few compiling steps [here](https://nancyyluu.blogspot.com/2017/12/raspberry-pi-opencvcontrib.html?fbclid=IwAR04es5w9Q44z1S-1ftq1_eWM-9EyT41oP0b8DH991P_87MF0ddEbGxG9PY):

Install cmake and QT5
```
$ sudo apt-get install cmake
$ sudo apt-get install -y qt5-default qtcreator
```

Download OpenCV 4.1.0 install package on gitHub
```
pi@raspberrypi:~ $ wget -O opencv.zip https://github.com/opencv/opencv/archive/4.1.0.zip
pi@raspberrypi:~ $ wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.1.0.zip
pi@raspberrypi:~ $ unzip opencv.zip
pi@raspberrypi:~ $ unzip opencv_contrib.zip
```

Create a folder to store the files after running cmake
```
pi@raspberrypi:~ $ cd opencv-4.1.0
pi@raspberrypi:~/opencv-4.1.0 $ mkdir build
pi@raspberrypi:~/opencv-4.1.0 $ cd build
```

Make file (**NOTE**: FFMPEG is not available on Raspberry Pi)
```
pi@raspberrypi:~/opencv-4.1.0/build $ cmake -D WITH_QT=ON
	-D WITH_FFMPEG=OFF 
	-D CMAKE_BUILD_TYPE=RELEASE \
	-D CMAKE_INSTALL_PREFIX=/usr/local \
	-D BUILD_WITH_DEBUG_INFO=OFF \
	-D BUILD_DOCS=OFF \
	-D BUILD_EXAMPLES=OFF \
	-D BUILD_TESTS=OFF \
	-D BUILD_opencv_ts=OFF \
	-D BUILD_PERF_TESTS=OFF \
	-D INSTALL_C_EXAMPLES=ON \
	-D INSTALL_PYTHON_EXAMPLES=ON \
	-D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib-4.1.0/modules \
	-D ENABLE_NEON=ON \
	-D WITH_LIBV4L=ON \
	      ../
```

Compile OpenCV
```
pi@raspberrypi:~/opencv-4.1.0/build $ make -j4
pi@raspberrypi:~/opencv-4.1.0/build $ sudo install make
pi@raspberrypi:~/opencv-4.1.0/build $ sudo ldconfig
```

Comparison (Different in FPS)
| Compile with GTK (About 0.5fps) | Compile with QT (About 1fps) |
| ---- | ---- |
| ![](https://github.com/yww1327/IOTrainer/blob/main/readme/OpenCVwithGTK.gif) | ![](https://github.com/yww1327/IOTrainer/blob/main/readme/OpenCVwithQT.gif) |


## Code Review
```graph_opt.pb``` The model weight trained by Tensorflow

```main.py``` Contain functions of flask

```camera.py``` Contain the function of estimation of poses and video streaming object

```template/index.html``` The layout of the webpage


I've added some comments on the documents. The following part will be the simple and brief explanation of the structure of my code.

* Use Flask --```main.py``` line [14](https://github.com/yww1327/IOTrainer/blob/main/main.py#L14)

```
app = Flask(__name__)
```

* Routing --```main.py``` line [18](https://github.com/yww1327/IOTrainer/blob/main/main.py#L18), [44](https://github.com/yww1327/IOTrainer/blob/main/main.py#L44), [50](https://github.com/yww1327/IOTrainer/blob/main/main.py#L50), [56](https://github.com/yww1327/IOTrainer/blob/main/main.py#L56)

> This is a way like create pages for the website. So the website will contain the addresses like

> 0.0.0.0:5000/
>
> 0.0.0.0:5000/shoulder_press
>
> 0.0.0.0:5000/chest_press
>
> 0.0.0.0:5000/plank

```
@app.route('/')
# define the function that will be called when the page rendered
def index():
    return render_template('index.html')		   
```

* Render layout --```main.py``` line [20](https://github.com/yww1327/IOTrainer/blob/main/main.py#L20)

> After the function called, the layout will be rendered. **NOTE**: The ```html``` file should be store in a folder named templates which is in the same path as ```main.py```.

```
return render_template('index.html')
```

* Edit Host and Port --```main.py``` line [64](https://github.com/yww1327/IOTrainer/blob/main/main.py#L64)

> 0.0.0.0 means all IPv4 address on local machine.
```
app.run(host='0.0.0.0', debug=False)
```

* Render Streaming Video --```index.html``` line [135](https://github.com/yww1327/IOTrainer/blob/af824f83ca0aff00c38043d5d3da6e6b999a50d2/templates/index.html#L135), [136](https://github.com/yww1327/IOTrainer/blob/af824f83ca0aff00c38043d5d3da6e6b999a50d2/templates/index.html#L136)

> Since /shoulder_press define a function to get frames from the streaming video, using url_for to render the frames in img tag.

```
<img class="camera-b flex" style="background-attachment: fixed; position: relative ; top: 80px;" id="bg" class="center" src="{{ url_for('shoulder_press') }}">
```



## Reference
### About Pose Estimation

* [Machine learning for everyone: How to implement pose estimation in a browser using your webcam](https://thenextweb.com/syndication/2020/02/01/machine-learning-for-everyone-how-to-implement-pose-estimation-in-a-browser-using-your-webcam/)

* [Pose Estimation with TensorFlow 2.0](https://medium.com/@gsethi2409/pose-estimation-with-tensorflow-2-0-a51162c095ba)

* [Real Time Human Pose Estimation on the edge with Movidius NCS and OpenVINO](https://medium.com/@oviyum/real-time-human-pose-estimation-on-the-edge-with-movidius-ncs-and-openvino-ac3b13536)

* [HIIT PI](https://github.com/jingw222/hiitpi)

### About OpenCV

* [Deep Learning based Human Pose Estimation using OpenCV ( C++ / Python )](https://www.learnopencv.com/deep-learning-based-human-pose-estimation-using-opencv-cpp-python/)

### About Streaming on Raspberry pi

* [PiCamera Doucument -Web Streaming](https://picamera.readthedocs.io/en/latest/recipes2.html#web-streaming)

* [Video Streaming On Flask Server Using RPi](https://www.hackster.io/ruchir1674/video-streaming-on-flask-server-using-rpi-ef3d75)
