cmake_minimum_required(VERSION 3.15)
project(test-darts)

set(CMAKE_CXX_STANDARD 14)

set(DARTS_H_SRC
        ${CMAKE_CURRENT_SOURCE_DIR}/include/*.h)

set(DARTS_CPP_SRC
        ${CMAKE_CURRENT_SOURCE_DIR}/test/test-darts.cc)

set(DARTS_INCLUDE_PATH
        ${CMAKE_CURRENT_SOURCE_DIR}/include)

add_executable(test-darts ${DARTS_CPP_SRC}
        ${DARTS_CPP_SRC})

target_include_directories(test-darts
        PUBLIC
        ${DARTS_INCLUDE_PATH})

