# 职教资讯技能包

基于《职教AI知识库-信息源网址与栏目映射》构建的职业教育新闻智能检索与报告生成技能。

## 适用平台

本技能为 Claude Code Skill 格式，可适配到支持 Skill 机制的 AI 编程助手平台。

## 文件结构

```
职教资讯技能包/
├── README.md                          # 本文件
├── 技能生成过程说明.md                 # 技能迭代演进文档
├── skill/
│   └── 职教资讯.md                     # 技能定义文件（可直接上传）
├── scripts/
│   ├── generate_quickview_word.py      # 速览模式 Word 生成脚本
│   ├── generate_thematic_word.py       # 主题模式 Word 生成脚本
│   └── requirements.txt               # Python 依赖
├── reference/
│   └── 职教AI知识库-信息源网址与栏目映射.md  # 17类信息源映射表
└── examples/
    ├── education_news_quickview.docx    # 速览模式输出示例
    └── education_news_thematic.docx     # 主题模式输出示例
```

## 快速使用

### 1. 安装技能

将 `skill/职教资讯.md` 上传到目标平台的技能目录中。

### 2. 安装 Python 依赖

```bash
pip install -r scripts/requirements.txt
```

### 3. 调用方式

在对话中输入：

```
/职教资讯 最近一周教育新闻动态汇总，速览模式导出word
/职教资讯 最近一月产教融合有什么新动态
/职教资讯 最近一周国际职教趋势
```

### 4. 生成脚本

从技能包根目录运行：

```bash
cd 职教资讯技能包

# 速览模式（表格式，按信息源分组，一句话速览）
python scripts/generate_quickview_word.py

# 主题模式（章节式，按主题归类，深度摘要）
python scripts/generate_thematic_word.py

# 输出文件在 examples/ 目录下
```

注意：脚本内的数据为示例（2026年6月10-16日），实际使用时需替换为实时检索结果。

## 核心能力

| 能力 | 说明 |
|------|------|
| **17类信息源覆盖** | 教育部、人社部、人民日报、tech.net.cn、eol.cn、6所重点院校等 |
| **智能意图匹配** | 根据用户问题自动匹配优先检索的信息源 |
| **双模式输出** | 速览（表格式一句话）+ 主题（章节式深度摘要） |
| **URL 校验** | 生成文件前 curl 逐条验证链接可达性，自动修复死域名 |
| **日期严格过滤** | 搜索结果按时间窗口精确过滤，杜绝旧闻混入 |
| **可点击链接** | Word 文件中所有来源链接均为可点击超链接 |
| **中文字体** | 西文 Calibri + 中文微软雅黑双字体，排版规范 |

## 信息源覆盖状态

| 状态 | 数量 | 说明 |
|------|:--:|------|
| ✅ 可检索 | 14 | 搜索引擎 site: + 中文名补搜 |
| 🟡 可爬取但更新慢 | 1 | civte.edu.cn（curl 直爬） |
| 🔴 无法爬取 | 1 | cvae.com.cn（Vue SPA 需无头浏览器） |
| — | 1 | 国际源（按需启用） |

## 技能版本

v3.0 — 2026-06-17
