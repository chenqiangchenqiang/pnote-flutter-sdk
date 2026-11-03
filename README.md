# pnote_flutter_sdk（lite 精简版）

声云录音笔 BLE Flutter 插件 **lite 精简版** —— 纯 Dart 实现，基于 flutter_blue_plus，不依赖厂商原生 SDK。

## 功能一览

| 能力 | 说明 |
|------|------|
| 扫描 / 连接 | 按 0xAE20 服务过滤；支持按名+地址重连（`connectDeviceAndAddress`） |
| 设备信息 | 时间同步 / 电量 / 容量 / 固件版本 / MAC |
| 录音控制 | 开始 / 停止 / 暂停 / 继续 / 状态查询；接收设备端物理按键与暂停继续事件 |
| 文件管理 | 列表 / 下载（断点续传、区间分段）/ 删除单个 / 删除全部 |
| AppJson 兼容层 | 产出与厂商 SDK 完全一致的 JSON 事件（`getCBCAppJson()` 等 ×11） |

## 快速上手

### 1. 添加依赖

```yaml
dependencies:
  pnote_flutter_sdk:
    git:
      url: https://gitee.com/chenqiangchenqiang/pnote-flutter-sdk.git
```

### 2. 配置权限

**Android** `AndroidManifest.xml`：

```xml
<uses-permission android:name="android.permission.BLUETOOTH_SCAN"
    android:usesPermissionFlags="neverForLocation" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
<!-- Android 11 及以下 -->
<uses-permission android:name="android.permission.BLUETOOTH" android:maxSdkVersion="30" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" android:maxSdkVersion="30" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```

**iOS** `Info.plist`：

```xml
<key>NSBluetoothAlwaysUsageDescription</key>
<string>用于连接录音笔</string>
```

运行时权限需在 App 内动态申请（示例工程依赖系统弹窗，正式接入建议用 permission_handler）。

### 3. 使用

```dart
final sdk = AudioPenSdk();

// 扫描（按 0xAE20 过滤）
sdk.startSearch().listen((results) {
  // results: List<ScanResult>（flutter_blue_plus 类型）
});

// 连接
final device = await sdk.connectDevice(results.first.device);

// 基础信息
await device.syncTime();
final battery = await device.getCBC();          // BatteryInfo
final capacity = await device.getDeviceCapacity();
final version = await device.getDeviceVersion();

// 录音
await device.startRecord();
await device.stopRecord();

// 文件
final files = await device.getRecordFileList(); // List<PenFile>
await device.startGetFile(files.first.name);    // 经 fileDataStream 推流
await device.delAllFileData();

// 断开
await device.closeConnect();
```

## API 速览（`AudioPenDevice` 常用）

| 分类 | 方法 |
|------|------|
| 连接 | `isDeviceConnected` / `connectionState` / `closeConnect()` |
| 信息 | `syncTime()` / `getCBC()` / `getDeviceCapacity()` / `getDeviceVersion()` / `getSn()`（协议无 SN，返回 MAC） |
| 录音 | `startRecord()` / `stopRecord()` / `pauseRecord()` / `continueRecord()` / `getRecordState()` |
| 文件 | `getRecordFileList()` / `startGetFile(name, {offset})` / `startGetFileByFromToEnd(name, {offset, end})` / `stopGetFile([fileName])` / `delFileData(name)` / `delAllFileData()` |
| 流 | `statusStream` / `keyEventStream` / `deviceRecordStream` / `fileDataStream` / `downloadProgressStream` / `appEventStream` |
| AppJson | `getCBCAppJson()` / `getRecordFileListAppJson()` / `startRecordAppJson()` …（共 11 个，输出厂商同形 JSON） |

完整签名与协议细节见 [ARCHITECTURE.md](ARCHITECTURE.md)（帧格式 / 指令表 / 真机校准点 / 调试日志开关）。

## 注意事项

- **BLE 必须真机调试**：模拟器无蓝牙，扫描列表恒空。
- 调试开关：`AudioPenSdk(debugLog: true)` 打印扫描/GATT/原始字节日志（`[BLE-SCAN]` 等前缀）；连接后可用 `device.debugLog = false` 关闭；`sdk.scanAll()` 可不过滤扫描排查广播问题。
- 协议层（帧编解码 / CRC / 回包解析）无蓝牙依赖，可离线单元测试（`flutter test`，48 个用例）。

## License

MIT（见 [LICENSE](LICENSE)）。
