# 禁用功能清单

## 🎯 目标
只保留WiFi转发核心功能，禁用所有多余组件以：
- 节省Flash空间
- 节省RAM空间
- 提高系统稳定性
- 加快启动速度

---

## ❌ 已禁用的功能

### 1. 蓝牙 (BT/BLE) ✅
```ini
CONFIG_BT_ENABLED=n
```

**节省资源**：
- Flash: ~400KB
- RAM: ~80KB
- 栈空间: ~8KB (BT任务)

**禁用组件**：
- ✅ Bluetooth Controller
- ✅ Bluedroid协议栈
- ✅ BLE功能
- ✅ HCI over SDIO/SPI/UART

---

### 2. CLI命令行接口 ✅
```ini
CONFIG_ESP_HOSTED_CLI_ENABLED=n
```

**节省资源**：
- Flash: ~50KB (命令解析代码)
- RAM: ~5KB (命令缓冲区)

**禁用功能**：
- ✅ 串口命令行
- ✅ WiFi命令（wifi_cmd组件）
- ✅ Ping命令
- ✅ 系统命令

---

### 3. MQTT示例 ✅
```ini
CONFIG_ESP_HOSTED_COPROCESSOR_EXAMPLE_MQTT=n
```

**节省资源**：
- Flash: ~30KB
- RAM: ~10KB

**说明**：
- 仅示例代码被禁用
- 如需在P4端使用MQTT，不受影响

---

### 4. HTTP Client示例 ✅
```ini
CONFIG_ESP_HOSTED_COPROCESSOR_EXAMPLE_HTTP_CLIENT=n
```

**节省资源**：
- Flash: ~20KB
- RAM: ~5KB

**说明**：
- 仅示例代码被禁用
- P4端可正常使用HTTP

---

### 5. GDB调试 ✅
```ini
CONFIG_ESP_GDBSTUB_ENABLED=n
```

**节省资源**：
- Flash: ~30KB
- RAM: ~8KB

**说明**：
- 生产环境不需要GDB调试
- 开发调试时可临时启用

---

### 6. Core Dump ✅
```ini
CONFIG_ESP_COREDUMP_ENABLE_TO_NONE=y
```

**节省资源**：
- Flash: ~50KB (不保存coredump分区)
- RAM: ~5KB

**说明**：
- 崩溃时不保存内存转储
- 稳定后可禁用节省空间

---

### 7. WiFi企业级支持 ✅
```ini
CONFIG_ESP_WIFI_ENTERPRISE_SUPPORT=n
```

**节省资源**：
- Flash: ~80KB
- RAM: ~15KB

**禁用功能**：
- ✅ WPA2-Enterprise
- ✅ 802.1X认证
- ✅ EAP-TLS/PEAP等

**说明**：
- 家庭/办公WiFi不需要
- 只使用WPA2-PSK

---

### 8. 中断看门狗CPU1检查 ✅
```ini
CONFIG_ESP_INT_WDT_CHECK_CPU1=n
```

**说明**：
- ESP32-C6是单核，无CPU1
- 禁用无效配置

---

### 9. Newlib标准格式化 → Nano ✅
```ini
CONFIG_NEWLIB_NANO_FORMAT=y
```

**节省资源**：
- Flash: ~40KB (printf/scanf精简版)

**说明**：
- nano版本功能够用
- 不支持浮点格式化（如果需要可改回）

---

## ✅ 保留的核心功能

### WiFi功能
- ✅ STA模式连接AP
- ✅ AMPDU聚合
- ✅ WPA2-PSK加密
- ✅ DHCP客户端

### SDIO传输
- ✅ SDIO Slave接口
- ✅ 数据收发
- ✅ 流控机制

### 网络转发
- ✅ WiFi ↔ SDIO桥接
- ✅ 零拷贝转发
- ✅ 统计功能（可选）

### 基础系统
- ✅ FreeRTOS
- ✅ LWIP协议栈
- ✅ NVS存储
- ✅ OTA升级

---

## 📊 总计节省资源

| 资源 | 节省量 | 说明 |
|------|--------|------|
| **Flash** | ~700KB | 约17%的4MB Flash |
| **RAM** | ~130KB | 约25%的512KB RAM |
| **任务栈** | ~8KB | 减少后台任务 |
| **启动时间** | ~500ms | 减少初始化组件 |

---

## 🔧 可选功能开关

### 如需调试，可临时启用：

#### 1. 统计功能
```ini
CONFIG_ESP_PKT_STATS=y
```
查看丢包率、重试率等统计

#### 2. 函数性能分析
```ini
CONFIG_ESP_HOSTED_FUNCTION_PROFILING=y
```
分析各函数执行时间

#### 3. GDB调试
```ini
CONFIG_ESP_GDBSTUB_ENABLED=y
```
通过串口使用GDB调试

#### 4. Task Watchdog
```ini
CONFIG_ESP_TASK_WDT_EN=y
```
监控任务是否卡死

---

## ⚠️ 禁用前确认

以下功能**已确认不需要**：

- ❌ 蓝牙功能 - P4端不需要BT
- ❌ CLI命令行 - P4端通过RPC控制
- ❌ MQTT/HTTP示例 - 仅示例代码
- ❌ WiFi企业级 - 使用WPA2-PSK即可
- ❌ Core Dump - 稳定后不需要

---

## 📝 配置文件

所有禁用配置在以下文件中：
- `sdkconfig.defaults` - 通用配置
- `sdkconfig.defaults.esp32c6` - C6专用配置

---

## 🚀 编译验证

```bash
# 删除旧配置
rm -f sdkconfig
rm -rf build/

# 重新配置
idf.py reconfigure

# 编译
idf.py build

# 查看Flash占用
idf.py size-components
```

### 预期Flash占用

| 组件 | 大小 | 说明 |
|------|------|------|
| app.bin | ~1.5MB | 应用程序 |
| bootloader.bin | ~30KB | 引导加载 |
| partition_table | ~3KB | 分区表 |
| **总计** | ~1.6MB | 剩余2.4MB可用 |

---

## 💡 进一步优化建议

### 如还需要更多空间：

#### 1. 禁用日志颜色
```ini
CONFIG_LOG_COLORS=n
```
节省 ~5KB Flash

#### 2. 降低日志级别
```ini
CONFIG_LOG_DEFAULT_LEVEL_WARN=y
```
减少日志输出

#### 3. 禁用断言
```ini
CONFIG_COMPILER_OPTIMIZATION_ASSERTIONS_DISABLE=y
```
节省 ~10KB Flash（不推荐）

#### 4. 减小LWIP缓冲区
```ini
CONFIG_LWIP_TCP_SND_BUF_DEFAULT=8192
CONFIG_LWIP_TCP_WND_DEFAULT=8192
```
节省 ~6KB RAM（会影响吞吐量）

---

## ✅ 总结

当前配置已优化为：
- **最小资源占用** - 禁用所有非必需组件
- **最佳性能** - WiFi buffer和AMPDU优化
- **专注转发** - 只保留WiFi↔SDIO桥接功能

适用场景：
- ✅ WiFi网卡模式
- ✅ 网络桥接
- ✅ 数据转发

**配置完成！可以编译测试了！** 🎉
