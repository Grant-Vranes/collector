@ -0,0 +1,85 @@
# Collector 项目概览

**简介**: GNOME 桌面拖拽文件管理工具，支持多窗口、图片下载、剪贴板粘贴等功能。

## 技术栈

| 类型     | 技术                                                   |
| -------- | ------------------------------------------------------ |
| 语言     | Python 3                                               |
| GUI框架  | GTK 4 + LibAdwaita                                     |
| 构建系统 | Meson                                                  |
| 打包分发 | Flatpak                                                |
| 依赖库   | Pillow (图片处理), requests (HTTP), pybind11 (C++绑定) |

## 运行方式

### Flatpak 方式 (推荐)

```bash
flatpak-builder --user --install _flatpak it.mijorus.collector.json
flatpak run it.mijorus.collector

# 调试模式
flatpak run --env=APP_DEBUG=1 it.mijorus.collector

# 多窗口
flatpak run it.mijorus.collector --w=3
```

### 本地构建

```bash
meson setup _build
meson compile -C _build
meson install -C _build
```

## 目录结构

```
collector/
├── src/                    # 源代码
│   ├── main.py            # 应用入口
│   ├── window.py          # 主窗口 (核心逻辑)
│   ├── preferences.py     # 设置窗口
│   ├── lib/               # 工具模块
│   │   ├── DroppedItem.py # 拖拽项封装
│   │   ├── CarouselItem.py# 轮播项
│   │   ├── CsvCollector.py# CSV收集器
│   │   ├── constants.py   # 常量
│   │   └── utils.py       # 工具函数
│   ├── gtk/               # UI模板
│   └── assets/            # CSS样式
├── data/                   # 桌面文件、图标、GSchema
├── po/                     # 国际化翻译文件
├── it.mijorus.collector.json  # Flatpak构建清单
└── meson.build            # Meson构建配置
```

## 核心功能

- 拖拽文件/文件夹到窗口收集
- 支持从浏览器拖拽图片自动下载
- 多窗口支持 (`--w=3`)
- 剪贴板粘贴 (`Ctrl+V`)
- 文本收集导出为CSV
- Google Images 支持
- 窗口颜色自定义

## 依赖说明

- **GTK 4.0 / Adw 1**: 现代 GNOME GUI 框架
- **Pillow**: 图片裁剪、缩略图生成
- **requests**: HTTP 请求，用于图片下载
- **pybind11**: C++ 扩展支持

## 调试

```bash
# 启用调试日志
APP_DEBUG=1 flatpak run it.mijorus.collector

# 日志位置
# $XDG_CACHE_DIR/logs/collector.log
```