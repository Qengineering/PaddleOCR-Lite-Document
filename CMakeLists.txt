cmake_minimum_required(VERSION 3.0 FATAL_ERROR)

project(ocr_db_crnn)

macro(use_cxx11)
  if (CMAKE_VERSION VERSION_LESS "3.1")
    if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
      set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17")
    endif ()
  else ()
    set (CMAKE_CXX_STANDARD 17)
  endif ()
endmacro(use_cxx11)

find_package(OpenCV REQUIRED)                              

set(SOURCES src/clipper.cpp src/cls_process.cc src/crnn_process.cc src/db_post_process.cc src/ocr_db_crnn.cc)

set(EXTRA_LIBS ${OpenCV_LIBS} /usr/local/lib/paddle-lite/libpaddle_api_light_bundled.a)

set(EXECUTABLE_OUTPUT_PATH "../")

include_directories(include ${OpenCV_INCLUDE_DIRS} /usr/local/include/paddle-lite)

add_link_options(-fopenmp -s -ldl -pthread -lgomp -rdynamic)

add_executable(ocr_db_crnn ${SOURCES})

target_link_libraries(ocr_db_crnn ${EXTRA_LIBS})