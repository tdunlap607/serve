# Set sentencepiece options before fetching tokenizer_cpp
# tokenizer_cpp will fetch sentencepiece as a dependency
# Configure sentencepiece to use external protobuf from FetchContent
set(SPM_USE_BUILTIN_PROTOBUF OFF CACHE BOOL "Use builtin protobuf" FORCE)
set(SPM_PROTOBUF_PROVIDER "package" CACHE STRING "Use find_package for protobuf" FORCE)
set(SPM_ENABLE_SHARED OFF CACHE BOOL "Build shared library" FORCE)
set(SPM_ENABLE_EXECUTABLE OFF CACHE BOOL "Build sentencepiece executables" FORCE)

# Find system protoc compiler (needed for code generation)
find_program(PROTOC_EXECUTABLE protoc REQUIRED)

# Make the fetched protobuf discoverable by find_package(Protobuf)
# Create the necessary variables and targets that find_package expects
set(Protobuf_FOUND TRUE)
set(Protobuf_VERSION "32.1")
set(Protobuf_INCLUDE_DIRS "${protobuf_SOURCE_DIR}/src")
set(Protobuf_LIBRARIES libprotobuf-lite)
set(Protobuf_PROTOC_EXECUTABLE ${PROTOC_EXECUTABLE})
set(Protobuf_LITE_LIBRARIES libprotobuf-lite)

# Ensure the protobuf include directory is available
include_directories(${protobuf_SOURCE_DIR}/src)

FetchContent_Declare(
  tokenizer_cpp
  GIT_REPOSITORY https://github.com/tdunlap607/tokenizers-cpp
  GIT_TAG 4ef19e9f947f4a96c57b038e24ac1556fcd39725
)

FetchContent_MakeAvailable(tokenizer_cpp)

add_custom_command(
    OUTPUT bert-seq.so
    COMMAND TOKENIZERS_PARALLELISM=false python ${CMAKE_CURRENT_SOURCE_DIR}/aot_compile_export.py
    COMMAND cp ${CMAKE_CURRENT_BINARY_DIR}/Transformer_model/tokenizer.json ${CMAKE_CURRENT_BINARY_DIR}/
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/aot_compile_export.py
)

add_library(bert_handler SHARED src/bert_handler.cc bert-seq.so)
target_include_directories(bert_handler PRIVATE)
target_link_libraries(bert_handler PRIVATE ts_backends_core ts_utils ${TORCH_LIBRARIES} tokenizers_cpp)
