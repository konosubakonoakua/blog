---
number: 158
title: "[epics] epics ioc example notes"
state: open
url: https://github.com/konosubakonoakua/blog/issues/158
author: konosubakonoakua
createdAt: 2025-03-03T03:27:57Z
updatedAt: 2025-03-25T09:23:18Z
labels: []
---

# 158 [epics] epics ioc example notes

在 EPICS 中，`registrar(helloRegister)` 和 `function(myAsubProcess)` 是两种不同的机制，分别用于不同的目的。以下是它们的区别和各自的作用：

<details>

---

### 1. **`registrar(helloRegister)`**

#### **作用**
- **注册器函数**：`registrar(helloRegister)` 用于声明一个注册器函数 `helloRegister`，该函数在 IOC 启动时被自动调用。
- **功能**：通常用于注册自定义命令、设备支持或其他需要在 IOC 启动时初始化的功能。
- **使用场景**：例如，注册一个自定义命令到 `iocsh`，或者注册设备支持模块。

#### **示例**
```c
static void helloRegister(void) {
    iocshRegister(&helloFuncDef, helloCallFunc);
}
epicsExportRegistrar(helloRegister);
```
在 `.dbd` 文件中声明：
```c
registrar(helloRegister)
```

#### **特点**
- 在 IOC 启动时自动调用。
- 用于一次性初始化工作（如注册命令、设备支持等）。
- 与 IOC 的启动过程紧密相关。

---

### 2. **`function(myAsubProcess)`**

#### **作用**
- **函数注册**：`function(myAsubProcess)` 用于注册一个函数（如 `myAsubProcess`），使其可以在数据库记录中被调用。
- **功能**：通常用于定义记录的处理逻辑，例如 `aSubRecord` 的处理函数。
- **使用场景**：例如，为 `aSubRecord` 记录定义一个处理函数。

#### **示例**
```c
static long myAsubProcess(aSubRecord *precord) {
    // 处理逻辑
    return 0;
}

struct {
    long number;
    DEVSUPFUN report;
    DEVSUPFUN init;
    DEVSUPFUN init_record;
    DEVSUPFUN get_ioint_info;
    DEVSUPFUN process;
} devAsubMyDeviceSupport = {
    5, NULL, NULL, NULL, NULL, myAsubProcess
};

epicsExportAddress(dset, devAsubMyDeviceSupport);
```
在 `.dbd` 文件中声明：
```c
function(myAsubProcess)
```

#### **特点**
- 在数据库记录中通过 `field(PROC, "myAsubProcess")` 调用。
- 用于定义记录的处理逻辑。
- 与数据库记录的处理过程紧密相关。

---

### 3. **主要区别**

| **特性**                | **`registrar(helloRegister)`**                     | **`function(myAsubProcess)`**                  |
|-------------------------|---------------------------------------------------|-----------------------------------------------|
| **用途**                | 注册自定义命令、设备支持等初始化工作。             | 定义记录的处理逻辑。                          |
| **调用时机**            | IOC 启动时自动调用。                              | 在记录处理时调用（如 `aSubRecord` 的处理）。   |
| **关联对象**            | IOC 启动过程。                                    | 数据库记录。                                  |
| **示例**                | 注册一个自定义命令到 `iocsh`。                    | 为 `aSubRecord` 记录定义处理函数。            |
| **声明方式**            | 在 `.dbd` 文件中使用 `registrar(helloRegister)`。 | 在 `.dbd` 文件中使用 `function(myAsubProcess)`。 |

---

### 4. **总结**
- **`registrar(helloRegister)`**：
  - 用于在 IOC 启动时注册自定义命令、设备支持等初始化工作。
  - 与 IOC 的启动过程相关。
- **`function(myAsubProcess)`**：
  - 用于定义数据库记录的处理逻辑。
  - 与数据库记录的处理过程相关。

这两种机制在 EPICS 中分别用于不同的场景，`registrar` 用于初始化工作，而 `function` 用于记录处理逻辑。
</details>


---

## Comments

### konosubakonoakua · 2025-03-03T03:29:08Z

这是一个 EPICS 的 **替换文件**（Substitutions File），通常以 `.substitutions` 或 `.template` 为扩展名。它的作用是为数据库文件（`.db` 文件）中的变量提供具体的值，从而生成多个实例化的数据库记录。以下是对这段代码的详细解释：

<details>


---

### 1. **文件结构**
替换文件由多个 `file` 块组成，每个 `file` 块指定一个数据库文件，并为其变量提供具体的值。

