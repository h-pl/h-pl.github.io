---
title: 'Vijeo Citect 7.2 遗留系统数据解耦与采集方案'
date: 2026-01-05 18:20:04
tags: []
published: true
hideInList: false
feature: 
isTop: false
---
施耐德电气消防系统数据读取技术方案，基于 Cicode 异步文件桥接（File Bridge）的技术实施白皮书

**文档版本**：V2.0 (Final)
**最后更新**：2026-01-05
**适用环境**：Schneider Vijeo Citect 7.2 / Windows Server
**关键词**：Cicode, Data Logging, JSON, NodeJS, Legacy System Integration

---

## 1. 项目背景与设计思路 (Background & Concept)

### 1.1 背景与现状 (The Context)

现场运行着一套基于 **Vijeo Citect 7.2** 的施耐德隧道消防监测系统。通过 citect explore 可知，系统底层通过 `variable.dbf` 定义了大量 Tag 变量，并归属于 **`Cluster1`** 集群。

* **当前痛点**：系统是一个封闭的 SCADA 环境。虽然界面（Panel 1 & Panel 2）能够显示 CO2、CH4 等气体的实时读数，但外部的 **Node.js 上报服务** 无法直接获取这些模拟量（Analog）数据，只能抓取文本报警日志。
* **业务需求**：以 **5秒/次** 的频率，获取界面上红框区域的 10 个气体读数及设备在线状态，并上报至云端。

### 1.2 核心设计思路 (Design Philosophy)

面对老旧工业软件，采用了 **“非侵入式文件桥接”** 方案。
我们不尝试破解 Citect 的内存或使用不稳定的 DCOM 接口，而是利用 Citect 自身的脚本引擎，**将“内存数据”周期性地固化为 “磁盘文件”**。

**设计逻辑链**：

1. **定位**：在 `variable.dbf` 中找到数据源头，定位面板展示 `tag` 含义。
2. **提取**：编写 Cicode 脚本从内存“捞取”数据。
3. **调度**：利用 Event（事件）机制进行周期性触发。
4. **执行**：**关键一步**，指定 IO Server 进程负责运行该调度。
5. **消费**：外部 Node.js 服务只读文件，不碰系统核心。

---

## 2. 软件架构解析 (Software Architecture)

本方案在逻辑上分为三层：**数据源层 (Citect)** -> **中转层 (File System)** -> **应用层 (Node.js)**。

