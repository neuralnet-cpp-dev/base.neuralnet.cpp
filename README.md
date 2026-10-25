# base.neuralnet.cpp

neuralnet.cpp 生态的**核心基础库**（header-only，无上层依赖）。

| 头文件 | 内容 |
|---|---|
| `core_errors.hpp` | `Result<T>` / `NN_FAIL` / `NN_TRY` —— 无异常错误处理 |
| `core_assert.hpp` | `NN_ASSERT` / `NN_ASSERT_ALWAYS` 断言宏 |
| `precision.hpp` | 标量/精度类型（f16 边界 cast、Scalar 等） |
| `core_config.hpp` | 编译期常量（`Scalar`/`BLOCK_SIZE`/`EPSILON`/`PARALLEL_THRESHOLD`/`NN_VECTORIZE_PRAGMA`） |
| `core_file.hpp` | 文件读写工具 |
| `core_sha256.hpp` | SHA-256 |

## 用法

```cpp
#include <neuralnet.cpp/core_errors.hpp>
#include <neuralnet.cpp/precision.hpp>
```

CMake：

```cmake
add_subdirectory(base)   # 或 FetchContent / submodule
target_link_libraries(app PRIVATE neuralnet::base)
```

## 说明

所有 neuralnet.cpp 子库共享 `include/neuralnet.cpp/` 目录前缀——下游的
`#include <neuralnet.cpp/xxx.hpp>` 在各子库间透明解析，无需改 include 路径。
本库是该生态依赖 DAG 的最底层。
