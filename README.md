# Ground camera AprilTag detect (C++)

# Pre-requisitions
## OpenCV 3.4.16
- Install OpenCV following [this guide](https://www.geeksforgeeks.org/how-to-install-opencv-in-c-on-linux/); Only by the 2nd step we choose `3.4.16` instead of `4.6.0`
- after all the percedures, check your opencv version with this command:
    ```bash
    pkg-config --cflags opencv
    ```
    it should return the version number `3.4.16`
    **IF NOT**:
    e.g. returns a message like this:
    ```bash
    Package opencv was not found in the pkg-config search path.
    Perhaps you should add the directory containing `opencv.pc'
    to the PKG_CONFIG_PATH environment variable
    No package 'opencv' found
    ```
    It might because your opencv not yet added to the path correctly. follow the solotion [here](https://stackoverflow.com/questions/15320267/package-opencv-was-not-found-in-the-pkg-config-search-path)  

  - cd to `/usr/local/lib/pkgconfig`, open in terminal
  - ```bash
      touch opencv.pc
      sudo gedit opencv.pc  # here only whith sudo you can save your changes!
      ```
  - the contents of `opencv.pc` is:
      ```
      # Package Information for pkg-config

      prefix=/usr/local
      exec_prefix=${prefix}
      libdir=${exec_prefix}/lib
      includedir_old=${prefix}/include/opencv
      includedir_new=${prefix}/include

      Name: OpenCV
      Description: Open Source Computer Vision Library
      Version: 3.4.16
      Libs: -L${exec_prefix}/lib -lopencv_dnn -lopencv_highgui -lopencv_ml -lopencv_objdetect -lopencv_shape -lopencv_stitching -lopencv_superres -lopencv_videostab -lopencv_calib3d -lopencv_videoio -lopencv_imgcodecs -lopencv_features2d -lopencv_video -lopencv_photo -lopencv_imgproc -lopencv_flann -lopencv_viz -lopencv_core
      Libs.private: -ldl -lm -lpthread -lrt
      Cflags: -I${includedir_old} -I${includedir_new}
      ```
  - check again using command `pkg-config --cflags opencv`
- Now you should have finished with the installation and setup for OpenCV!
## Install `canberra-gtk-module`
  ```bash
  sudo apt-get install libcanberra-gtk-module
  ```
<br/>

# Install and Build
## 1) Clone the repo
 clone this git repo: https://github.com/swatbotics/apriltags-cpp  
and thanks @swatbotics for their awesome work!
## 2) Modify the `CMakelists.txt` in the `/src` folder:  
> ATTENTION: don't mix with the CMakeLists.txt in the root folder  

add `target_link_libraries(<main_name> ${OpenCV_LIBS})` for **EVERY** executable's build task. 
(or for convience you can simply copy the following and substitute all the contents of your original CMakeLists.txt in /src)
```cmake
add_library(apriltags
CameraUtil.cpp
DebugImage.cpp
Geometry.cpp
GrayModel.cpp
MathUtil.cpp
Refine.cpp
TagDetector.cpp
TagFamily.cpp
TagFamilies.cpp
UnionFindSimple.cpp
)

find_package(OpenCV REQUIRED)

include_directories(${OpenCV_INCLUDE_DIRS}) # not needed for opencv>=4.0

set(AT_LIBS apriltags ${OPENCV_LDFLAGS})

target_link_libraries(apriltags ${OpenCV_LIBS}) #
target_link_libraries(apriltags ${AT_LIBS})

add_executable(camtest camtest.cpp)
target_link_libraries(camtest ${OpenCV_LIBS}) #
target_link_libraries(camtest ${AT_LIBS})

add_executable(tagtest tagtest.cpp)
target_link_libraries(tagtest ${OpenCV_LIBS}) #
target_link_libraries(tagtest ${AT_LIBS})

add_executable(quadtest quadtest.cpp)
target_link_libraries(quadtest ${OpenCV_LIBS}) #
target_link_libraries(quadtest ${AT_LIBS})

if(GLUT_LIBRARY)
    add_executable(gltest gltest.cpp)
    target_link_libraries(gltest ${GLUT_LIBRARY} ${OPENGL_LIBRARY} ${AT_LIBS})
endif()

if(CAIRO_FOUND)
    add_executable(maketags maketags.cpp)
    target_link_libraries(maketags ${CAIRO_LIBRARIES} ${AT_LIBS} ${CAIRO_LIBS})
endif()

```
## 3) Compile the code
cd to the root folder `/apriltags-cpp`
```bash
mkdir build
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make
```
### Possible issues
- ### missing some packages
- ### errors like "xxx was not declared in this scope"
  like this:
  ```bash
  /home/jun/apriltag_ros2/apriltags-cpp/src/gltest.cpp:490:18: error: ‘CV_CAP_PROP_FRAME_WIDTH’ was not declared in this scope
  490 |     capture->set(CV_CAP_PROP_FRAME_WIDTH, opts.frame_width);
      |                  ^~~~~~~~~~~~~~~~~~~~~~~
  /home/jun/apriltag_ros2/apriltags-cpp/src/gltest.cpp:491:18: error: ‘CV_CAP_PROP_FRAME_HEIGHT’ was not declared in this scope
  491 |     capture->set(CV_CAP_PROP_FRAME_HEIGHT, opts.frame_height);
      |                  ^~~~~~~~~~~~~~~~~~~~~~~~
  ```
  This is because of the different OpenCV version. The variable names in this project's scripts are not consistent with them of our OpenCV library.  
  You can modify them one by one according to the error messages (that is exactly how I managed to build it ... x_x)  
  Some common substitutions:
  ```cpp
  CV_AA -> cv::LINE_AA
  CV_CAP_PROP_FRAME_HEIGHT -> cv::CAP_PROP_FRAME_HEIGHT
  CV_CAP_PROP_FRAME_WIDTH -> cv::CAP_PROP_FRAME_WIDTH
  CV_INTER_AREA-> cv::INTER_AREA
  CV_RGB2GRAY -> cv::COLOR_RGB2GRAY
  ...
  ```
  You can search them on Google if you don't know the variable name in new version OpneCV
## Run
### tagtest
```bash
# stay in /apriltags-cpp/build
./tagtest ../images/iphonecam/IMG_0612.jpg
```
### camtest
```bash
# stay in /apriltags-cpp/build
./camtest
```
You can modify the device id in the source code `/src/camtest.cpp` and then recompile:
```cpp
typedef struct CamTestOptions {
  CamTestOptions() :
      params(),
      family_str(DEFAULT_TAG_FAMILY),
      error_fraction(1),
      //device_num(0),  // 0 is the laptop's built-in webcam
      device_num(2),  // here 2 is our external SONY a7r4
      focal_length(500),
      tag_size(0.1905),
      frame_width(0),
      frame_height(0),
      mirror_display(true)
  {
  }
  TagDetectorParams params;
  std::string family_str;
  double error_fraction;
  int device_num;
  double focal_length;
  double tag_size;
  int frame_width;
  int frame_height;
  bool mirror_display;
} CamTestOptions;
```
