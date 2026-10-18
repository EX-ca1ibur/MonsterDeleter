# 👹 MonsterDeleter · 召唤大将怪兽摧毁文件

一个 Windows 桌面小玩具：删除文件不再是系统弹窗，而是**召唤一只大将怪兽**——它从屏幕外走来，指着你要删的文件问一句「喂，是这个吗？」，你确认后它一记飞踹，文件在爆炸中进了回收站，然后怪兽扬长而去。

> ⚠️ **本仓库只收录源代码**（`main.py`）。`assets/` 素材（约 26 MB）与打包好的 `MonsterDeleter.exe`（约 76 MB）因体积原因没有入库，详见 [关于素材](#关于素材)。

---

## ✨ 它做什么

- **右键菜单集成**：首次运行时自动写入注册表 `HKCU\Software\Classes\*\shell\SummonMonster`，任意文件的右键菜单里多出「召唤大将怪兽摧毁」。
- **全屏狙击界面**：半透明遮罩淡入，鼠标变成红色准星光标，提示「请选择你要摧毁的文件」。
- **六套逐帧动画**：走路、指着文件、踹文件、爆炸、雷欧登场、出场飞行，各是一张 5×3 的 spritesheet，按 8 fps 播放。
- **三路独立音轨**：BGM（循环）、怪兽语音、爆炸音效，各用一个 `QMediaPlayer` + `QAudioOutput`，音量 0.5 / 1.0 / 0.3。
- **删除到回收站**：通过 `send2trash` 移入回收站，**不是永久删除**，踹错了还能捞回来。

## 🎞️ 动画流程

| 阶段 | 触发 | 表现 |
| --- | --- | --- |
| 1. 选目标 | 程序启动 | 遮罩淡入（800 ms），准星光标 |
| 2. 出场 | 点击屏幕 | 遮罩淡出（500 ms），怪兽从左侧走入，4.5 s |
| 3. 指认 | 走到文件旁 | 怪兽语音 + 指着文件的动画，气泡「喂，是这个吗？」 |
| 4. 确认 | 点击按钮 | 三个选项：**是的** / **嘤嘤嘤就是这个** / **算了，放过它吧** |
| 5. 踹飞 | 选择删除 | 踹文件动画，第 5 帧触发爆炸音效与爆炸动画，同时把文件移入回收站 |
| 6. 收尾 | 动画结束 | 雷欧登场 → 向右侧飞出屏幕 → 程序退出 |

选「算了，放过它吧」会跳过踹文件与爆炸，直接进入收尾。

## 🚀 运行

需要 Python 3.8+、`PyQt6`、`send2trash`，以及自备的 `assets/` 目录放在 `main.py` 同级：

```bash
pip install PyQt6 send2trash

# 手动狙击模式
python main.py

# 指定文件
python main.py "D:\某个\不要的文件.txt"
```

## 📦 打包 exe

```bash
pip install pyinstaller
pyinstaller --noconfirm --onefile --windowed --name MonsterDeleter ^
  --add-data "assets;assets" --hidden-import send2trash main.py
```

`--add-data` 会把 `assets/` 封进 exe，运行时由 `sys._MEIPASS` 解包定位（见代码里的 `get_resource_path()`）。产物在 `dist/MonsterDeleter.exe`。

## 🧹 卸载右键菜单

```powershell
reg delete "HKCU\Software\Classes\*\shell\SummonMonster" /f
```

## ⚠️ 已知问题

- **缺素材会卡死桌面**：`assets/` 不存在时动画帧为空，`animationFinished` 信号不再触发，流程停在第一阶段；但全屏透明遮罩（`WindowStaysOnTopHint`）已经淡到看不见却仍在最上层，鼠标点不到任何东西——只能去任务管理器结束 python 进程。
- **没有键盘退出**：当前版本只处理了鼠标点击，按 `Esc` 无效。
- 右键菜单里记录的是 exe / python 的**绝对路径**，移动程序后需要重新运行一次刷新。

## 📁 项目结构

```
MonsterDeleter/
├── main.py        # 全部逻辑：窗口、动画、音频、注册表、删除
└── .gitignore     # 忽略 exe、build/dist、assets
```

代码分了四个部件：`SpriteAnimator`（切分 spritesheet 逐帧播放）、`BubbleWidget`（自绘气泡）、`ChoicesWidget`（三个按钮）、`MonsterDeleter`（按阶段串起整场演出）。

## 📦 关于素材

`assets/` 包含 6 套 spritesheet、选择界面图与 3 个音频，合计约 26 MB（其中「爆炸」那张就有 13 MB）。仓库不收录，原因有三：体积、单文件过大，以及素材版权归属原项目。需要运行请从下方的上游仓库获取。

## 🙏 来源与致谢

本项目**基于 [531149627/MonsterDeleter](https://github.com/531149627/MonsterDeleter) 修改**，素材、动画与原始设计均来自该仓库（上游 446 ⭐ / 71 fork）。

上游未声明开源许可证，按默认规则保留所有权利。因此本仓库**仅作个人学习留档**，不提供打包程序、不做分发；如需使用请前往上游。

## 📄 依赖

| 依赖 | 用途 |
| --- | --- |
| `PyQt6` | 窗口、动画、`QtMultimedia` 音频播放 |
| `send2trash` | 把文件移入回收站 |
| `pyinstaller` | 打包单文件 exe（仅打包时需要） |
