# 飞书文档代码抓取与运行记录

来源：https://my.feishu.cn/wiki/XJlawBeOOiBFStk7yAfcYYcXngk
文档标题：**12.13号圣诞树代码🎄V17.6版本**
文档 token：`TFkzdvKKxowu8cxJUWBcsEranVf`（wiki 空间 `7551390496917176348`）

## 怎么拿到的

1. 该文档对匿名访客开放（`isPermitted: true`），但正文不在 SSR 首屏里，需要执行页面 JS 才会渲染。
2. 正文实际藏在页面里的 `window.DATA = Object.assign({}, window.DATA, { clientVars: Object({...}) })` 中：
   `clientVars.data.block_map`（59 个块）+ `block_sequence`（块顺序）。
3. 按 `block_sequence` 顺序、用 `data.text.initialAttributedTexts.text` 拼回每个块的纯文本，
   唯一的 `type: "code"` 块（id `J2Ixd8Y0MouQQexUf8Vc8rMLnrd`，47593 字符）就是完整代码。
4. 文档其余部分（源提示词 / 功能介绍 / Win-Mac 运行流程 / 已知问题）见 `full_doc.txt`。

## 修复了 5 处被飞书吞掉的换行

飞书文本导出会把「行注释 `// ...` 结尾的那一行」的下一个换行吃掉，导致注释把下一行代码一起注释掉，
其中 `// Clean Database Name` 后面的 `const DB_NAME` 被打包进注释，直接导致整段 module 脚本语法错误（白屏/一直转圈）。

| 位置 | 修前 | 修后 |
| --- | --- | --- |
| DB 初始化 | `// Clean Database Nameconst DB_NAME = ...` | 拆成两行 |
| setupEvents | `// ==== V17.6.5: UI + Keyboard ====window.setupEvents = function() {` | 拆成两行 |
| 键盘控制 | `// --- Keyboard Controls ---window.addEventListener('keydown', ...)` | 拆成两行 |
| 键盘交互 | `// 键盘交互window.addEventListener('keydown', ...)` | 拆成两行 |
| 旋转逻辑 | `// Updated Rotation Logicif(manualRotateState.x!==0 ...)` | 拆成两行 |

修复后 `node --check` 通过（0 error），页面正常进入渲染。

## 怎么跑

```powershell
# 任意静态服务器都行，必须走 http（MediaPipe 取摄像头需要安全上下文，file:// 不给摄像头）
cd christmas_tree
python -m http.server 8123 --bind 127.0.0.1
# 浏览器打开
start http://127.0.0.1:8123/index.html
```

- 依赖外网：three@0.160（jsdelivr）、@mediapipe/tasks-vision@0.10.3（jsdelivr + googleapis 模型）、Google Fonts。
- 手势需要摄像头权限（距离 20–30cm、手倾斜约 45°、光线充足）。
- 键盘：`空格` 聚合 / `Z` 散开 / `X` 抓照片 / 方向键旋转；鼠标拖标题、滑块调密度与雪花。

## 已知坑（实测）

- 软件渲染（headless SwiftShader）下帧间隔 dt 会到 0.3s 以上，`position.lerp(target, 4*dt)` 的系数 >1，
  粒子会指数级飞散、树看不见。真机 GPU（本机 AMD Vega 8）60fps 无此问题；
  如果某台机器很卡，可在 `animate()` 里把 `dt` 卡一下：`dt = Math.min(dt, 0.05);`。
- 首次加载依赖 CDN，网络差会一直停在 `SYSTEM INITIALIZING`。

## 文件

- `index.html` —— 修好换行、可直接运行的完整单文件应用
- `raw_text_from_feishu.txt` —— 从飞书块里原样拼出的代码文本（未修复换行，留档）
- `full_doc.txt` —— 整篇文档的纯文本（含源提示词与排障说明）
- `shots/tree_mode2.png` —— 聚合态（圣诞树）实跑截图
- `shots/scatter_mode.png` —— 散开态（星云）实跑截图
