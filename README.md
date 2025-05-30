# 🧘‍♀️ AutoDingKe · 自动禅修定课工具

一个基于 macOS 的自动化工具，支持每天早晚定时启动 Zoom 会议、共享 Keynote 禅修幻灯片并自动结束会议，帮助主持人实现“无人值守”的定课流程。

---

## 🌱 项目背景

这是为“静心学堂”开发的自动化工具。我们每日早晚会进行线上禅修，由学员轮流担任主持人。但现实中，主持人偶尔会因事缺席或忘记操作，为了维持禅修的稳定性与连续性，我们开发了这个自动化工具。

---

## 🛠 功能

- ⏰ 自动唤醒电脑（通过 pmset 预设）
- 🔗 自动打开 Zoom 并加入指定会议
- 🖱 使用 `cliclick` 模拟鼠标操作（屏幕共享、勾选“共享声音”、选择窗口）
- 📽 自动打开并播放 Keynote 幻灯片
- 📴 播放结束后自动停止共享、结束会议、退出程序
- 🧘 完全“无人值守”，每日静心不间断

---

## 🔧 安装与准备

### 1️⃣ 安装 [`cliclick`](https://github.com/BlueM/cliclick)

```bash
brew install cliclick
```

### 2️⃣ 授权权限

打开系统设置：

- 系统设置 > 隐私与安全性 > **辅助功能**
- 系统设置 > 隐私与安全性 > **完全磁盘访问权限**

勾选以下程序：

- **Automator**
- **Terminal**
- **稍后你创建的 .app 文件（如 ZenAuto.app）**

---

## 🪄 操作流程

### 📜 第一步：编写/复制 AppleScript 脚本

你可以参考项目中的 `AutoDingKe.scpt`，或使用 Script Editor 编写一个如下结构的脚本：

1. 自动打开 Zoom 并加入会议  
2. 使用 `cliclick` 点击按钮（共享屏幕、共享声音、选择窗口、停止共享、结束会议等）  
3. 打开 Keynote 并自动播放课件  
4. 播放结束后自动停止共享、结束会议、退出 Keynote 和 Zoom  

请根据你的实际情况修改以下内容：

- Zoom 会议链接与密码  
- Keynote 文件路径  
- 所有点击操作的坐标（需要测试获取）

---

### 📍 第二步：获取正确的点击坐标

1. 如果尚未安装cliclick，在终端Terminal执行：
```bash
brew install cliclick
```
2. 在终端中运行下面的命令，它会每两秒打印一次当前鼠标位置：
```bash
while true; do /opt/homebrew/bin/cliclick p; sleep 2; done
```
按 ⌃C 停止监听。

3. 找到并记录坐标
移动鼠标 到你要点击的第一个按钮上（如 Zoom “共享屏幕”）。终端会输出类似：
```bash
p:743,853
```
继续移动鼠标 到下一个按钮（如“共享声音”），再记录输出的坐标。
将所有需要点击的按钮坐标整理成一张小表，方便后续脚本调用：

| 操作           | 鼠标坐标（x,y） |
| ------------ | --------- |
| Zoom 控制栏浮出区域 | 960,313   |
| 点击“共享屏幕”     | 743,853   |
| 勾选“共享声音”     | 1018,468  |
| 选择共享窗口缩略图    | 348,334   |
| 点击“Share”    | 736,772   |
| 停止共享         | 806,70    |
| 点击“结束会议”     | 744,464   |
| 确认“退出会议”     | 1344,805  |


根据记录下来的坐标，修改对应Script脚本里的点击动作坐标参数, 例如：
```applescript
-- Step 4: 点击“共享屏幕”按钮（请替换为你实际坐标）
do shell script "/usr/bin/env PATH=/opt/homebrew/bin:/usr/local/bin:$PATH cliclick c:743,853"
```

完成全部的修改替换后，可以直接在AppleScript运行测试效果，直至满足需求，可进行下一步封装。

---

### 💾 第三步：使用 Automator 封装创建 .app

1. 打开 **Automator.app**  
2. 选择 **“应用程序（Application）”**  
3. 从左侧库中拖入 **“运行 AppleScript（Run AppleScript）”** 操作  
4. 将你在 Script Editor 中测试通过的完整脚本粘贴进去  
5. 点击菜单栏 **文件 > 存储**  
6. 命名为 `ZenAuto.app`（或其他英文名称，以免路径有中文造成权限问题）  
7. 将 `ZenAuto.app` 添加到：

   - 系统设置 > 隐私与安全性 > **辅助功能**  
   - 系统设置 > 隐私与安全性 > **完全磁盘访问权限**

---

## ⏲ 第四步：设置定时运行

### ✅ 方法一：使用“日历”自动运行

1. 打开“日历”应用  
2. 创建每天早上 **7:15** 和晚上 **21:15** 的两个事件  
3. 在事件提醒中选择 **打开文件**  
4. 指定要打开的文件为你的 `ZenAuto.app`

### ✅ 方法二：使用 crontab 启动

打开终端，输入：

```bash
crontab -e
```

在编辑器中添加以下两行：

```bash
15 7 * * * open /Applications/ZenAuto.app
15 21 * * * open /Applications/ZenAuto.app
```

保存并退出。

---

## ⚡ 第五步：设置每天自动唤醒电脑

提前 5 分钟唤醒电脑，确保定课流程可以顺利开始。

打开终端输入：

```bash
sudo pmset repeat wakeorpoweron MTWRFSU 07:10:00
sudo pmset repeat wakeorpoweron MTWRFSU 21:10:00
```

> **注意事项**  
> - 该命令只能唤醒“睡眠”状态的 Mac，无法开机  
> - 请保证 Mac 一直插电  
> - 若需要更灵活地设置多个唤醒点，可结合 `pmset schedule` 与脚本动态更新  

---

## 🧯 常见问题

**Q: `cliclick` 无法运行？**  
- 确认已通过 Homebrew 安装  
- 命令中使用完整路径 `/opt/homebrew/bin/cliclick`  
- 已在系统设置中授权辅助功能和完全磁盘访问  

**Q: `.app` 无法执行 shell？**  
- Script Editor 导出的 `.app` 会被沙箱拦截，请使用 Automator 创建  
- 确保给 Automator 应用本身授权  

**Q: Zoom 共享声音失效？**  
- 确保勾选了“共享声音”选项  
- 尝试重启 Zoom 或更新到最新版本  

---


**Fiona Pan（静心学堂）**  
欢迎交流改进建议，也欢迎 Fork 本项目，打造你自己的“自动定课助手”🌱

---

## 📄 License

MIT License
