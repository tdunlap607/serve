
set(SPM_PROTOBUF_PROVIDER internal CACHE STRING "Use internal protobuf" FORCE)
set(SPM_USE_BUILTIN_PROTOBUF ON CACHE BOOL "Use builtin protobuf" FORCE)
set(SPM_ENABLE_SHARED OFF CACHE BOOL "Build shared library" FORCE)

FetchContent_Declare(
  sentencepiece
  GIT_REPOSITORY https://github.com/google/sentencepiece.git
  GIT_TAG v0.2.1
)
FetchContent_MakeAvailable(sentencepiece)

FetchContent_Declare(
  tokenizer_cpp
  GIT_REPOSITORY https://github.com/mlc-ai/tokenizers-cpp.git
  GIT_TAG 55d53aa38dc8df7d9c8bd9ed50907e82ae83ce66
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
