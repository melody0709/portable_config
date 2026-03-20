# MPV IPTV Player with EPG & Catch-up

 基于 mpv + uosc 5.12（已定制）的 IPTV 播放器，支持：

- M3U 频道分组
- EPG 节目单（x-tvg-url）
- 回看（catchup-source）
- 频道历史记忆（m3u 文件级别）

## 快速使用

1. 下载 mpv Windows 版（推荐 shinchiro mpv-winbuild）
2. 把 `portable_config` 放到 下载好的mpv 根目录
3. 双击 M3U，或命令行：
   ```bash
   mpv tv.m3u
   ```

## 目录说明

- `mpv.conf`：全局 mpv 配置
- `input.conf`：快捷键
- `scripts/epg.lua`：IPTV EPG 主脚本
- `scripts/uosc/`：定制 uosc 组件
- `script-opts/epg.conf`：EPG 备用 URL 配置

## 核心功能

- 解析 M3U `group-title` 分组、频道
- 自动加载 `x-tvg-url` EPG（xml/xml.gz）
- 支持 `catchup-source` 3 种回看模板
- 支持 `epg_history.json` 记录最后播放频道（每个 m3u）

## M3U 模板

```m3u
 #EXTM3U x-tvg-url="http://your-epg-server/t.xml"
 #EXTINF:-1 tvg-id="CCTV1" tvg-name="CCTV-1高清" tvg-logo="http://..." group-title="央视" catchup="default" catchup-source="http://server/ch{id}?starttime=${utc:yyyyMMddHHmmss}&endtime=${utcend:yyyyMMddHHmmss}",CCTV-1综合
 http://your-stream-url/playlist.m3u8
```

## 字段说明

- `x-tvg-url`：EPG 数据源
- `tvg-id`：频道 ID（与 EPG channel id 关联）
- `group-title`：频道分组
- `catchup-source`：回看 URL 模板

## 频道历史记录

- 文件位置：`scripts/epg_history.json` 或 `%TEMP%/mpv_epg_epg_history.json`
- 格式：`{ "path/to/tv.m3u": { "url":"...", "name":"...", "group":"...", "timestamp":123456789 } }`
- 下次打开同一路径的 m3u 将自动还原上次频道

## 配置示例

 `script-opts/epg.conf`

```ini
 epg_download_url=http://your-epg-source.com/epg.xml
```

## 运行日志调试

```bash
 mpv --msg-level=all=info,epg=trace tv.m3u
```

 关注 `epg.lua` 输出：历史记录路径、EPG 下载状态、自动恢复频道

## 常见问题

- 无 EPG：确认 `x-tvg-url` 可访问
- 无回看：检查 `catchup-source` 时间模板
- 无历史记忆：确认脚本有写权限

## 版本更新

- v5.2 (2026-03-20): 新增频道历史记忆、从历史自动恢复
- v5.1 (2026-01-18): EPG 时间解析修复
- v5.0 (2026-01-15): 首次发布

## 许可证

 MIT

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

   - 支持以下三种时间参数格式：
     - **标准回看模板**: `${utc:yyyyMMddHHmmss}` 和 `${utcend:yyyyMMddHHmmss}`
     - **KU9回看模板**: `${(b)yyyyMMddHHmmss|UTC}` 和 `${(e)yyyyMMddHHmmss|UTC}`
     - **APTV回看模板**: `${(b)yyyyMMddHHmmss:utc}` 和 `${(e)yyyyMMddHHmmss:utc}`
2. **检查时间格式**

   - 确保 EPG 中的 `start` 和 `stop` 时间是标准 XMLTV 格式

## 📝 更新日志

### v5.2 (2026-03-19)

- ✅ **增强回看功能兼容性** - 新增支持三种时间参数格式：
  - 标准回看模板：`${utc:yyyyMMddHHmmss}` 和 `${utcend:yyyyMMddHHmmss}`
  - KU9回看模板：`${(b)yyyyMMddHHmmss|UTC}` 和 `${(e)yyyyMMddHHmmss|UTC}`
  - APTV回看模板：`${(b)yyyyMMddHHmmss:utc}` 和 `${(e)yyyyMMddHHmmss:utc}`
- ✅ 简化时间参数替换函数，移除调试信息输出
- ✅ 更新文档，提供完整的时间参数格式说明

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
