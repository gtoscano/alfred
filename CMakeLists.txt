cmake_minimum_required(VERSION 3.14)
project(alfred)

include(FetchContent)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

SET(CMAKE_CXX_FLAGS "-g")

# -lboost_log
set(Boost_USE_STATIC_LIBS OFF)
set(Boost_USE_MULTITHREADED ON)
set(Boost_USE_STATIC_RUNTIME OFF)
set(THREADS_PREFER_PTHREAD_FLAG ON)

# -DBOOST_LOG_DYN_LINK
# definitions
add_definitions(-DBOOST_ALL_NO_LIB)
add_definitions(-DBOOST_LOG_DYN_LINK=1)

# -lboost_log -lboost_thread -lboost_log_setup
find_package(Boost 1.74.0 COMPONENTS thread log log_setup REQUIRED)
find_package(PkgConfig)

pkg_check_modules(PROTOBUF REQUIRED IMPORTED_TARGET protobuf)
pkg_check_modules(SNAPPY REQUIRED IMPORTED_TARGET snappy)
pkg_check_modules(CURLPP REQUIRED IMPORTED_TARGET curlpp)
pkg_check_modules(LIBCURL REQUIRED IMPORTED_TARGET libcurl)

#set(Parquet_DIR /usr/lib/x86_64-linux-gnu/cmake/arrow)
find_package(Parquet REQUIRED)

# -lpthread
find_package(Threads REQUIRED)
# -larrow
find_package(Arrow CONFIG REQUIRED)
# -lparquet
find_package(Parquet CONFIG REQUIRED)
# sudo apt-get install uuid-dev
# -laws-cpp-sdk-core
find_package(aws-cpp-sdk-core CONFIG REQUIRED)
# -laws-cpp-sdk-s3
find_package(aws-cpp-sdk-s3 CONFIG REQUIRED)
find_package(Protobuf MODULE REQUIRED)

set(SOURCE_DIR "${CMAKE_SOURCE_DIR}/src")
set(INCLUDE_DIR "${CMAKE_SOURCE_DIR}/include")

add_executable(alfred
    ${SOURCE_DIR}/alfred.cpp 
    ${SOURCE_DIR}/config.cpp 
    ${SOURCE_DIR}/utils.cpp 
    ${SOURCE_DIR}/myawss3.cpp 
    ${SOURCE_DIR}/myamqp.cpp 
    ${SOURCE_DIR}/file_conversion.cpp
)

target_include_directories(alfred PUBLIC include)
target_link_libraries(alfred PUBLIC Boost::log Boost::log_setup hiredis redis++ fmt arrow_shared parquet aws-cpp-sdk-core aws-cpp-sdk-s3 SimpleAmqpClient Threads::Threads PkgConfig::LIBCURL)
#PkgConfig::PROTOBUF PkgConfig::SNAPPY)

install(TARGETS alfred RUNTIME DESTINATION /usr/local/bin)

