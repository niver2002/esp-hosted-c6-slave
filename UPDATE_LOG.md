# Slave 代码更新日志

## 2025-11-23 官方最新版本更新 + 性能优化

### 更新来源
- **官方仓库**: https://github.com/espressif/esp-hosted-mcu/tree/main/slave/main
- **更新时间**: 2025-11-23 21:35
- **版本**: main分支最新版本

### 更新的文件（全部36个文件）
所有 `slave/main/` 目录下的文件已更新至官方最新版本，包括：

#### 核心文件
- `esp_hosted_coprocessor.c` (33,861 → 33,928 bytes) ✅ **已应用优化**
- `slave_control.c` (153,257 bytes)
- `CMakeLists.txt` ✅ **已修复**

#### SDIO/SPI/UART 接口
- `sdio_slave_api.c` / `sdio_slave_api.h`
- `spi_slave_api.c`
- `spi_hd_slave_api.c`
- `uart_slave_api.c`

#### 蓝牙支持
- `slave_bt.c` / `slave_bt.h`
- `slave_bt_uart*.c` (ESP32/C3/S3/XX)

#### 内存管理
- `mempool.c` / `mempool.h`
- `mempool_ll.c` / `mempool_ll.h`

#### 其他组件
- `protocomm_pserial.c/.h`
- `lwip_filter.c/.h`
- `stats.c/.h`
- `host_power_save.c/.h`
- `http_req.c`
- `mqtt_example.c/.h`
- `interface.h`
- `slave_wifi_config.h`
- `idf_component.yml`
- `Kconfig.projbuild` (37,453 bytes)

---

## ⚡ 性能优化配置（已保留）

### 1. 禁用WiFi TX重试 ✅
```c
// esp_hosted_coprocessor.c 行64
#define MAX_WIFI_STA_TX_RETRY  0  // 禁用重试，避免vTaskDelay阻塞
```

**原因**：
- 官方默认 `MAX_WIFI_STA_TX_RETRY = 2`，会在失败时调用 `vTaskDelay(1ms)` 阻塞
- 重试会导致 recv_task (优先级22) 阻塞，影响SDIO接收性能
- WiFi driver内部已有队列和重传机制，应用层重试是冗余的

### 2. 移除重试循环逻辑 ✅
```c
// process_rx_pkt() 函数 (行657-671)
if (buf_handle->if_type == ESP_STA_IF && station_connected) {
    /* Forward data to wlan driver - 无重试，避免阻塞 */
#if ESP_PKT_STATS
    int ret = esp_wifi_internal_tx(WIFI_IF_STA, payload, payload_len);
#else
    esp_wifi_internal_tx(WIFI_IF_STA, payload, payload_len);
#endif
    // 无 do-while 重试循环
    // 无 vTaskDelay 阻塞
}
```

**官方代码（已移除）**：
```c
int retry_wifi_tx = MAX_WIFI_STA_TX_RETRY;
do {
    ret = esp_wifi_internal_tx(WIFI_IF_STA, payload, payload_len);
    if (ret) {
        vTaskDelay(pdMS_TO_TICKS(1));  // ❌ 阻塞1ms
    }
    retry_wifi_tx--;
} while (ret && retry_wifi_tx);
```

### 3. 增加TO_HOST_QUEUE_SIZE ✅
```c
// esp_hosted_coprocessor.c 行61
#define TO_HOST_QUEUE_SIZE  20  // 增加到20，提高WiFi→Host下载吞吐量
```

**原因**：
- 官方默认 `TO_HOST_QUEUE_SIZE = 10`
- 增加队列深度减少WiFi→SDIO方向的丢包
- 提高下载吞吐量（虽然启用了BYPASS模式，但定义保留）

### 4. 保持BYPASS_TX_PRIORITY_Q ✅
```c
// esp_hosted_coprocessor.c 行57
#define BYPASS_TX_PRIORITY_Q  1  // 官方已启用，直接TX不入队
```

**说明**：
- 官方最新版本已经启用此优化
- `send_to_host_queue()` 直接调用 `process_tx_pkt()`
- 减少队列延迟和内存占用

---

## 📊 优化效果预期

### 吞吐量
- ⬆️ **上传（P4→WiFi）**: 减少SDIO→WiFi的阻塞延迟
- ⬆️ **下载（WiFi→P4）**: 更大的队列缓冲，减少丢包

### 延迟
- ⬇️ **平均延迟**: 消除1ms重试阻塞
- ⬇️ **抖动**: recv_task不会因重试而暂停

### 稳定性
- ✅ **无recv_task阻塞**: 高优先级任务能及时响应
- ✅ **依赖WiFi内部机制**: 让硬件层处理重传

### 资源使用
- ✅ **CPU利用率**: 无无效重试消耗
- ✅ **内存**: BYPASS模式减少队列占用

---

## 🔧 其他修改

### CMakeLists.txt 修复
```cmake
# 修复CLI支持为条件编译
if(CONFIG_ESP_HOSTED_CLI_ENABLED)
    list(APPEND COMPONENT_ADD_INCLUDEDIRS "${common_dir}/utils")
    list(APPEND COMPONENT_SRCS "${common_dir}/utils/esp_hosted_cli.c")
endif()
```

**修复内容**：
- 移除重复的 `uart_slave_api.c`
- 移除多余的末尾字符
- CLI代码仅在启用时编译

---

## ✅ 验证检查清单

### 编译测试
- [ ] `idf.py build` 成功
- [ ] 无编译警告
- [ ] 链接成功

### 运行时测试
- [ ] SDIO连接正常
- [ ] WiFi连接AP成功
- [ ] ping测试通过
- [ ] iperf3上传测试
- [ ] iperf3下载测试

### 统计验证
```bash
# 查看30秒统计报告
I STATS: === 30s Report ===
I STATS: ETH→WiFi: recv/sent/drop
I STATS: WiFi→ETH: recv/sent/drop
I STATS: TX errors: X (应该很低)
```

### 预期指标
- 丢包率: < 1%
- TX errors: < 0.5%
- 吞吐量: 接近AMPDU理论值
- 延迟抖动: 显著减少

---

## 📝 备份信息
- 旧版本备份: `esp_hosted_coprocessor.c.backup`
- 备份时间: 2025-11-23 21:35

## 🚀 下一步
1. **立即编译测试** - `idf.py build`
2. **烧录测试** - 验证SDIO/WiFi连接
3. **性能测试** - iperf3基准测试
4. **长时间稳定性测试** - 24小时运行