---

### 2. **`file` 块解析**

#### **`file "db/circle.db"`**
```c
file "db/circle.db" {
    { user = "root" }
}
```
- **作用**：为数据库文件 `db/circle.db` 提供替换值。
- **替换内容**：
  - 将数据库文件中的 `$(user)` 替换为 `"root"`。
- **结果**：生成一个实例化的数据库文件，其中所有 `$(user)` 被替换为 `"root"`。

---

#### **`file "db/dbExample1.db"`**
```c
file "db/dbExample1.db" {
    { user = "root" }
}
```
- **作用**：为数据库文件 `db/dbExample1.db` 提供替换值。
- **替换内容**：
  - 将数据库文件中的 `$(user)` 替换为 `"root"`。
- **结果**：生成一个实例化的数据库文件，其中所有 `$(user)` 被替换为 `"root"`。

---

#### **`file db/dbExample2.db`**
```c
file db/dbExample2.db {
    pattern { user, no, scan }
        { "root", 1, "1 second" }
        { "root", 2, "2 second" }
        { "root", 3, "5 second" }
}
```
- **作用**：为数据库文件 `db/dbExample2.db` 提供多组替换值。
- **替换内容**：
  - 使用 `pattern` 关键字定义多组替换值。
  - 每组替换值对应数据库文件中的变量 `$(user)`、`$(no)` 和 `$(scan)`。
- **结果**：生成 **多个实例化的数据库文件**，每组替换值对应一个实例。例如：
  - 第一组：`$(user) = "root"`，`$(no) = 1`，`$(scan) = "1 second"`。
  - 第二组：`$(user) = "root"`，`$(no) = 2`，`$(scan) = "2 second"`。
  - 第三组：`$(user) = "root"`，`$(no) = 3`，`$(scan) = "5 second"`。

---

### 3. **数据库文件中的变量**
在数据库文件（如 `db/circle.db`、`db/dbExample1.db`、`db/dbExample2.db`）中，通常会使用 `$(variable)` 的形式定义变量。例如：
```c
record(ai, "$(user):temperature$(no)") {
    field(DESC, "Temperature Sensor $(no)")
    field(SCAN, "$(scan)")
}
```
替换文件会将这些变量替换为具体的值。

---

### 4. **生成的实例化数据库文件**
替换文件会生成多个实例化的数据库文件，每个文件中的变量被替换为具体的值。例如：
- 对于 `db/dbExample2.db`，生成的实例化文件可能如下：
  ```c
  record(ai, "root:temperature1") {
      field(DESC, "Temperature Sensor 1")
      field(SCAN, "1 second")
  }
  record(ai, "root:temperature2") {
      field(DESC, "Temperature Sensor 2")
      field(SCAN, "2 second")
  }
  record(ai, "root:temperature3") {
      field(DESC, "Temperature Sensor 3")
      field(SCAN, "5 second")
  }
  ```

---

### 5. **总结**
这段替换文件的作用是：
1. 为 `db/circle.db` 和 `db/dbExample1.db` 提供单一的替换值。
2. 为 `db/dbExample2.db` 提供多组替换值，生成多个实例化的数据库文件。
3. 通过替换文件，可以灵活地为数据库文件中的变量提供具体的值，从而生成多个实例化的记录。

这种机制在 EPICS 中非常有用，特别是在需要创建多个相似记录时（如多个传感器、设备等）。通过替换文件，可以避免重复编写相似的数据库文件，提高开发效率。
</details>


### konosubakonoakua · 2025-03-03T08:32:49Z

- https://docs.epics-controls.org/en/latest/getting-started/creating-ioc.html
- https://github.com/ISISComputingGroup/ibex_developers_manual/wiki/IOC-Start-Example
- https://epics.anl.gov/base/R3-15/5-docs/AppDevGuide/GettingStarted.html#x3-40002

### konosubakonoakua · 2025-03-10T03:42:41Z

iocCore envs
```
  EPICS_CA_ADDR_LIST
  EPICS_CA_AUTO_ADDR_LIST
  EPICS_CA_CONN_TMO
  EPICS_CAS_BEACON_PERIOD
  EPICS_CA_REPEATER_PORT
  EPICS_CA_SERVER_PORT
  EPICS_CA_MAX_ARRAY_BYTES
  EPICS_TS_NTP_INET
  EPICS_IOC_LOG_PORT
  EPICS_IOC_LOG_INET
```