![](https://h-pl.github.io/post-images/1767609165620.png)


* **Cluster1 的作用**：它是 Citect 系统的数据命名空间。我们在 `variable.dbf` 中确认了所有气体变量都属于 `Cluster1`，因此后续所有的 Event 和 Server 配置都必须挂载在 `Cluster1` 下，否则找不到变量。

---

## 3. 详细实施路线 (Implementation Roadmap)

### 3.1 变量映射 (Tag Mapping)

通过解析工程根目录下的 `variable.dbf`，确定了界面数值对应的底层变量名。
![](https://h-pl.github.io/post-images/1767609634270.png)
* **数据归属**：Cluster: `Cluster1` / Type: `REAL(浮点型)` /  ana: `analog(实时读数)`
* **变量清单**：

| 监测项 | FCP Panel 1 变量 | FCP Panel 2 变量 | 备注 |
| --- | --- | --- | --- |
| **CO2** | `GAS_1_ch1_ana` | `GAS_2_ch1_ana` | 模拟量 |
| **CH4** | `GAS_1_ch2_ana` | `GAS_2_ch2_ana` | 模拟量 |
| **CO** | `GAS_1_ch3_ana` | `GAS_2_ch3_ana` | 模拟量 |
| **H2S** | `GAS_1_ch4_ana` | `GAS_2_ch4_ana` | 模拟量 |
| **O2** | `GAS_1_ch5_ana` | `GAS_2_ch5_ana` | 模拟量 |
| **状态** | `IODeviceInfo` | `IODeviceInfo` | 在线检测 |

---

### 3.2 脚本开发 (Cicode Development)

**前置操作**：在 Windows C 盘根目录下手动新建文件夹 `C:\CitectData`。（Cicode 无法自动创建文件夹，若无此步，后续操作无效）。

**完整代码 (`ExportGasToText.ci`)**：

```c
/* * 函数名: ExportGasToText
 * 作用: 读取 Cluster1 中的气体变量，封装为 JSON 写入磁盘
 * 注意: 需要在 Computer Setup 中赋予 Server 运行权限
 */
FUNCTION ExportGasToText()
    INT hFile;
    STRING sPath;
    STRING sStatus1;
    STRING sStatus2;

    // 1. 指定输出路径 (文件夹必须预先存在)
    sPath = "C:\CitectData\GasRealtime.json";

    // 2. 检查设备在线状态 (3 = 状态模式, 返回 "1" 代表在线)
    IF IODeviceInfo("GAS_1", 3) = "1" THEN sStatus1 = "true"; ELSE sStatus1 = "false"; END
    IF IODeviceInfo("GAS_2", 3) = "1" THEN sStatus2 = "true"; ELSE sStatus2 = "false"; END

    // 3. 打开文件 ('w' = 覆盖模式，确保文件不无限增长)
    hFile = FileOpen(sPath, "w");

    IF hFile <> -1 THEN
        // 4. 构建 JSON 字符串
        FileWrite(hFile, "{");

        // --- Panel 1 ---
        FileWrite(hFile, "  ^"FCP1^": {");
        FileWrite(hFile, "    ^"Online^": " + sStatus1 + ",");
        FileWrite(hFile, "    ^"CO2^": " + RealToStr(GAS_1_ch1_ana, 6, 2) + ",");
        FileWrite(hFile, "    ^"CH4^": " + RealToStr(GAS_1_ch2_ana, 6, 2) + ",");
        FileWrite(hFile, "    ^"CO^": "  + RealToStr(GAS_1_ch3_ana, 6, 2) + ",");
        FileWrite(hFile, "    ^"H2S^": " + RealToStr(GAS_1_ch4_ana, 6, 2) + ",");
        FileWrite(hFile, "    ^"O2^": "  + RealToStr(GAS_1_ch5_ana, 6, 2)); 
        FileWrite(hFile, "  },");

        // --- Panel 2 ---
        FileWrite(hFile, "  ^"FCP2^": {");
        FileWrite(hFile, "    ^"Online^": " + sStatus2 + ",");
        FileWrite(hFile, "    ^"CO2^": " + RealToStr(GAS_2_ch1_ana, 6, 2) + ",");
        FileWrite(hFile, "    ^"CH4^": " + RealToStr(GAS_2_ch2_ana, 6, 2) + ",");
        FileWrite(hFile, "    ^"CO^": "  + RealToStr(GAS_2_ch3_ana, 6, 2) + ",");
        FileWrite(hFile, "    ^"H2S^": " + RealToStr(GAS_2_ch4_ana, 6, 2) + ",");
        FileWrite(hFile, "    ^"O2^": "  + RealToStr(GAS_2_ch5_ana, 6, 2));
        FileWrite(hFile, "  }");

        FileWrite(hFile, "}");
        FileClose(hFile);
    ELSE
        // 错误处理: 无法打开文件
        TraceMsg("Error: Cannot write to gas data file. Check Folder Path.");
    END
END

```

---

### 3.3 任务调度配置 (Events Configuration)

在 Project Editor -> System -> **Events** 中配置定时器。
![](https://h-pl.github.io/post-images/1767609654269.png)
* **Name**: `ExportGasToText`
* **Cluster Name**: `Cluster1` (此处必须填 `Cluster1`，因为变量都在此集群下。若留空，脚本找不到变量会报错)。
* **Time**: (留空)
* **Period**: `00:00:05` (5秒周期)
* **Action**: `ExportGasToText()`

---

### 3.4 进程执行分配 (Computer Setup) —— **[至关重要]**

**逻辑说明**：
配置了 Event 只是在数据库里写了一条“规则”，**必须指派具体的 Windows 进程（Server）去执行这条规则，否则 Event 永远不会启动。**

很多 Citect 开发者的脚本不生效，90% 都是死在这一步：默认情况下，新建的 Event **没有人认领**。

**操作步骤**：

1. 运行 **Computer Setup Wizard** -> Custom Setup。
2. 一路 Next 直到 **Events Setup** 页面。
3. **Client (客户端)**：
* ❌ **取消勾选** `ExportGasToText`。
* *原因*：客户端负责显示画面，不应负责后台写文件，避免多台客户端同时写入导致文件锁死。
![](https://h-pl.github.io/post-images/1767609667387.png)
4. **Cluster1.IOServer (IO服务器)**：
* ✅ **必须勾选** `ExportGasToText`。
* *原因*：IO Server 是系统的核心数据引擎，负责处理 Tag 数据，由它来执行写文件最高效、最稳定。


5. **User Privileges**：
* 选择 `Default Server User`。确保 IO Server 进程有权限在这个电脑上新建和写入文件。



---

## 4. 调试与验证 (Validation & Debugging)

完成上述配置后，**必须先编译（Compile）**，然后**完全重启 Runtime**（不仅是刷新，要关掉重开）。

### 4.1 验证步骤

1. **方法一：黑盒测试（观察结果）**
* 直接打开 `C:\CitectData\` 文件夹。
* 观察 `GasRealtime.json` 的 **“修改日期”**。
* **预期结果**：时间戳每 5 秒跳动一次。


2. **方法二：白盒测试（逻辑验证）**
* 如果文件未自动生成，在 Citect Graphic 画图编辑器中，打开 `Startup_Small` 页面。
* 画一个按钮，Input 设置为 `ExportGasToText()`。
* 运行并点击按钮。
* **结果分析**：
* 若点击按钮生成了文件，但自动运行不生成 -> 说明代码没问题，**步骤 3.4 配置有误**。
* 若点击按钮也不生成文件 -> 说明 **步骤 3.2 代码路径错误** 或文件夹不存在。

---

## 5. 总结 (Conclusion)

本方案通过标准化的 Citect 开发流程，解决了遗留系统数据封闭的问题。

* **安全性**：使用 `Cluster1.IOServer` 单进程写入，避免了文件冲突。
* **可靠性**：引入了 `IODeviceInfo` 在线判断，防止断线后上传脏数据。
* **扩展性**：基于 JSON 格式，Node.js 端解析灵活，未来增加变量只需修改 Cicode 即可。