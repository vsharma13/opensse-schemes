add_library(
    schemes
    SHARED
    utils/logger.cpp
    utils/rocksdb_wrapper.cpp
    utils/utils.cpp
    utils/db_generator.cpp
    sophos/sophos_common.cpp
    sophos/sophos_client.cpp
    sophos/sophos_server.cpp
    diana/diana_common.cpp
    janus/janus_client.cpp
    janus/janus_server.cpp
)
# Add an alias
add_library(OpenSSE::schemes ALIAS schemes)

# Generate PIC for the library
set_target_properties(schemes PROPERTIES POSITION_INDEPENDENT_CODE ON)

target_compile_features(
    schemes
    PUBLIC
    cxx_auto_type
    cxx_nullptr
    cxx_constexpr
    cxx_defaulted_functions
    cxx_deleted_functions
    cxx_extern_templates
    cxx_noexcept
    cxx_static_assert
    cxx_thread_local
    PRIVATE
    cxx_lambdas
)

target_include_directories(
    schemes
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
    PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}
    PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include/sse/schemes
    PUBLIC ${ROCKSDB_INCLUDE_DIRS}
)

target_link_libraries(
    schemes
    OpenSSE::crypto
    OpenSSE::dbparser
    ${ROCKSDB_LIBRARIES}
    spdlog::spdlog
)

set(PROTOS
    ${CMAKE_CURRENT_SOURCE_DIR}/protos/sophos.proto
    ${CMAKE_CURRENT_SOURCE_DIR}/protos/diana.proto
)

set(PROTO_SRC_DIR ${CMAKE_CURRENT_BINARY_DIR})
# file(MAKE_DIRECTORY ${PROTO_SRC_DIR})

protobuf_generate_cpp(PROTO_SRCS PROTO_HDRS ${PROTOS})
grpc_generate_cpp(GRPC_SRCS GRPC_HDRS ${PROTO_SRC_DIR} ${PROTOS})

add_custom_target(grpc_gen DEPENDS ${PROTO_SRCS} ${PROTO_HDRS} ${GRPC_SRCS} ${GRPC_HDRS})
set_source_files_properties(${PROTO_SRCS} ${PROTO_HDRS} ${GRPC_SRCS} ${GRPC_HDRS} PROPERTIES COMPILE_FLAGS -Wno-unused-parameter)

add_library(
    runners
    SHARED
    sophos/sophos_client_runner.cpp
    sophos/sophos_server_runner.cpp
    diana/client_runner.cpp
    diana/server_runner.cpp
    ${PROTO_SRCS} 
    ${GRPC_SRCS} 
)

target_include_directories(runners SYSTEM PRIVATE ${PROTO_SRC_DIR})

# Add an alias
add_library(OpenSSE::runners ALIAS runners)

# Generate PIC for the library
set_target_properties(runners PROPERTIES POSITION_INDEPENDENT_CODE ON)

target_compile_features(
    runners
    PUBLIC
    cxx_auto_type
    cxx_nullptr
    cxx_constexpr
    cxx_defaulted_functions
    cxx_deleted_functions
    cxx_extern_templates
    cxx_noexcept
    cxx_static_assert
    cxx_thread_local
    PRIVATE
    cxx_lambdas
)

target_include_directories(
    runners
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
    PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}
    PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include/sse/runners
    PRIVATE ${ROCKSDB_INCLUDE_DIRS}
)

target_link_libraries(
    runners
    OpenSSE::crypto
    OpenSSE::dbparser
    OpenSSE::schemes
    gRPC::grpc++
    protobuf::libprotobuf
    spdlog::spdlog
)
