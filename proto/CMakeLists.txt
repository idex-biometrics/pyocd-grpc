project(pyocd_grpc LANGUAGES CXX)
cmake_minimum_required(VERSION 3.15)

find_package(protobuf REQUIRED)
find_package(gRPC CONFIG REQUIRED)
find_package(Threads REQUIRED)

set(PROTO_FILES
    ${CMAKE_CURRENT_LIST_DIR}/pyocd_grpc/debugprobe.proto
)

set(PROTO_BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR})
set(PROTO_IMPORT_DIR ${CMAKE_CURRENT_LIST_DIR})

add_library(pyocd_grpc ${PROTO_FILES})
target_link_libraries(pyocd_grpc PUBLIC protobuf::libprotobuf gRPC::grpc gRPC::grpc++)
target_include_directories(pyocd_grpc PUBLIC "$<BUILD_INTERFACE:${PROTO_BINARY_DIR}>")

#
# Compile protobuf and grpc files in pyocd_grpc target to cpp
#
get_target_property(grpc_cpp_plugin_location gRPC::grpc_cpp_plugin LOCATION)

protobuf_generate(
    TARGET pyocd_grpc
    IMPORT_DIRS ${PROTO_IMPORT_DIR}
    PROTOC_OUT_DIR ${PROTO_BINARY_DIR}
    LANGUAGE cpp)

protobuf_generate(
    TARGET pyocd_grpc
    LANGUAGE grpc
    IMPORT_DIRS ${PROTO_IMPORT_DIR}
    PROTOC_OUT_DIR ${PROTO_BINARY_DIR}
    GENERATE_EXTENSIONS .grpc.pb.h .grpc.pb.cc
    PLUGIN "protoc-gen-grpc=${grpc_cpp_plugin_location}")