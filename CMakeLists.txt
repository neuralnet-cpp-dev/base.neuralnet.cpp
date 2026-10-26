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

# ── 测试（NN_BUILD_TESTS；顶层工程默认 ON，被 add_subdirectory 消费时默认 OFF）──
# 独立开发/CI：cmake -B build && cmake --build build && ctest --test-dir build
option(NN_BUILD_TESTS "Build base.neuralnet.cpp tests" ${PROJECT_IS_TOP_LEVEL})
if(NN_BUILD_TESTS)
    enable_testing()
    # 警告口径与门面仓一致：严格警告 + 零异常（测试代码同样守规）
    if(MSVC)
        add_compile_options(/W4 /permissive- /Zc:preprocessor /utf-8)
    else()
        add_compile_options(-fno-exceptions -Wall -Wextra -Wpedantic -Werror)
    endif()

    foreach(_t error_macro precision_type sha256 file assert)
        add_executable(base_${_t}_test tests/${_t}_test.cpp)
        target_link_libraries(base_${_t}_test PRIVATE neuralnet::base)
        add_test(NAME base_${_t}_test COMMAND base_${_t}_test)
    endforeach()
endif()
