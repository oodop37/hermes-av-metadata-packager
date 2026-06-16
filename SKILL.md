---
name: av-metadata-packager
description: "AV 视频元数据整理流程：搜索视频→查找元数据→下载横版封面→裁剪右半部为竖版封面→按模板写 NFO→打包文件夹→移动到目标目录"
version: 2.0
author: 老婆
tags: [av, jellyfin, nfo, metadata, cover]
---

# AV 元数据打包流程

## ⚙️ 首次使用：设置文件夹路径

**本技能首次运行时，会向你询问两个路径：**

1. **视频来源目录** — 你放待整理视频的文件夹
2. **整理后目标目录** — 打包完成后移动到的文件夹

请提前准备好这两个路径，例如：
- 来源：`/volume1/视频/待整理`
- 目标：`/volume1/视频/已整理`

设置好后，技能会记住你的选择，后续直接使用。

## 触发条件

用户在来源目录下有新视频，需要整理元数据并打包到目标目录。

## 工作流程

### 1. 询问文件夹路径（首次运行）
首次使用时，向用户询问：
- `source_dir` — 视频来源目录（如 `/volume1/视频/待整理`）
- `dest_dir` — 整理后目标目录（如 `/volume1/视频/已整理`）

后续运行直接使用已保存的路径。

### 2. 列出视频文件
```bash
ls -la "<source_dir>/"
```
只处理视频文件（.mp4, .avi, .mkv, .ts, .rmvb 等），跳过已存在的文件夹。

### 3. 搜索元数据
用番号（文件名中的字母+数字组合，如 CEAD-751、NITR-078）搜索：
- `web_search(query="<番号> JAV metadata")`
- 重点找：女优、发行日期、片商、时长、标题、标签/类别

### 4. 找横版封面
从搜索结果中找到带封面图的页面，用浏览器打开获取图片链接。

常见封面来源：
- **akiba-online.com** — 帖子里的附件图，横版（通常 328×220 左右）
- **pornjav.org** — 封面图通常在 `uploads/posts/` 路径下
- **dmm.co.jp** — `pics.dmm.co.jp/mono/movie/adult/<番号小写>/<番号小写>pl.jpg`（横版大图）
- **dmm.co.jp 老片备选** — `pics.dmm.co.jp/digital/video/<content_id>/<content_id>pl.jpg`

### 5. 下载横版封面
```bash
curl -sL -o "landscape.jpg" "<图片URL>" -H "User-Agent: Mozilla/5.0"
```

### 6. 裁剪竖版封面（横版右半部分）
用 ffmpeg 裁剪横版封面右半部分：
```python
python3 -c "
import struct, subprocess
with open('landscape.jpg', 'rb') as f:
    data = f.read()
i = 0
while i < len(data):
    if data[i] == 0xFF and data[i+1] in [0xC0, 0xC1, 0xC2]:
        height = struct.unpack('>H', data[i+5:i+7])[0]
        width = struct.unpack('>H', data[i+7:i+9])[0]
        break
    i += 1
subprocess.run(['ffmpeg', '-y', '-i', 'landscape.jpg',
    '-vf', f'crop={width//2}:{height}:{width//2}:0',
    '-q:v', '2', 'poster.jpg'], capture_output=True)
"
```

### 7. 写 NFO 文件
严格按以下模板写，文件名为 `<番号>.nfo`：

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes" ?>
<movie>
  <title><番号></title>
  <originaltitle><原标题（日文）></originaltitle>
  <rating><评分，没有就填0></rating>
  <runtime><时长（分钟）></runtime>
  <mpaa>NC-17</mpaa>
  <uniqueid type="num" default="true"><番号></uniqueid>
  <uniqueid type="cid"><番号小写></uniqueid>
  <!-- genre 和 tag 各写一份，内容相同 -->
  <genre><类别1></genre>
  <genre><类别2></genre>
  <tag><类别1></tag>
  <tag><类别2></tag>
  <country>日本</country>
  <premiered><发行日期 YYYY-MM-DD></premiered>
  <studio><片商></studio>
  <actor>
    <name><女优名></name>
    <role>女优</role>
  </actor>
</movie>
```

### 8. 重命名视频文件为番号
```bash
# 获取视频文件扩展名
EXT=".<视频文件扩展名>"
# 重命名为番号+原扩展名
mv "<视频文件>" "<番号>$EXT"
```

### 9. 创建文件夹并打包
```bash
# 创建以番号命名的文件夹
mkdir -p "<source_dir>/<番号>"

# 移动视频、封面、NFO 到文件夹
mv "<番号>$EXT" "<source_dir>/<番号>/"
mv "landscape.jpg" "<source_dir>/<番号>/"
mv "poster.jpg" "<source_dir>/<番号>/"
mv "<番号>.nfo" "<source_dir>/<番号>/"
```

### 10. 移动到目标目录
```bash
mv "<source_dir>/<番号>" "<dest_dir>/"
```

## 最终目录结构
```
<dest_dir>/
└── <番号>/
    ├── <番号>.<ext>         🎬 视频（已重命名）
    ├── poster.jpg           🖼️ 竖版封面（横版右半裁剪）
    ├── landscape.jpg        🖼️ 横版封面
    └── <番号>.nfo           📄 元数据
```

## 注意事项
- 竖版封面**不要**去网上找，直接从横版封面右半部分裁剪
- NFO 的 `<genre>` 和 `<tag>` 内容一致，各写一份
- `<uniqueid type="num">` 用番号（大写），`<uniqueid type="cid">` 用番号小写
- **视频文件重命名为番号**（保持原扩展名不变），文件夹名也用番号
- 横版封面用 `landscape.jpg`，竖版封面用 `poster.jpg`
- NFO 文件名用 `<番号>.nfo`

## 常见陷阱

### DMM 封面 URL 有两个路径
DMM 的封面图有两种路径，尺寸差异很大：
- **新片/主流片**：`pics.dmm.co.jp/mono/movie/adult/<番号小写>/<番号小写>pl.jpg` — 800×536 大图 ✅
- **老片**：同上路径可能只有 90×122 极小图 ❌
- **老片备选**：`pics.dmm.co.jp/digital/video/<content_id>/<content_id>pl.jpg` — 800×536 大图 ✅
  - content_id 可以从 javtrailers.com 等站点的页面 URL 中找到（如 `49nitr00078`）
  - 或者从 javlibrary 的搜索结果中找 Content ID

如果 `mono/movie/adult/` 路径返回的图太小（< 200px），立即尝试 `digital/video/` 路径。

### 文件名可能不含番号
有些视频文件名是中文描述而非番号（如 `[高清有碼]精液射精肛門喜歡柔軟的身體的女人(内村里菜).rmvb`），此时：
1. 用女优名 + 关键词搜索找出番号
2. 视频重命名为番号（保持原扩展名），如 `NITR-078.rmvb`
3. 文件夹名也用番号
4. NFO 文件名用 `<番号>.nfo`
