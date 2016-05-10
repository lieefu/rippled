project(rippled)
cmake_minimum_required(VERSION 3.3)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
if(UNIX)
    SET(CMAKE_FIND_LIBRARY_SUFFIXES .a .so)
else()
    SET(CMAKE_FIND_LIBRARY_SUFFIXES .lib .dll)
endif()
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
endif(CMAKE_BUILD_TYPE STREQUAL "Debug" )
message("Build Type:" ${CMAKE_BUILD_TYPE} ${CMAKE_CXX_FLAGS})

INCLUDE_DIRECTORIES(./src ./src/beast ./src/beast/include ./src/soci/include ./src/rocksdb2/include
    ./src/sqlite ./src/ed25519-donna ./src/snappy/snappy ./src/snappy/config
    ./src/websocketpp ./src/rocksdb2 ./src/secp256k1
    ./src/soci/src ./src/soci/include/private
    )

#execute protoc proto/ripple.proto --cpp_out= ,generate ripple.pb.cc and ripple.pb.h
execute_process(COMMAND protoc proto/ripple.proto --cpp_out=${CMAKE_CURRENT_BINARY_DIR}/
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/src/ripple/
    )
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_BINARY_DIR}/proto)

aux_source_directory(./src/ripple/app/main SRC_LIST)
aux_source_directory(./src/ripple/app/ledger SRC_LIST)
aux_source_directory(./src/ripple/app/ledger/impl SRC_LIST)
aux_source_directory(./src/ripple/app/misc SRC_LIST)
aux_source_directory(./src/ripple/app/misc/impl SRC_LIST)
aux_source_directory(./src/ripple/app/paths SRC_LIST)
aux_source_directory(./src/ripple/app/paths/impl SRC_LIST)
aux_source_directory(./src/ripple/app/paths/cursor SRC_LIST)
aux_source_directory(./src/ripple/app/tx/impl SRC_LIST)
aux_source_directory(./src/ripple/core/impl SRC_LIST)
aux_source_directory(./src/ripple/basics/impl SRC_LIST)
aux_source_directory(./src/ripple/json/impl SRC_LIST)
aux_source_directory(./src/ripple/ledger/detail SRC_LIST)
aux_source_directory(./src/ripple/ledger/impl SRC_LIST)
aux_source_directory(./src/ripple/protocol/impl SRC_LIST)
aux_source_directory(./src/ripple/rpc/impl SRC_LIST)
aux_source_directory(./src/ripple/rpc/handlers SRC_LIST)
aux_source_directory(./src/ripple/server/impl SRC_LIST)
aux_source_directory(./src/ripple/crypto/impl SRC_LIST)
aux_source_directory(./src/ripple/shamap/impl SRC_LIST)
aux_source_directory(./src/ripple/net/impl SRC_LIST)
aux_source_directory(./src/ripple/overlay/impl SRC_LIST)
aux_source_directory(./src/ripple/peerfinder/impl SRC_LIST)
aux_source_directory(./src/ripple/unl/tests SRC_LIST)

set(UNITY_COMMON
    ./src/sqlite/sqlite_unity.c
    ./src/ripple/unity/soci.cpp
    ./src/ripple/unity/nodestore.cpp
    ./src/ripple/unity/git_id.cpp
    ./src/ripple/beast/unity/beast_clock_unity.cpp
    ./src/ripple/beast/unity/beast_container_unity.cpp
    ./src/ripple/beast/unity/beast_insight_unity.cpp
    ./src/ripple/beast/unity/beast_net_unity.cpp
    ./src/ripple/beast/unity/beast_nudb_unity.cpp
    ./src/ripple/beast/unity/beast_utility_unity.cpp
    ./src/ripple/beast/unity/beast_hash_unity.cpp
    ./src/ripple/unity/beast.cpp
    ./src/ripple/unity/lz4.c
    ./src/ripple/unity/protobuf.cpp
    ./src/ripple/unity/ripple.proto.cpp
    ./src/ripple/unity/resource.cpp
    ./src/ripple/unity/websocket02.cpp
    ./src/ripple/unity/rocksdb.cpp
    ./src/ripple/unity/snappy.cpp
    ./src/ripple/unity/secp256k1.cpp
    ./src/ripple/unity/ed25519.c
    )
add_executable(${PROJECT_NAME} ${SRC_LIST} ${UNITY_COMMON})
#add_library(${PROJECT_NAME}_object OBJECT ${SRC_LIST})

set(UNITY_LIST
    ./src/ripple/beast/unity/beast_clock_unity.cpp
    ./src/ripple/beast/unity/beast_container_unity.cpp
    ./src/ripple/beast/unity/beast_insight_unity.cpp
    ./src/ripple/beast/unity/beast_net_unity.cpp
    ./src/ripple/beast/unity/beast_nudb_unity.cpp
    ./src/ripple/beast/unity/beast_utility_unity.cpp
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
    ./src/ripple/unity/server.cpp
    ./src/ripple/unity/test.cpp
    )

#add_executable(${PROJECT_NAME} ${UNITY_LIST} ${UNITY_COMMON})

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

#find_library(PROTOBUF_LIBRARIES NAMES protobuf)
find_package(Protobuf REQUIRED)
if(NOT PROTOBUF_FOUND)
  message(FATAL_ERROR "Not find Protobuf library")
endif()

if(UNIX)
    set(LINK_FLAGS "-lpthread -ldl -lgmp")
endif()

target_link_libraries(${PROJECT_NAME}  ${LINK_FLAGS}  ${Boost_LIBRARIES} ${OPENSSL_LIBRARIES} ${PROTOBUF_LIBRARIES})
