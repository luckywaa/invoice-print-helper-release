# 发票打印助手

浏览器本地运行的**发票拼版打印工具**：批量识别发票 PDF/OFD，自动拼版到 A4 打印，并按报销科目分类统计。

**双击 `index.html` 即可使用**（推荐 Edge / Chrome）。无需安装、无需服务器、无需联网——所有识别与拼版都在你自己的电脑上完成，**发票文件不会被上传到任何地方**。

> 完整使用文档见压缩包内的**《使用说明.md》**。

## 下载使用

1. 进入 [公开发布仓 Releases](https://github.com/luckywaa/invoice-print-helper-release/releases) 页面，下载最新版压缩包 `invoice-print-helper-vX.Y.Z.zip`
2. 解压到任意位置
3. 双击 `index.html`，用 **Edge / Chrome** 打开
4. 把发票文件（或整个文件夹）拖进页面即可

整个文件夹就是完整应用，可以拷给同事、换电脑使用。

## 功能特性

### 导入与识别

- 拖入文件或**整个文件夹**（PDF / OFD，自动跳过 XML 副本），数百个文件的归档批量导入不卡顿
- 自动识别票种：**数电普票**（含老版增值税电子普票）、**数电专票**（含老版）、**铁路电子客票**、**航空行程单**
- 自动提取票面金额（乱序文字流、中文大写金额、OFD 标签数值分层、逐字切分版式等真实票面均已适配）与**发票号**
- **同发票号自动去重**（PDF 版优先于 OFD），重复件收进"垃圾箱"，可预览、可恢复
- 带电子章（Stamp 注释）的数电票自动**矢量烘焙**进拼版成品——章不丢、文字仍是矢量
- 水印画在裁剪框外的票面（如部分网约车发票）按 CropBox 精确裁剪嵌入，不把水印带进成品

### 拼版打印（四个打印任务）

1. **智能拼版**：普通发票 2 张/A4、3 张/A4 或智能分析；铁路客票行式布局自动排布（竖放统一右旋 90°）
2. **专票副本**：数电专票自动额外生成一份，页顶印红字"进项抵扣专用，单独提交财务"
3. **纸质凭据粘贴单**：内嵌单位粘贴单模板，可选张数直接打印
4. **报销贴票说明 / 凭据打印双面版 docx** 查看、下载

其他拼版能力：

- 页边距可调（默认上/左 25mm 装订边、下/右 8mm），所有模式统一生效
- 每页左侧装订边自动竖排"**合计金额￥xx.xx元，请在背面签字**"（该页票面金额合计，右旋 90° 居中）
- 切割虚线**默认不打印**，需要时在设置区勾选开启
- 页边标注文字可整体关闭；所有设置随浏览器保存，可导出/导入 JSON

### 报销统计

- 自动按 **火车 / 机票 / 大巴 / 住宿 / 市内交通 / 其他** 六类统计金额与小计
- 支持导出 CSV 明细；打印/预览时可一键**自动整理**按报销顺序排序（火车 → 机票 → 大巴 → 住宿 → 市内交通 → 其他）

## 快速开始

1. [下载](https://github.com/luckywaa/invoice-print-helper-release/releases)最新版压缩包并解压（或下载本仓库）
2. 双击 `index.html`，用 Edge / Chrome 打开
3. 把发票文件或整个文件夹拖进页面即可

整个文件夹就是完整的应用，可以拷给同事、换电脑使用，不需要任何安装步骤。

## 隐私说明

- **纯本地运行**：不联网、无后端、无统计埋点，发票数据不出你的电脑
- 识别、去重、拼版、生成 PDF 全部在浏览器内存中完成
- 设置只保存在浏览器 localStorage，可随时导出/清空

## 目录结构

```
├─ index.html            入口（双击打开）
├─ css/app.css           样式
├─ js/
│  ├─ detect.js          票种识别 / 金额提取 / 科目分类（纯函数，双环境）
│  ├─ ofd.js             OFD 解析 + Canvas 渲染（自研，无第三方 OFD 库）
│  ├─ compose.js         拼版引擎：plan() 纯逻辑 + build() pdf-lib 合成
│  ├─ app.js             主程序（状态、列表 UI、预览、打印流程）
│  └─ vendor/            三方库与内嵌数据（见下方"第三方组件"）
├─ assets/               粘贴单模板 PDF、报销贴票说明、凭据打印双面版 docx
├─ test/                 Node 测试与开发工具（见下方"开发与测试"）
└─ 使用说明.md            用户文档
```

## 开发与测试

无构建步骤、无 npm 依赖，任意文本编辑器 + Node 即可开发。代码风格偏 ES5、经典 `<script>` 顺序加载；除 app.js 外均为浏览器/Node 双环境通用。

```bash
node test/smoke.js            # 单元测试：票种识别 + 金额解析 + 拼版规划 + pdf-lib 合成管线
node test/dump.js <文件>      # 开发工具：打印样例 PDF 的文字层，用于设计/校验识别规则
node test/ofd-dump.js <文件>  # 开发工具：查看 OFD 内部结构
node test/build-assets.js     # 重新生成 cmaps/fonts 内嵌数据（更新 js/vendor 下原始资源后必须跑）
```

**关于样例票据**：真实发票包含个人信息，**不入库**。端到端测试（`test/e2e-samples.js` + `样例发票/` 目录）属于本地私有资产，已被 `.gitignore` 排除；如需完整回归，请自行准备票据目录与预期值表。

## 第三方组件

本项目在 `file://` 协议下离线运行，因此以下组件全部内嵌于 `js/vendor/`：

| 组件 | 许可证 | 用途 |
|---|---|---|
| [pdf.js](https://github.com/mozilla/pdf.js) | Apache License 2.0 | PDF 文字层提取与预览 |
| [pdf-lib](https://github.com/Hopding/pdf-lib) | MIT | 拼版 PDF 合成 |
| [pako](https://github.com/nodeca/pako) | MIT | OFD（ZIP）解压 |
| [@pdf-lib/fontkit](https://github.com/Hopding/pdf-lib-fontkit) | MIT | 自定义字体嵌入 |
| [Noto Sans SC](https://fonts.google.com/noto/specimen/Noto+Sans+SC)（字形子集） | SIL Open Font License 1.1 | 页边标注文字 |

## 分支与发版（GitHub 双仓库）

源码与发布分离，**外部只能看到发布的安装包，看不到源码**：

- **私有源码仓**（本仓库）：`master` 仅稳定版本，`dev` 日常开发
- **公开发布仓** `invoice-print-helper-release`：只有 README（本文件发版时自动同步过去）和 Release 压缩包
- 发版流程：dev 测试通过 → 合并到 master → [CHANGELOG.md](CHANGELOG.md) 写更新日志 → 打 tag（`vX.Y.Z`）并推送——GitHub Actions 自动构建 zip、在公开发布仓创建 Release 上传资产、并把本 README 同步为公开发布仓的 README（需预先配置 `RELEASE_TOKEN` 密钥，见 `.github/workflows/release.yml`）

本项目代码目前未声明独立开源许可证（保留所有权利）；如需正式开源请自行添加 LICENSE 文件。
