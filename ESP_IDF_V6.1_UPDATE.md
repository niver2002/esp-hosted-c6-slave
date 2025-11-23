# ESP32-C6 固件配置更新 - ESP-IDF v6.1

## 更新日期
2025-11-22

## 更新目标
根据 ESP-IDF v6.1 的最新最佳实践，更新 ESP32-C6 slave 固件的配置，提升性能、稳定性和兼容性。

---

## 📝 更新内容

### 1. 芯片版本支持明确化

**文件**：`sdkconfig.defaults.esp32c6`

**新增配置**：
```
# 2. Chip Revision Support
CONFIG_ESP32C6_REV_MIN_0=y
```

**说明**：
- ESP32-C6 支持所有版本（v0.0 到 v0.99）
- 明确配置最小支持版本，与 P4 配置保持一致性

---

### 2. 日志系统升级到 Log V2

**文件**：`sdkconfig.defaults` 和 `sdkconfig.defaults.esp32c6`

**新增配置**：
```
# Log Configuration (ESP-IDF v6.1)
CONFIG_LOG_VERSION_2=y
CONFIG_LOG_DEFAULT_LEVEL_INFO=y
CONFIG_LOG_COLORS=y
```

**优势**：
- ✅ 更好的实时性能
- ✅ 自动检测约束环境
- ✅ 减少日志开销
- ✅ 保持彩色输出，便于调试

---

### 3. 睡眠和电源管理优化

**文件**：`sdkconfig.defaults.esp32c6`

**新增配置**：
```
# 12. Sleep and Power Management
CONFIG_ESP_SLEEP_FLASH_LEAKAGE_WORKAROUND=y
CONFIG_ESP_SLEEP_GPIO_RESET_WORKAROUND=y
```

**说明**：
- Flash 漏电修复 - 改善睡眠模式功耗
- GPIO 复位修复 - 确保正确的 GPIO 行为

---

### 4. 内存优化

**文件**：`sdkconfig.defaults.esp32c6`

**新增配置**：
```
# 13. Memory Configuration
CONFIG_LIBC_NEWLIB_NANO_FORMAT=y
```

**效果**：
- ✅ 节省 RAM 使用
- ✅ 节省 Flash 空间
- ✅ 适合嵌入式环境

---

### 5. 硬件加密加速

**文件**：`sdkconfig.defaults.esp32c6`

**新增配置**：
```
# 14. Hardware Acceleration
CONFIG_MBEDTLS_HARDWARE_AES=y
CONFIG_MBEDTLS_HARDWARE_SHA=y
```

**效果**：
- ✅ 利用硬件 AES/SHA 加速
- ✅ 减少 CPU 负载
- ✅ 提升加密性能

---

### 6. 任务栈大小优化

**文件**：`sdkconfig.defaults.esp32c6`

**新增配置**：
```
# 15. Task Stack Sizes
CONFIG_ESP_MAIN_TASK_STACK_SIZE=4096
CONFIG_ESP_SYSTEM_EVENT_TASK_STACK_SIZE=3072
```

**说明**：
- 与 P4 配置保持一致
- 平衡内存使用和功能需求

---

### 7. 编译器优化增强

**文件**：`sdkconfig.defaults` 和 `sdkconfig.defaults.esp32c6`

**新增配置**：
```
CONFIG_COMPILER_OPTIMIZATION_PERF=y
CONFIG_COMPILER_OPTIMIZATION_ASSERTIONS_ENABLE=y
```

**说明**：
- 性能优化（-O2）
- 保持断言以便调试

---

### 8. Bootloader 日志配置

**文件**：`sdkconfig.defaults`

**新增配置**：
```
# Use Log V2 for bootloader (ESP-IDF v6.1)
CONFIG_BOOTLOADER_LOG_LEVEL_INFO=y
```

**说明**：
- Bootloader 也使用 Log V2
- 保持 INFO 级别以便启动诊断

---

## 📊 配置文件结构优化

### 之前
```
sdkconfig.defaults           - 简单的基础配置
sdkconfig.defaults.esp32c6   - C6 特定配置
```

### 现在
```
sdkconfig.defaults           - 通用最佳实践配置（所有芯片）
sdkconfig.defaults.esp32c6   - C6 特定配置 + ESP-IDF v6.1 优化
```

**改进**：
- ✅ 更清晰的组织结构
- ✅ 详细的注释说明
- ✅ 分节管理，易于维护
- ✅ 与 P4 配置风格一致

