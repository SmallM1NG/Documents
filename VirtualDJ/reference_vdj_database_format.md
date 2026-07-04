---
name: VirtualDJ 数据库格式与目录结构
description: VirtualDJ 完整技术参考 — Windows注册表、用户目录布局、播放历史/播放列表/我的歌单三种列表文件格式、EXTVDJ标签字段、database.xml数据库结构、BPM转换公式
type: reference
---

# VirtualDJ 技术参考汇总

## 来源
- database.xml 格式: https://cn.virtualdj.com/wiki/VDJ_database.html (2026-07-04)
- 注册表与目录结构: 用户提供 (2026-07-04)
- 列表文件格式: 实际读取用户数据目录文件分析 (2026-07-04)

---

# ============================================================
# 第一部分：VDJ 安装位置与目录总览
# ============================================================

## 一、Windows 注册表

官方 MSI 安装包创建，路径: `HKEY_CURRENT_USER\Software\VirtualDJ`

| 值名称 | 说明 |
|---|---|
| RunFolder64 | VirtualDJ 安装目录 (如 `D:\VIrtualDJ\`) |
| HomeFolder | VirtualDJ 用户数据目录 (如 `C:\Users\USERNAME\AppData\Local\VirtualDJ`) |

该路径下无子键。

---

## 二、用户数据目录 (HomeFolder)

路径: `C:\Users\USERNAME\AppData\Local\VirtualDJ`

### 1. 完整子目录列表

| 子目录 | 用途说明 |
|---|---|
| Backup | 备份 |
| Cache | 缓存 |
| Devices | 设备包 |
| Drivers | 驱动 |
| **Folders** | 其他类型的歌单/文件夹（筛选器 Filters、本地曲库索引、在线音乐索引、Ideas 等），结构类似 MyLists（vdjfolder+order）。注意：NeteaseCloudMusic 等第三方插件生成的子目录并非 VDJ 默认自带 |
| **History** | 播放历史记录 |
| Languages | 语言包 |
| Mappers | 映射文件 |
| **MyLists** | 「我的歌单」(MyLists) |
| Pads | 打击垫 |
| **Playlists** | 「播放列表」(Playlists) |
| Plugins64 | 64位插件 |
| RemoteSkins | VirtualDJ Remote 皮肤 |
| Sampler | 采样包 |
| ScratchBanks | 搓碟素材包 |
| Sideview | 侧边视图 |
| Skins | 皮肤 |
| VideoSkins | 视频皮肤 |

---

# ============================================================
# 第二部分：列表文件 — History / Playlists / MyLists / Folders
# ============================================================

## 一、History 文件夹 — 播放历史

### 1. 目录结构

```
History/
├── 2026-07-04.m3u       ← 最新一段时间的历史(根目录直接存放)
├── 2026-06-28.m3u
├── tracklist.txt         ← 纯文本格式历史(同步写入)
├── 2023/                 ← 年份归档子目录
│   └── ...
├── 2024/
│   └── ...
├── 2025/
│   └── ...
└── 2026/
    ├── 01/
    ├── 02/
    │   └── 2026-02-01.m3u
    ├── ...
    └── 06/
        ├── 2026-06-01.m3u
        └── ...
```

注意: VDJ 会将较新的播放历史文件直接存放在 History 根目录下，一段时间后自动将其移动到 `年份/月份/` 归档子目录中。同一日期的历史文件**不会**同时出现在根目录和归档子目录中，只会存在于其中一处。

### 2. .m3u 文件格式 (History)

VDJ 每播放过一首曲目后就将其追加写入当天对应的 m3u 文件。

每首曲目占两行：
- 第1行: `#EXTVDJ:` 开头的元数据标签行（含 `<time>`、`<lastplaytime>`）
- 第2行: 文件绝对路径

格式模板:
```
#EXTVDJ:<time>HH:MM</time><lastplaytime>Unix时间戳</lastplaytime><filesize>字节数</filesize><artist>艺人名</artist><title>曲目标题</title><songlength>秒数</songlength>
文件绝对路径
```

实际示例:
```
#EXTVDJ:<time>12:36</time><lastplaytime>1782707791</lastplaytime><filesize>9744561</filesize><artist>Odymel/Durdenhauer</artist><title>Love Bullet (Pt.2)</title><songlength>241.622</songlength>
F:\曲库\Odymel,Durdenhauer - Love Bullet (Pt.2).mp3
```

### 3. tracklist.txt 格式

纯文本格式的历史记录，与 m3u 文件同步写入，按日期分段，最新内容追加在文件末尾：

```
VirtualDJ History YYYY/MM/DD
------------------------------
HH:MM : 艺人 - 曲目标题
HH:MM : 艺人 - 曲目标题
VirtualDJ History YYYY/MM/DD
------------------------------
...
```

实际示例:
```
VirtualDJ History 2025/06/20
------------------------------
21:32 : Yushiro - Just H Party 3.0
21:36 : Yushiro - Just H Party 3.0
22:09 : Fat Tony - Industry Baby (Remix)
VirtualDJ History 2025/06/21
------------------------------
12:00 : KREAM/Adam Port/stryv/Keinemusik/Orso/Malachiii - Move (KREAM Remix) [Extended Mix]
```

---

## 二、MyLists 文件夹 — 「我的歌单」

### 1. 文件类型

| 文件名 | 格式 | 说明 |
|---|---|---|
| `歌单名称.vdjfolder` | XML | 歌单的实际内容 |
| `order` | 纯文本 | 所有歌单在 VDJ 面板中的排列顺序 |

### 2. .vdjfolder 文件格式

XML 格式。根元素 `<VirtualFolder>`，每首曲目为一个 `<song>` 自闭合标签。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<VirtualFolder noDuplicates="no" singleDrive="yes">
	<song path="文件绝对路径" size="字节数" songlength="秒数" bpm="BPM值" key="调性" artist="艺人" title="曲目标题" idx="索引号" />
	...
</VirtualFolder>
```

**VirtualFolder 属性:**

| 属性 | 值 | 说明 |
|---|---|---|
| noDuplicates | "yes"/"no" | 是否不允许重复，通常 "no" |
| singleDrive | "yes"/"no" | 控制歌单的存储位置 — `"yes"` = 歌单存储在用户数据目录 `HomeFolder\MyLists` 下；`"no"` = 歌单存储在对应盘符根目录的 `X:\VirtualDJ\MyLists` 下 |

**song 元素属性:**

| 属性 | 必填 | 数据类型 | 说明 |
|---|---|---|---|
| path | **是** | string(绝对路径) | 音频文件路径 |
| size | **是** | int | 文件大小（字节） |
| idx | **是** | int(从0起) | 列表中排序索引 |
| artist | 否 | string | 艺人名 |
| title | 否 | string | 曲目标题 |
| songlength | 否 | float | 时长（秒） |
| bpm | 否 | float | BPM 值 |
| key | 否 | string | 音乐调性 (如 "Cm", "F#m") |
| remix | 否 | string | 混音类型标记 |

实际示例:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<VirtualFolder noDuplicates="no" singleDrive="yes">
	<song path="F:\Bass House\Skrillex - Bangarang (ChillPanic Remix).mp3" size="5571962" artist="Skrillex" title="Bangarang (ChillPanic Remix)" remix="ChillPanic Remix" idx="0" />
	<song path="F:\Bass House\bbno$,Ironmouse - 1-800.mp3" size="10195005" songlength="209.5" bpm="132.003" key="Em" artist="bbno$,Ironmouse" title="1-800" idx="1" />
	<song path="F:\Bass House\WA-FU - Bad Habits.mp3" size="10135053" songlength="213.7" bpm="132.000" key="Am" artist="WA-FU" title="Bad Habits" idx="2" />
</VirtualFolder>
```

### 3. order 文件格式 (MyLists)

纯文本，每行一个歌单名称（不含 `.vdjfolder` 扩展名）。分组分隔标题行也作为独立行存在。

```
歌单名称1
歌单名称2
----------(分隔符)-----------
歌单名称3
歌单名称4
```

---

## 三、Playlists 文件夹 — 「播放列表」

### 1. 文件类型

| 文件名 | 格式 | 说明 |
|---|---|---|
| `歌单名称.m3u` | EXTVDJ-M3U | 播放列表的实际内容 |
| `order` | 纯文本 | 所有播放列表在 VDJ 面板中的排列顺序 |

### 2. .m3u 文件格式 (Playlists)

与 History 的 m3u 使用相同的 EXTVDJ 扩展格式，但 **关键区别在于不含 `<time>` 和 `<lastplaytime>` 字段**。

每首曲目两行：
- 第1行: `#EXTVDJ:` 开头的元数据标签行
- 第2行: 文件绝对路径

格式模板:
```
#EXTVDJ:<filesize>字节数</filesize><artist>艺人名</artist><title>曲目标题</title><songlength>秒数</songlength>
文件绝对路径
```

实际示例:
```
#EXTVDJ:<filesize>5571962</filesize><artist>Skrillex</artist><title>Bangarang (ChillPanic Remix)</title><remix>ChillPanic Remix</remix>
F:\Bass House\Skrillex - Bangarang (ChillPanic Remix).mp3
#EXTVDJ:<filesize>10195005</filesize><artist>bbno$,Ironmouse</artist><title>1-800</title><songlength>209.516</songlength>
F:\Bass House\bbno$,Ironmouse - 1-800.mp3
```

### 3. order 文件格式 (Playlists)

与 MyLists 的 order 格式完全相同：
- 纯文本
- 每行一个列表名称（不含 `.m3u` 扩展名）
- 分组分隔标题行独立存在

---

## 四、Folders 文件夹 — 其他类型的文件夹（筛选器/索引等）

### 1. 目录结构

```
Folders/
├── order               ← 根级排序
├── Filters/            ← 筛选器文件夹
│   ├── order
│   ├── Recently added.vdjfolder
│   ├── Last played.vdjfolder
│   ├── Most played.vdjfolder
│   ├── Compatible songs.vdjfolder
│   ├── Decades.vdjfolder
│   ├── Duplicates.vdjfolder
│   ├── Genres.vdjfolder
│   └── My Filter.vdjfolder
├── LocalMusic/         ← 本地曲库索引 (排列作用，order 格式与其他类型一致)
├── OnlineMusic/        ← 在线音乐索引
├── Ideas/              ← Ideas 创意
└── Orders/             ← 订单
```

### 2. .vdjfolder 文件格式 (Filters)

筛选器类 vdjfolder 使用 `<FilterFolder>` 标签，与 MyLists 的 `<VirtualFolder>` 不同：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<FilterFolder filter="top 50 first seen" scope="database" />
```

**FilterFolder 属性:**

| 属性 | 说明 |
|---|---|
| filter | 筛选规则描述 (如 "top 50 first seen") |
| scope | 作用范围 (如 "database") |

### 3. order 文件格式

与其他类型的 order 格式一致——纯文本，每行一个名称（不含扩展名），分组标题行独立存在。

---

## 五、四种列表文件格式对比

注：所有类型歌单/文件夹的 order 文件都是各自独立的，彼此之间没有任何关联。

| 特性 | History (.m3u) | Playlists (.m3u) | MyLists (.vdjfolder) | Folders (.vdjfolder) |
|---|---|---|---|
| 文件格式 | EXTVDJ-M3U | EXTVDJ-M3U | XML | XML |
| 根元素 | 无 | 无 | `<VirtualFolder>` | `<FilterFolder>` (筛选器) 或 `<VirtualFolder>` |
| 有 `<time>` | 有 (HH:MM) | 无 | — | — |
| 有 `<lastplaytime>` | 有 (Unix时间戳) | 无 | — | — |
| 有 `<filesize>` | 有 | 有 | 有 (属性 "size") | 有 (属性 "size") |
| 有 `<artist>` | 有 | 有 | 有 (属性 "artist") | 有 (属性 "artist") |
| 有 `<title>` | 有 | 有 | 有 (属性 "title") | 有 (属性 "title") |
| 有 `<songlength>` | 有 | 有 | 有 (属性 "songlength") | 有 (属性 "songlength") |
| 有 `<remix>` | 可选 | 可选 | 有 (属性 "remix") | 有 (属性 "remix") |
| 有 BPM | 无 | 无 | 有 (属性 "bpm") | 有 (属性 "bpm") |
| 有 Key | 无 | 无 | 有 (属性 "key") | 有 (属性 "key") |
| 有 idx/索引 | 无 | 无 | 有 (属性 "idx" 从0起) | 有 (属性 "idx" 从0起) |
| 排序控制 | 按播放时间顺序 | 由 order 文件决定 | 由 order 文件决定 | 由 order 文件决定 |
| 重复控制 | 无 | 无 | `noDuplicates` 属性 | 视具体类型而定 |
| 编码 | UTF-8 | UTF-8 | UTF-8 | UTF-8 |
| 用途 | 播放历史 | 用户播放列表 | 用户我的歌单 | 筛选器/曲库索引/其他 |

---

## 六、EXTVDJ 标签字段总表

EXTVDJ 是 VirtualDJ 对标准 M3U 格式的扩展，采用 XML 风格内联标签嵌入 `#EXTVDJ:` 行中。不同上下文下出现的字段如下：

| 字段 | 含义 | 数据类型 | History | Playlists |
|---|---|---|---|---|
| `<time>` | 播放时间 (HH:MM 格式) | string | 有 | 无 |
| `<lastplaytime>` | 最后播放时间 (Unix 时间戳) | int | 有 | 无 |
| `<filesize>` | 文件大小（字节） | int | 有 | 有 |
| `<artist>` | 艺人名 | string | 有 | 有 |
| `<title>` | 曲目标题 | string | 有 | 有 |
| `<songlength>` | 歌曲时长（秒） | float | 有 | 有 |
| `<remix>` | 混音版本标记 | string | 有(可选) | 有(可选) |

---

# ============================================================
# 第三部分：database.xml 数据库格式
# ============================================================

## 一、database.xml 存储位置

VDJ 会在**每个盘符的根目录**下创建 `X:\VirtualDJ\` 文件夹，其中包含该盘符的 database.xml。另外用户数据目录下也有一个 database.xml。

**生成规则:** 当某个盘符内的任意位置有文件被 VDJ 播放或分析过后，该盘符根目录下即会生成 `X:\VirtualDJ\database.xml`。如果没有文件被扫描过则不会生成。

**各位置的 database.xml 及其作用:**

| 位置 | 记录内容 |
|---|---|
| `C:\Users\USERNAME\AppData\Local\VirtualDJ\database.xml` | C 盘中的文件 + 网络曲库播放过的曲目 |
| `D:\VirtualDJ\database.xml` | D 盘中的文件 |
| `F:\VirtualDJ\database.xml` | F 盘中的文件 |
| ... 以此类推 | |

---

## 二、重要警告

- 可以从外部应用程序**读取**数据库，但**强烈不推荐写入**
- Atomix Productions 不对由外部应用造成的数据库损坏负责，且不提供技术支持
- 自 VirtualDJ v6.0 起，XML 数据库使用 **UTF-8 编码**

## 三、XML 完整结构

```xml
<?xml version="1.0" encoding="UTF-8"?>
<VirtualDJ_Database Version="8.1">
  <Song FilePath="D:\\Music\\file1.mp3" FileSize="100000">
    <Tags .../>
    <Infos ... />
    <Scan .../>
    <Poi .../>
    <Poi .../>
    <Poi .../>
    <Comment>xxxx</Comment>
    <CustomMix>xxxx</CustomMix>
    <Link .../>
  </Song>
</VirtualDJ_Database>
```

## 四、元素与属性详解

### 1. VirtualDJ_Database (根元素)

| 属性 | 说明 |
|---|---|
| Version | 数据库架构版本号 (如 "8.1") |

### 2. Song (子元素, 可多个)

| 属性 | 说明 |
|---|---|
| FilePath | 音频文件的完整文件系统路径 |
| FileSize | 文件大小（字节） |
| Flag | 歌曲条目的状态标志 |

### 3. Tags (Song 的子元素) — 标签元数据

| 属性 | 说明 |
|---|---|
| Flag | 标签状态标志 |
| Author | 艺人/作者 |
| Title | 曲目标题 |
| Year | 年份 |
| Genre | 流派 |
| Bpm | 标签中记录的 BPM 值 |
| Key | 调性 |
| Album | 专辑名称 |
| Composer | 作曲家 |
| Label | 唱片厂牌 |
| TrackNumber | 音轨编号 |
| Remix | 混音类型 |
| Stars | 评分（星数） |
| Remixer | 混音师 |
| Grouping | 分组 |
| User1 | 用户自定义字段 1 |
| User2 | 用户自定义字段 2 |
| Internal | 内部标记 |

### 4. Scan (Song 的子元素) — 音频扫描分析结果

| 属性 | 说明 |
|---|---|
| Version | 扫描引擎版本号 |
| Flag | 扫描状态标志 |
| Volume | 分析出的音量/增益级别 |
| Bpm | **两拍之间的时间（秒）**，不是显示的 BPM（见转换公式） |
| AltBpm | 扫描引擎检测到的**第二可能的 BPM 值** |
| Key | 检测到的音乐调性 |

### 5. Infos (Song 的子元素) — 运行时统计信息

| 属性 | 说明 |
|---|---|
| SongLength | 歌曲时长 |
| Bitrate | 比特率 |
| Cover | 封面 |
| Color | 颜色 |
| FirstSeen | 首次被数据库扫描到的时间 |
| FirstPlay | 首次播放时间 |
| LastPlay | 最后播放时间 |
| PlayCount | 播放次数 |
| Corrupted | 文件是否损坏 |
| Faked | **[DEPRECATED]** |
| BpmTag | **[DEPRECATED]** |
| KeyTag | **[DEPRECATED]** |
| Gain | 增益值 |
| UserColor | 用户自定义颜色 |

### 6. Poi (Song 的子元素, 可多个) — Point of Interest (Cue点/循环等)

| 属性 | 说明 |
|---|---|
| Pos | POI 在歌曲中的位置 |
| Type | POI 类型 (cue, loop, automix 等) |
| Point | 具体点位引用 |
| Name | 用户为 POI 指定的名称 |
| Num | POI 编号/索引 |
| Bpm | 此 POI 位置的 BPM（与 Scan->Bpm 相同，存储的是两拍之间的秒数，需转换） |
| Size | POI 的尺寸/持续时间（用于 loop） |
| Color | POI 显示颜色 |
| Slot | POI 的卡槽分配 |

### 7. 其他 Song 子元素

| 元素 | 内容类型 | 说明 |
|---|---|---|
| Comment | 文本 | 用户备注 |
| CustomMix | 文本 | 自定义混音数据 |
| Link | (属性/子元素) | 链接相关数据 |

---

## 五、BPM 转换公式

`Scan->Bpm` 存储的值是**两拍之间的时间间隔（秒）**，不是常规意义下的 "每分钟节拍数"。

```
显示BPM = 1 / Scan->Bpm × 60
```

即：取 `Scan->Bpm` 值的倒数，再乘以 60。

`AltBpm` 是扫描引擎检测到的第二个最可能的 BPM 值。用户可在标签编辑器(Tag Editor)的 BPM 下拉菜单中看到 AltBpm。

注意: `.vdjfolder` 文件和 `Tags->Bpm` 属性中存储的 BPM 则是经过转换后的标准 BPM 值（如 "132.003"），不需要再转换。
