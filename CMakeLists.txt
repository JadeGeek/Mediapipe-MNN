cmake_minimum_required(VERSION 3.27)

project(Mediapipe-MNN)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17 -O3")

set(3RD_DIR ${CMAKE_CURRENT_LIST_DIR}/3rdLibs)
set(SRC_DIR ${CMAKE_CURRENT_LIST_DIR}/source)

add_subdirectory(${3RD_DIR}/MNN-2.3.0)

file(GLOB_RECURSE SRC_FILES
    ${SRC_DIR}/*.h
    ${SRC_DIR}/*.cc
)

if(WIN32)
  message("Windows Platform")

  include_directories(
    ${SRC_DIR}
    ${3RD_DIR}/MNN-2.3.0/include
    ${3RD_DIR}/opencv-3.4.2/include
  )
  link_directories(${CMAKE_CURRENT_LIST_DIR}/3rdLibs/opencv-3.4.2/lib)
  list(APPEND OBJECTS_TO_LINK opencv_world342)

elseif(APPLE)
  message("Apple Platform")

  find_library(opencv_world342 OpenCV2 "${3RD_DIR}/opencv2.xcframework/macos-arm64_x86_64/")
  if (NOT opencv_world342)
    message(FATAL_ERROR "OPENCV_MOBILE not found")
  endif()

  include_directories(
    ${SRC_DIR}
    ${3RD_DIR}/MNN-2.3.0/include
    ${3RD_DIR}/opencv-3.4.2/include
  )

elseif(UNIX)
  message("Unix Platform")
endif()

add_library(${PROJECT_NAME} STATIC ${SRC_FILES})
target_link_libraries(${PROJECT_NAME} MNN ${opencv_world342})

set(EXAMPLE_LIST
    TestPalmDetection
    TestFaceDetection
    TestFaceDetectorLite
    TestFaceDetectorYN
    TestImageEmbedding
    # TestFeatureDetector # ERROR:ld: symbol(s) not found for architecture arm64
    TestPoseDetector
    TestClassifier
    # TestSegmentDetector # ERROR:ld: symbol(s) not found for architecture arm64
)

foreach(EXAMPLE ${EXAMPLE_LIST})
  add_executable(${EXAMPLE} examples/${EXAMPLE}.cc)
  if(WIN32)
    target_link_libraries(${EXAMPLE} ${PROJECT_NAME} ${OBJECTS_TO_LINK})
  elseif(APPLE)
    target_link_libraries(${EXAMPLE} ${PROJECT_NAME} ${opencv_world342})
  elseif(UNIX)
    message("Unix Platform")
    target_link_libraries(${EXAMPLE} ${PROJECT_NAME} ${opencv_world342})
  endif()
endforeach(EXAMPLE ${EXAMPLE_LIST})

