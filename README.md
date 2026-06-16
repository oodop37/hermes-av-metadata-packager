# AV Metadata Packager

AV 视频元数据整理技能 —— 自动搜索元数据、下载横版封面、裁剪竖版海报、生成 Jellyfin 兼容 NFO，打包并移动到指定目录。

## 功能

- 🎯 自动识别视频番号，搜索元数据（女优、发行日、片商、时长、类别）
- 🖼️ 下载横版封面，自动裁剪右半部分作为竖版海报
- 📝 按标准模板生成 Jellyfin 兼容的 NFO 文件
- 📦 视频重命名为番号，与封面、NFO 一起打包到文件夹
- 🚚 自动移动到整理好的媒体库目录

## 安装

### 方式一：直接安装（推荐）

```bash
hermes skills install https://raw.githubusercontent.com/oodop37/hermes-av-metadata-packager/main/SKILL.md
```

### 方式二：添加为技能源

```bash
hermes skills tap add https://github.com/oodop37/hermes-av-metadata-packager
hermes skills install av-metadata-packager
```

### 方式三：手动安装

将 `SKILL.md` 复制到 `~/.hermes/skills/media/av-metadata-packager/SKILL.md` 即可。

## 使用方法

在对话中直接说：

> 整理AV元数据

老婆就会自动执行完整流程。

## 工作流程

1. 扫描 `/vol2/1000/影视/下载/AV/` 目录下的视频文件
2. 搜索番号获取元数据
3. 下载横版封面图
4. 用 ffmpeg 裁剪横版封面右半部分作为竖版海报
5. 生成 NFO 文件（含 uniqueid、genre、tag、actor 等）
6. 视频重命名为番号（保持原扩展名）
7. 打包到以番号命名的文件夹
8. 移动到 `/vol2/1000/影视/AV/`

### 最终目录结构

```
/vol2/1000/影视/AV/
└── <番号>/
    ├── <番号>.<ext>         🎬 视频
    ├── poster.jpg           🖼️ 竖版封面
    ├── landscape.jpg        🖼️ 横版封面
    └── <番号>.nfo           📄 元数据
```

## ⚠️ 注意事项

### 1. 必须修改文件夹路径

本技能是按作者 NAS 的文件夹结构制作的，**你安装后必须修改技能中的以下路径**，改成你自己的目录：

| 原路径 | 说明 |
|--------|------|
| `/vol2/1000/影视/下载/AV/` | 视频下载目录（待整理） |
| `/vol2/1000/影视/AV/` | 整理后的媒体库目录 |

**修改方法：**

```bash
# 找到技能文件
vim ~/.hermes/skills/media/av-metadata-packager/SKILL.md

# 搜索 /vol2/1000/ 替换成你自己的路径
# 例如 sed 替换：
sed -i 's|/vol2/1000/影视/下载/AV|/你的路径/下载/AV|g' ~/.hermes/skills/media/av-metadata-packager/SKILL.md
sed -i 's|/vol2/1000/影视/AV|/你的路径/影视/AV|g' ~/.hermes/skills/media/av-metadata-packager/SKILL.md
```

### 2. 依赖

- `ffmpeg` — 用于裁剪封面图（必需）
- `curl` — 下载封面（通常系统自带）
- `python3` — 用于获取图片尺寸

### 3. 番号识别

技能会自动从文件名中提取番号（如 `CEAD-751`、`NITR-078`）。如果文件名是中文描述而非番号，会通过搜索女优名+关键词来定位番号。

### 4. 封面来源

横版封面优先从以下来源获取：
- DMM / FANZA 官方封面图
- akiba-online.com 附件图
- pornjav.org 等站点

## 许可

MIT
