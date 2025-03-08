
# 异常体系

```
Throwable
├── Error
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── ...
└── Exception
    ├── RuntimeException
    │   ├── NullPointerException
    │   ├── IndexOutOfBoundsException
    │   └── ...
    └── 检查型异常（Checked Exceptions）
        ├── IOException
        ├── SQLException
        └── ...
```

# Error

系统级严重问题（如内存耗尽、虚拟机崩溃），程序无法恢复。
- `OutOfMemoryError`：内存不足。
- `StackOverflowError`：栈溢出。

处理方式：通常不捕获，应修复代码或调整环境；

# Exception

程序可处理的异常情况
- **检查型异常（Checked Exceptions）**：
    - 继承自`Exception`但不属于`RuntimeException`。
    - **必须显式处理**（`try-catch`或`throws`），否则编译失败。
    - **示例**：`IOException`、`ClassNotFoundException`。
        
- **非检查型异常（Unchecked Exceptions）**：
    - 继承自`RuntimeException`或`Error`。
    - 无需显式处理，通常由代码逻辑错误导致。
    - **示例**：`NullPointerException`、`ArrayIndexOutOfBoundsException`。

|**特性**|**检查型异常**|**非检查型异常**|
|---|---|---|
|**继承关系**|继承`Exception`但不包括`RuntimeException`|继承`RuntimeException`或`Error`|
|**处理要求**|必须显式处理（`try-catch`或`throws`）|无需显式处理|
|**典型场景**|外部因素（如文件不存在、网络中断）|程序逻辑错误（如空指针、数组越界）|

# 异常处理机制

1. try...catch...：包裹代码处理异常；
2. throw：显式抛出异常；

# 异常使用实践：

1. **精准捕获**：避免捕获宽泛的`Exception`，应针对具体异常类型；
2. **自定义异常**：继承`Exception`或`RuntimeException`，提供有意义的信息；
3. **资源管理**：优先使用`try-with-resources`，避免资源泄漏；
4. **避免异常滥用**：不要用异常控制流程（如代替条件判断）；