---

## 🔄 与已有优化的兼容性

### OPTIMIZATION_CHANGES.md 中的优化
**保持不变**：
- ✅ `MAX_WIFI_STA_TX_RETRY = 0`（代码修改）
- ✅ WiFi buffer 增大（48）
- ✅ SDIO 队列优化（20）
- ✅ SDIO timing 优化
- ✅ 蓝牙禁用

**新增配置是补充**，不会与已有优化冲突。

---

## 🚀 重新编译步骤

### 1. 清理旧构建（推荐）

```bash
cd f:/sta2eth/esp-hosted-mcu-slave/slave
idf.py fullclean
```

### 2. 重新配置

```bash
idf.py set-target esp32c6
```

### 3. （可选）检查配置

```bash
idf.py menuconfig
```

验证以下配置：
- Log V2 已启用
- 硬件加速已启用
- 睡眠 workaround 已启用

### 4. 编译

```bash
idf.py build
```

### 5. 验证构建结果

检查关键配置：

```bash
# 1. Log V2
grep "CONFIG_LOG_VERSION_2" build/config/sdkconfig.h
# 应显示：#define CONFIG_LOG_VERSION_2 1

# 2. 硬件加速
grep "CONFIG_MBEDTLS_HARDWARE" build/config/sdkconfig.h
# 应显示：#define CONFIG_MBEDTLS_HARDWARE_AES 1
#        #define CONFIG_MBEDTLS_HARDWARE_SHA 1

# 3. 芯片版本
grep "CONFIG_ESP32C6_REV_MIN" build/config/sdkconfig.h
# 应显示：#define CONFIG_ESP32C6_REV_MIN_0 1
```

---

## ✅ 更新验证清单

编译完成后，验证以下内容：

- [ ] **编译成功**，无错误
- [ ] **Log V2 已启用**
- [ ] **硬件加速已启用**（AES + SHA）
- [ ] **睡眠 workaround 已启用**
- [ ] **内存优化已应用**（Newlib nano）
- [ ] **芯片版本支持正确**（v0.0+）
- [ ] **已有优化保持不变**（WiFi buffer, SDIO 等）

---

## 🎯 预期效果

### 性能提升
- ✅ **更好的日志性能**（Log V2）
- ✅ **更快的加密操作**（硬件加速）
- ✅ **更低的功耗**（睡眠优化）

### 资源节省
- ✅ **节省 RAM**（Newlib nano）
- ✅ **节省 Flash**（Newlib nano）
- ✅ **节省 CPU**（硬件加速）

### 兼容性
- ✅ **支持所有 C6 版本**
- ✅ **与 ESP-IDF v6.1 完全兼容**
- ✅ **与 P4 配置风格一致**

---

## 📋 配置对比表

| 配置项 | 旧配置 | 新配置 | 说明 |
|--------|--------|--------|------|
| Log 版本 | 默认（v1） | V2 | 更好的性能 |
| 硬件 AES | 未指定 | 启用 | 利用硬件加速 |
| 硬件 SHA | 未指定 | 启用 | 利用硬件加速 |
| Newlib | 标准 | Nano | 节省内存 |
| 睡眠 workaround | 未指定 | 启用 | 改善功耗 |
| 芯片版本 | 默认 | 明确（v0.0+） | 与 P4 一致 |
| 断言 | 默认 | 启用 | 便于调试 |

---

## 📚 参考文档

1. ESP-IDF v6.1 Release Notes
2. ESP-IDF Logging System V2 Documentation
3. ESP32-C6 Technical Reference Manual
4. ESP-IDF Hardware Acceleration Guide
5. ESP-IDF Power Management Documentation

---

## 🔗 相关文件

- `sdkconfig.defaults` - 通用配置
- `sdkconfig.defaults.esp32c6` - C6 特定配置
- `OPTIMIZATION_CHANGES.md` - WiFi 转发性能优化
- `README.md` - 项目概览

---

## 📌 重要提示

1. **必须 fullclean**：配置更改需要完全重新构建
2. **保持已有优化**：新配置不会影响 OPTIMIZATION_CHANGES.md 中的优化
3. **与 P4 一致**：配置风格与 P4 项目保持一致，便于维护
4. **向后兼容**：支持所有 ESP32-C6 芯片版本

---

**更新完成！请重新编译并测试。** 🚀
