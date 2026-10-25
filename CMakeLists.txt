# base.neuralnet.cpp — neuralnet.cpp 核心基础库（header-only）
#
# 内容：Result<T> 错误处理 / 断言 / 精度类型 / 编译期配置常量 / 文件 I/O / SHA-256。
# 这是 neuralnet.cpp 生态的最底层，无任何上层依赖（仅标准库）。
#
# 消费方式：
#   add_subdirectory(base)            # 或 FetchContent / submodule
#   target_link_libraries(app PRIVATE neuralnet::base)
#   #include <neuralnet.cpp/core_errors.hpp> 等
cmake_minimum_required(VERSION 3.20)
project(neuralnet_base LANGUAGES CXX)

add_library(neuralnet_base INTERFACE)
add_library(neuralnet::base ALIAS neuralnet_base)

# 头文件根：下游 #include <neuralnet.cpp/xxx.hpp> 由此解析
target_include_directories(neuralnet_base INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>)

# Result<T> = std::expected，需 C++23
target_compile_features(neuralnet_base INTERFACE cxx_std_23)
