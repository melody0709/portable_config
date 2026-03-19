# MPV IPTV Player with EPG & Catch-up

基于 [mpv](https://mpv.io/) + [uosc](https://github.com/tomasklaen/uosc) 5.12（已定制）的 IPTV 播放器，支持 M3U 分组、EPG 节目单和回看功能。

## ✨ 功能特性

- 📺 **频道分组** - 自动解析 M3U 的 `group-title`，按分组展示频道
- 📋 **EPG 节目单** - 支持 `x-tvg-url` 加载 XMLTV 格式节目单
- ⏪ **回看功能** - 支持 `catchup-source` 格式，点击节目即可回看
- 🎨 **精美界面** - 基于 uosc 的现代化菜单界面
- ⌨️ **快捷键支持** - F8 打开频道菜单（鼠标右键同样可打开）

## 📁 目录结构

```
portable_config/
├── mpv.conf              # mpv 配置文件
├── scripts/
│   ├── epg.lua           # IPTV EPG 主脚本 (本功能核心)
│   ├── thumbfast.lua     # 缩略图脚本
│   └── uosc/             # uosc 界面库（已根据本项目定制，默认使用此目录内脚本）
│       ├── main.lua
│       └── ...
├── script-opts/
│   ├── thumbfast.conf    # 缩略图配置
│   └── uosc.conf         # uosc 配置

```

## 🚀 使用方法

### 1. 准备 M3U 播放列表

确保你的 M3U 文件包含以下字段：

```m3u
#EXTM3U x-tvg-url="http://your-epg-server/t.xml"

#EXTINF:-1 
  tvg-id="CCTV1" 
  tvg-name="CCTV-1高清" 
  tvg-logo="http://example.com/logo.png" 
  group-title="央视" 
  catchup="default" 
  catchup-source="http://server/ch{id}?starttime=${(b)yyyyMMddHHmmss|UTC}&endtime=${(e)yyyyMMddHHmmss|UTC}",
CCTV-1综合
http://your-stream-url/playlist.m3u8
```

**关键字段说明：**

| 字段               | 说明                        |
| ------------------ | --------------------------- |
| `x-tvg-url`      | EPG XML 文件地址            |
| `tvg-id`         | 频道唯一标识，需与 EPG 匹配 |
| `group-title`    | 分组名称                    |
| `catchup-source` | 回看 URL 模板               |

### 2. 打开 M3U 文件

```bash
mpv your-playlist.m3u
```

或者用 mpv 打开 M3U 文件后，脚本会自动加载并显示提示：

```
IPTV 已加载！F8:选台
```

### 3. 快捷键操作

| 快捷键 | 功能             |
| ------ | ---------------- |
| `F8` | 打开频道分组菜单 |

## 📸 界面预览

### 频道分组菜单

```
📁 IPTV 直播源
├── 📂 央视 (12 频道)
│   ├── 📺 CCTV-1综合
│   ├── 📺 CCTV-2财经
│   └── ...
├── 📂 卫视 (8 频道)
└── 📅 查看当前频道节目单（右键菜单）
```

### EPG 节目单菜单

```
📅 节目单

今天 08:00 朝闻天下        [回看] ⏪
今天 09:00 新闻直播间      [回看] ⏪
今天 12:00 新闻30分        [直播中] 🔴
今天 19:00 新闻联播        [预告]
```

## ⚙️ 配置说明

### 脚本配置（epg.lua 内）

如需修改默认行为，可编辑 `scripts/epg.lua`：

```lua
local state = {
    m3u_path = "",           -- M3U 文件路径（自动检测）
    epg_url = "",            -- EPG URL（从 M3U 解析）
    groups = {},             -- 分组数据
    group_names = {},        -- 分组名称列表
    epg_data = {},           -- EPG 节目数据
    is_loaded = false,       -- 是否已加载
    current_channel = nil    -- 当前播放频道
}
```

### 快捷键绑定

在 `mpv.conf` 中添加或修改：

```ini
# IPTV 快捷键
F8 script-binding epg/show-iptv-menu  # 打开频道菜单
```

### EPG 下载 URL（可选）

脚本会优先使用 `script-opts/epg.conf` 中的 `epg_download_url` 配置来下载 EPG 数据，若该值为空则会回退到 M3U 中的 `x-tvg-url` 字段。

`script-opts/epg.conf` 示例：

```ini
# EPG 下载连接配置
# 当此参数存在时，优先使用该连接下载 EPG，而不是 M3U 表头中的 x-tvg-url
epg_download_url=http://your-epg-source.com/epg.xml
```

## 🔧 故障排除

### EPG 无法加载

1. **检查网络连接**

   ```bash
   curl -I http://your-epg-server/t.xml
   ```
2. **检查 EPG URL 格式**

   - 确保 M3U 中的 `x-tvg-url` 是可访问的 XML 文件
   - 文件编码应为 UTF-8
3. **查看 mpv 日志**

   ```bash
   mpv your-playlist.m3u --log-file=mpv.log
   ```

   搜索 `epg.lua` 相关日志

### 回看功能不工作

1. **检查 `catchup-source` 格式**

   - 必须包含 `${(b)yyyyMMddHHmmss|UTC}` 和 `${(e)yyyyMMddHHmmss|UTC}`
2. **检查时间格式**

   - 确保 EPG 中的 `start` 和 `stop` 时间是标准 XMLTV 格式

### 菜单不显示

1. **确认 uosc 已正确安装**

   ```
   scripts/
   └── uosc/
       ├── main.lua
       └── ...
   ```
2. **检查 uosc 版本**

   - 本脚本需要 uosc 5.12+ 版本
3. **查看控制台错误**
   按 `` ` ``（反引号）打开控制台查看错误信息

## 📝 更新日志

### v5.1 (2026-01-18)

- ✅ 修复图标杂点问题 - 移除远程 URL 图标，使用本地 Material Icon
- ✅ 优化 EPG 时间解析，支持更多时区格式
- ✅ 改进回看 URL 生成逻辑

### v5.0 (2026-01-15)

- ✅ 初始版本发布
- ✅ 支持 M3U 分组解析
- ✅ 支持 EPG 节目单
- ✅ 支持回看功能
- ✅ 集成 uosc 菜单系统

## 🤝 致谢

- [mpv](https://mpv.io/) - 强大的媒体播放器
- [uosc](https://github.com/tomasklaen/uosc) - 优秀的 mpv 界面脚本
- [XMLTV](http://wiki.xmltv.org/) - EPG 数据格式标准

## 📄 许可证

本项目采用 MIT 许可证 - 详见 [LICENSE](LICENSE) 文件

---

**注意**: 本脚本仅供学习研究使用，请遵守当地法律法规，合理使用 IPTV 资源。
