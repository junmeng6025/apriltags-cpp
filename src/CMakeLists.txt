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

target_link_libraries(apriltags ${OpenCV_LIBS})
target_link_libraries(apriltags ${AT_LIBS})

add_executable(camtest camtest.cpp)
target_link_libraries(camtest ${OpenCV_LIBS})
target_link_libraries(camtest ${AT_LIBS})

add_executable(tagtest tagtest.cpp)
target_link_libraries(tagtest ${OpenCV_LIBS})
target_link_libraries(tagtest ${AT_LIBS})

add_executable(quadtest quadtest.cpp)
target_link_libraries(quadtest ${OpenCV_LIBS})
target_link_libraries(quadtest ${AT_LIBS})

if(GLUT_LIBRARY)
  add_executable(gltest gltest.cpp)
  target_link_libraries(gltest ${GLUT_LIBRARY} ${OPENGL_LIBRARY} ${AT_LIBS})
endif()

if(CAIRO_FOUND)
  add_executable(maketags maketags.cpp)
  target_link_libraries(maketags ${CAIRO_LIBRARIES} ${AT_LIBS} ${CAIRO_LIBS})
endif()
