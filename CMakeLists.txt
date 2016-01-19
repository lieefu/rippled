project(rippled)
cmake_minimum_required(VERSION 3.3)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
SET(CMAKE_FIND_LIBRARY_SUFFIXES .a .so)
include_directories(${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/libs)
#SET( CMAKE_VERBOSE_MAKEFILE on )
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++14" COMPILER_SUPPORTS_CXX14)

if(COMPILER_SUPPORTS_CXX11)
    set(STD_CXX -std=c++11)
endif()
if(COMPILER_SUPPORTS_CXX14)
    set(STD_CXX -std=c++14)
endif()
if(NOT STD_CXX)
    message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 C++14 support. Please use a different C++ compiler.")
endif()

set(CMAKE_BUILD_TYPE "Release")
if(CMAKE_BUILD_TYPE STREQUAL "Debug" )
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${STD_CXX} -g -Wall -Wno-unused-variable -pthread")
else()
    set(CMAKE_BUILD_TYPE "Release")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${STD_CXX} -pthread -fno-strict-aliasing -Wno-unused-but-set-variable -Wno-deprecated")
endif( CMAKE_BUILD_TYPE STREQUAL "Debug" )
message("Build Type:" ${CMAKE_BUILD_TYPE} ${CMAKE_CXX_FLAGS})


# Instruct CMake to run moc automatically when needed.
set(CMAKE_AUTOMOC ON)
INCLUDE_DIRECTORIES(/opt/protobuf/include)
INCLUDE_DIRECTORIES(./src ./src/beast ./src/soci/include ./src/rocksdb2/include
    ./src/sqlite ./src/ed25519-donna ./src/snappy/snappy ./src/snappy/config
    ./src/websocketpp ./src/rocksdb2 ./build/proto ./src/secp256k1
    ./src/soci/src ./src/soci/include/private
    )

aux_source_directory(./src/ripple/app/main SRC_LIST)
aux_source_directory(./src/ripple/app/ledger SRC_LIST)
aux_source_directory(./src/ripple/app/ledger/impl SRC_LIST)
aux_source_directory(./src/ripple/app/misc/impl SRC_LIST)
aux_source_directory(./src/ripple/app/path/impl SRC_LIST)
aux_source_directory(./src/ripple/tx/ledger/impl SRC_LIST)
aux_source_directory(./src/ripple/core/impl SRC_LIST)
aux_source_directory(./src/ripple/basics/impl SRC_LIST)
aux_source_directory(./src/ripple/json/impl SRC_LIST)
aux_source_directory(./src/ripple/ledger/impl SRC_LIST)
aux_source_directory(./src/ripple/protocol/impl SRC_LIST)
aux_source_directory(./src/ripple/rpc/impl SRC_LIST)
aux_source_directory(./src/ripple/rpc/handlers SRC_LIST)
#aux_source_directory(./src/ripple/server/impl SRC_LIST)
aux_source_directory(./src/ripple/crypto/impl SRC_LIST)
aux_source_directory(./src/ripple/shamap/impl SRC_LIST)
#aux_source_directory(./src/ripple/resource/impl SRC_LIST)
aux_source_directory(./src/ripple/net/impl SRC_LIST)
#aux_source_directory(./src/ripple/websocket SRC_LIST)
aux_source_directory(./src/ripple/overlay/impl SRC_LIST)
aux_source_directory(./src/ripple/nodestore/impl SRC_LIST)
#aux_source_directory(./src/ripple/unity SRC_LIST)

add_library(${PROJECT_NAME}_object OBJECT ${SRC_LIST})

set(UNITY_LIST
    ./src/ripple/unity/soci.cpp
    ./src/ripple/unity/soci_ripple.cpp
    ./src/ripple/unity/app_ledger.cpp
    ./src/ripple/unity/app_main.cpp
    ./src/ripple/unity/app_misc.cpp
    ./src/ripple/unity/app_paths.cpp
    ./src/ripple/unity/app_tests.cpp
    ./src/ripple/unity/app_tx.cpp
    ./src/ripple/unity/core.cpp
    ./src/ripple/unity/basics.cpp
    ./src/ripple/unity/crypto.cpp
    ./src/ripple/unity/ledger.cpp
    ./src/ripple/unity/net.cpp
    ./src/ripple/unity/overlay.cpp
    ./src/ripple/unity/peerfinder.cpp
    ./src/ripple/unity/json.cpp
    ./src/ripple/unity/protocol.cpp
    ./src/ripple/unity/rpcx.cpp
    ./src/ripple/unity/shamap.cpp
    ./src/ripple/unity/test.cpp
    ./src/ripple/unity/unl.cpp
    ./src/ripple/unity/nodestore.cpp
    ./src/ripple/unity/git_id.cpp
    ./src/beast/beast/unity/hash_unity.cpp
    ./src/ripple/unity/beast.cpp
    ./src/ripple/unity/lz4.c
    ./src/ripple/unity/protobuf.cpp
    ./src/ripple/unity/ripple.proto.cpp
    ./src/ripple/unity/resource.cpp
    ./src/ripple/unity/server.cpp
    ./src/ripple/unity/websocket02.cpp
    ./src/ripple/unity/beastc.c
    ./src/ripple/unity/rocksdb.cpp
    ./src/ripple/unity/snappy.cpp
    ./src/ripple/unity/websocket04.cpp
    ./src/ripple/unity/secp256k1.cpp
    ./src/ripple/unity/ed25519.c
    )

add_executable(${PROJECT_NAME} ${UNITY_LIST})




#find boost lib
set(BOOST_ROOT "/opt/boost_1_60_0")
find_package(Boost REQUIRED COMPONENTS coroutine context filesystem system date_time regex program_options thread)

if(NOT Boost_FOUND)
  message(FATAL_ERROR "Not find Boost library")
endif()

include_directories(${Boost_INCLUDE_DIRS})
#find openssl lib
find_package(OpenSSL REQUIRED)
if(NOT OPENSSL_FOUND)
  message(FATAL_ERROR "Not find OpenSSL library")
endif()
include_directories(${OPENSSL_INCLUDE_DIR})

#find_library(secp256k1
#    NAMES secp256k1
#    HINTS "/opt/secp256k1/lib"
#)
#message("secp256k1" ${secp256k1})

find_library(protobuf
    NAMES protobuf
    HINTS "/opt/protobuf/lib"
)

#add_library(ed25519 ./src/ed25519-donna/ed25519.c)
set(RIPPLED_DIR ./src)
set(BEAST_SRC
    ${RIPPLED_DIR}/beast/beast/chrono/Chrono.unity.cpp
    ${RIPPLED_DIR}/beast/beast/threads/Threads.unity.cpp
    ${RIPPLED_DIR}/beast/beast/strings/Strings.unity.cpp
    ${RIPPLED_DIR}/beast/beast/utility/Utility.unity.cpp
    ${RIPPLED_DIR}/beast/beast/module/core/core.unity.cpp
    ${RIPPLED_DIR}/beast/beast/unity/hash_unity.cpp
    ${RIPPLED_DIR}/beast/beast/insight/Insight.unity.cpp
    ${RIPPLED_DIR}/beast/beast/http/HTTP.unity.cpp)

add_library(beast  ${BEAST_SRC})

if(UNIX)
    set(LINK_FLAGS "-lpthread -ldl -lgmp")
endif()

#find_library(soci_core
#    NAMES soci_core
#    HINTS "/home/jeffrey/QtC++/rippled/build/lib"
#)
#message("soci:" ${soci_core})
#find_library(pthread
#    NAMES libpthread.so)
#message("pthread:" ${pthread})

target_link_libraries(${PROJECT_NAME}  ${LINK_FLAGS}  ${Boost_LIBRARIES} ${OPENSSL_LIBRARIES} ${protobuf})

message("show info:" INCLUDE_DIRECTORIES ${COMPILE_DEFINITIONS} and ${COMPILE_OPTIONS} )
