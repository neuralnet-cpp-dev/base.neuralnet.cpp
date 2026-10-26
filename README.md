# base.neuralnet.cpp

neuralnet.cpp 生态的**核心基础库**（header-only，无上层依赖——仅标准库）。
依赖 DAG 的最底层：`threadpool` / `kvrecord` / `dataset` / `tokenizer` 都构建在它之上。

## 内容

| 头文件 | 内容 |
|---|---|
| `core_errors.hpp` | `Result<T>`（= `std::expected`）/ `Error` / `NN_FAIL` / `NN_TRY` 族 —— 无异常错误处理 |
| `core_assert.hpp` | `NN_ASSERT`（可选诊断）/ `NN_ASSERT_ALWAYS` / `NN_FATAL`（Release 也终止）/ `NN_REQUIRE`（返回 Error） |
| `precision.hpp` | `Precision` 枚举 / `nn::f16`（round-half-to-even）/ f16↔f32 位转换 / `PrecisionProfile` 配方 |
| `core_config.hpp` | 编译期常量（`Scalar` / `BLOCK_SIZE` / `EPSILON` / `PARALLEL_THRESHOLD` / `NN_VECTORIZE_PRAGMA`） |
| `core_file.hpp` | 文本读取（`load_text_file`）/ 流式 SHA-256（`sha256_file_prefix` / `sha256_file_hex`）/ POD 二进制 I/O（`write_pod` / `read_pod` / `write_pod_span` / `read_pod_span`） |
| `core_sha256.hpp` | SHA-256（FIPS 180-4）：增量（`Sha256State` + `sha256_update` + `sha256_finalize`）与一次性（`sha256_digest` / `sha256_hex`） |

## API 速览

### 错误处理（`core_errors.hpp`）

```cpp
nn::Result<int> parse(std::string_view s);          // 成功 = 值，失败 = nn::Error

// 传播（失败即从当前函数返回错误）：
nn::Result<int> f() {
    NN_TRY(v, parse("42"));                         // 解值 + 原样传播
    NN_TRY_MSG(v, "解析配置", parse(x));             // 错误前缀加语境
    NN_TRY_CHECK(maybe_fail());                     // Result<void>/bool，只求值一次
    return *v;
}

// 消费（失败即终止进程，打印 [nn] 语境: 详情 (表达式) file:line）：
int v = NN_UNWRAP(parse("42"), "读取配置");
NN_UNWRAP(ok_condition, "前置条件");
NN_EXIT(parse(x), 3, "配置损坏");                   // 失败以退出码 3 终止

// 错误源头：
NN_FAIL("不支持的格式");                            // return std::unexpected(Error{...})
```

### 断言（`core_assert.hpp`）

| 宏 | NDEBUG（Release） | 用途 |
|---|---|---|
| `NN_ASSERT(cond, msg)` | **展开为空**（不求值 cond） | 可选诊断：失败也不影响后续正确性 |
| `NN_ASSERT_ALWAYS(cond, msg)` | **仍然终止**（打印位置 + 消息） | 前置条件：断言后继续用该条件 = UB |
| `NN_FATAL(msg)` | 仍然终止 | 无条件编程错误 |
| `NN_REQUIRE(cond, msg)` | 始终生效 | 返回 `std::unexpected(Error)`（不终止） |

### 精度类型（`precision.hpp`）

```cpp
nn::f16 h{0.1f};                    // round-half-to-even 舍入
float f = static_cast<float>(h);    // 精确升位
auto bits = h.bits();               // std::uint16_t 位型

nn::check_precision_supported(nn::Precision::F16);   // Result<void>：BF16/F64/T1_58 被拒
nn::precision_rank(nn::Precision::F16);              // F16(0) < F32(1)；无序精度 = -1
```

## 用法

```cpp
#include <neuralnet.cpp/core_errors.hpp>
#include <neuralnet.cpp/core_assert.hpp>
#include <neuralnet.cpp/precision.hpp>
```

CMake（三选一）：

```cmake
# 1) submodule + add_subdirectory
add_subdirectory(base)
# 2) FetchContent
# 3) 工程内已有 neuralnet_base 目标时直接用（各子库的 CMake 都做了目标存在性检测）
target_link_libraries(app PRIVATE neuralnet::base)   # 提供 include 路径 + C++23
```

## 构建与测试

本库是 header-only，**无需编译**即可作为库消费。测试独立可跑：

```bash
git submodule update --init --recursive   # 本仓无嵌套 submodule，此步可省
cmake -B build
cmake --build build
ctest --test-dir build --output-on-failure
```

| 测试 | 覆盖 |
|---|---|
| `base_error_macro_test` | `NN_UNWRAP`/`NN_EXIT` 解值三形态 + 打印格式 + `NN_TRY_CHECK` 单次求值回归 + 终止行为（子进程断言） |
| `base_precision_type_test` | f16 全 65536 位型往返 / RHE 舍入（密集采样 + 中点）/ 边界值 / 算术语义 / BF16/F64/T1_58 拒绝路径 |
| `base_sha256_test` | FIPS 180-4 标准向量 / 增量与一次性对拍 / hex 格式 / 雪崩性 |
| `base_file_test` | `load_text_file` / `sha256_file_prefix`（前缀摘要）/ POD I/O 往返 |
| `base_assert_test` | 断言宏两态语义（失败路径经子进程断言，NDEBUG 无关）/ `NN_REQUIRE` |

被父工程 `add_subdirectory` 消费时测试**默认不构建**（`NN_BUILD_TESTS` 默认值 =
是否顶层工程）；显式 `-DNN_BUILD_TESTS=ON` 可在嵌入时强制打开。

## 说明

所有 neuralnet.cpp 子库共享 `include/neuralnet.cpp/` 目录前缀——下游的
`#include <neuralnet.cpp/xxx.hpp>` 在各子库间透明解析，无需改 include 路径。

编译要求：C++23（`std::expected`）。Windows/MSVC 需带 `/Zc:preprocessor`
（`NN_UNWRAP` 等可变参数宏的实计数分发依赖标准预处理器行为）。

许可证：LGPL-3.0（见 [LICENSE](LICENSE)）。
