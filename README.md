# AV Metadata Packager

AV 视频元数据整理技能 —— 自动搜索元数据、下载横版封面、裁剪竖版海报、生成 Jellyfin 兼容 NFO，打包并移动到指定目录。

## 功能

- 🎯 自动识别视频番号，搜索元数据（女优、发行日、片商、时长、类别）
- 🖼️ 下载横版封面，自动裁剪右半部分作为竖版海报
- 📝 按标准模板生成 Jellyfin 兼容的 NFO 文件
- 📦 视频重命名为番号，与封面、NFO 一起打包到文件夹
- 🚚 自动移动到整理好的媒体库目录
- ⚙️ **首次运行自动引导设置文件夹路径**

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

## 首次使用

**首次运行时会自动询问你两个路径：**

1. **视频来源目录** — 你放待整理视频的文件夹
2. **整理后目标目录** — 打包完成后移动到的文件夹

设置好后会记住你的选择，后续直接使用。

## 工作流程

1. 首次运行询问用户设置 `source_dir` 和 `dest_dir`
2. 扫描来源目录下的视频文件
3. 搜索番号获取元数据
4. 下载横版封面图
5. 用 ffmpeg 裁剪横版封面右半部分作为竖版海报
6. 生成 NFO 文件（含 uniqueid、genre、tag、actor 等）
7. 视频重命名为番号（保持原扩展名）
8. 打包到以番号命名的文件夹
9. 移动到目标目录

### 最终目录结构

```
<dest_dir>/
└── <番号>/
    ├── <番号>.<ext>         🎬 视频
    ├── poster.jpg           🖼️ 竖版封面
    ├── landscape.jpg        🖼️ 横版封面
    └── <番号>.nfo           📄 元数据
```

## 依赖

- `ffmpeg` — 用于裁剪封面图（必需）
- `curl` — 下载封面（通常系统自带）
- `python3` — 用于获取图片尺寸

## 许可

MIT
