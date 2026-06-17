---
name: 职教资讯
description: 职教资讯搜索技能 — 基于17类信息源定向检索职业教育政策、动态、案例，支持速览/主题双模式输出，可生成Word报告。
---

# 职教资讯搜索技能

当用户调用此技能查询职教相关资讯时，基于《职教AI知识库-信息源网址与栏目映射》中的17类信息源，定向检索并输出结果。

## 触发条件

用户调用 `/职教资讯` 或明确询问职教领域相关资讯、政策、动态、案例等信息时触发。

## 核心流程

### Step 1: 确认时间范围
用户会在提问时告知需要"周"还是"月"的数据：
- **周** = 最近 7 天（近一周）
- **月** = 最近 30 天（近一月）

如用户未明确，主动询问需要周还是月的数据。

**⚠️ 日期严格过滤规则（必须执行）**：
- 搜索结果汇总后，**必须逐条检查日期**，剔除超出时间范围的条目
- "最近一周" = 当前日期往前推 7 天，**严格剔除 7 天之前的信息**
- 搜索词可以宽泛（如"2026年6月"），但结果必须手动过滤到目标窗口
- **禁止将数周前旧闻混入"最近一周"报告**
- 过滤后条目数不足时，标注「本周该源无新资讯」而非用旧闻填充

### Step 2: 意图匹配
根据用户问题，匹配下方快速检索索引，确定优先检索的信息源：

| 你想了解什么 | 优先检索网址 |
|-------------|-------------|
| 国内职教政策法规 | moe.gov.cn, mohrss.gov.cn, cvae.com.cn, civte.edu.cn |
| 职教统计数据 | moe.gov.cn（文献/统计公报）, mohrss.gov.cn |
| 产教融合 / 校企合作 | ndrc.gov.cn, tech.net.cn, vcsc.org.cn |
| 国际职教趋势 | unevoc.unesco.org, oecd.org, worldbank.org |
| 职教技能竞赛 | worldskills.org, vcsc.org.cn |
| 欧洲职教体系 | bibb.de, cedefop.europa.eu |
| 德国双元制 | bibb.de |
| 劳动力与就业技能 | worldbank.org, ilo.org |
| 高职院校实践案例 | tech.net.cn, eol.cn, **szpu.edu.cn, wxit.edu.cn, jhc.edu.cn, ntust.edu.tw, ntut.edu.tw, yuntech.edu.tw（6所全部必搜）** |
| 职教社会影响力报道 | 人民网/央视网/光明网职教专栏 |
| 职业技能标准与资格 | mohrss.gov.cn, worldskills.org |

### Step 2.5: 必检信息源清单（根据查询类型强制执行）

**⚠️ 以下信息源在对应查询类型中必须逐站检索，不得遗漏！仅靠聚合站（如 tech.net.cn）的转载报道不够，必须直接搜索各院校/机构官网一手信息。**

| 查询类型 | 必检信息源（每个都必须单独 `site:` 搜索） |
|---------|------------------------------------------|
| 国内教育/职教政策 | moe.gov.cn, mohrss.gov.cn, ndrc.gov.cn |
| 高职院校动态/案例 | tech.net.cn, szpu.edu.cn, wxit.edu.cn, jhc.edu.cn, ntust.edu.tw, ntut.edu.tw, yuntech.edu.tw |
| 技能竞赛/培训 | mohrss.gov.cn, worldskills.org, vcsc.org.cn |
| 国际职教趋势 | unevoc.unesco.org, oecd.org, worldbank.org, bibb.de |
| 职教社会影响/评论 | 人民网, 央视网, 光明网 |

**🚨 六所重点院校官网（信息源 #16）检索规则：**
- 每次查询涉及"高职院校""实践案例""院校动态""产教融合案例"时，**必须全部单独搜索**以下6个站点，不得跳过或只搜其中几所：
  - `site:szpu.edu.cn 新闻 动态`（深圳职业技术大学）
  - `site:wxit.edu.cn 新闻 动态`（无锡职业技术大学）
  - `site:jhc.edu.cn 新闻 动态`（金华职业技术大学）
  - `site:ntust.edu.tw 新聞 動態`（台湾科技大学）
  - `site:ntut.edu.tw 新聞 動態`（台北科技大学）
  - `site:yuntech.edu.tw 新聞 動態`（云林科技大学）

### Step 3: 执行定向搜索
使用 WebSearch 工具，对匹配的信息源进行定向搜索。搜索策略：

1. **国内网站搜索**：使用 `site:` 限定符，如 `site:moe.gov.cn 职业教育 政策`
2. **国际网站搜索**：使用英文关键词 + site 限定，如 `site:unevoc.unesco.org TVET policy 2026`
3. **六所重点院校**：每次涉及院校动态的查询，必须对 6 所院校逐站搜索（见 Step 2.5），不可仅依赖 tech.net.cn 转载
4. **综合搜索**：如涉及多源，同时发起多个搜索并行执行
5. **时间限定**：搜索词中包含时间关键词（如"2026年6月"），同时利用 WebSearch 返回结果中的时效性信息

每次搜索时，应在 query 中加入时间范围关键词以获取最新结果。

### Step 3.5: URL 校验与修正（必须执行）

**⚠️ 搜索引擎返回的 URL 可能指向已废弃的子域名或缓存页面，生成文件前必须校验！**

1. **校验方法**：对每条结果的 URL，用 `curl -sL -o /dev/null -w "%{http_code}" --max-time 10` 验证 HTTP 状态码
2. **常见死域名映射**（持续更新）：
   | 死域名 | 正确域名 |
   |--------|---------|
   | `old.tech.net.cn` | `www.tech.net.cn` |
   | `www2.tech.net.cn` | `www.tech.net.cn` |
3. **修正策略**：
   - 返回 000（连接失败）或 4xx/5xx → 尝试主域名 + 相同路径
   - 主域名验证通过（200）→ 使用修正后的 URL
   - 仍不可达 → 标记为「链接已失效」并在摘要中保留原文信息
4. **禁止行为**：绝不直接使用搜索引擎返回的原始 URL 而不校验，尤其是带有 `old.`、`www2.` 等非标准子域名的链接

### Step 4: 结果整理（两种输出模式）

用户可选择以下两种输出模式之一。如用户未明确指定，默认使用**速览模式**在对话中呈现，生成文件时自动使用**主题模式**。

---

#### 模式 A：速览模式（默认对话输出）

**适用场景**：用户说"速览""逐条""每条一句话"时使用。简洁高效，按信息源分组，每条一行。

**输出格式**：

```
### 信息源名称 (domain) — X条

| # | 标题 | 日期 | 一句话 |
|---|------|------|--------|
| 1 | xxx | 06.15 | 核心要点一句话概括（不超过40字） |
| 2 | xxx | 06.12 | 核心要点一句话概括 |
```

**速览规则**：
- 每条摘要压缩为**一句话（≤40字）**，只保留最核心事实
- 按信息源分组排列（教育部 → 人社部 → 人民日报 → tech.net.cn → 六校 → 其他）
- 末尾附总计行：`📊 X条资讯 | 覆盖 Y 个信息源 | 六校覆盖情况`
- **每条必须附可点击来源链接**

**速览示例**：
```
### 教育部 (moe.gov.cn) — 4条

| # | 标题 | 日期 | 一句话 |
|---|------|------|--------|
| 1 | 国务院审议通过《教育发展十五五规划》 | 06.11 | 李强主持，教育优先发展、健全学龄人口适配机制 |
| 2 | 习近平《一体推进教育科技人才发展》发表 | 06.16 | 统筹职教、高教、继教，职普融通、产教融合 |

📊 共28条资讯 | 覆盖10+信息源 | 六校：4官网一手+2媒体转载
```

---

#### 模式 B：主题模式（生成文件时默认）

**适用场景**：用户说"按主题""分类汇总""输出Word"时使用。深度摘要，按主题归类。

**输出格式**：

每个主题包含：
- **主题标题**（含颜色标识）
- 每条资讯：▶ 标题 + 📅 日期 + 来源 + 🔗 查看原文 + 摘要段落（100-200字深度概述）

**主题分类参考**：
| 主题 | 适用范围 |
|------|---------|
| 🏛️ 顶层政策与战略部署 | 国务院/教育部重大政策、规划、领导人讲话 |
| ⚖️ 职业教育法执法检查 | 人大执法检查、法律实施监督 |
| 🏭 产教融合与实训实践 | 产教共同体、校企合作、实训基地 |
| 🔵 课堂教学模式创新 | 课堂改革、教学方法、数智化教学 |
| 🏫 六所重点院校动态 | 六校官网一手资讯 |
| 🏆 世界技能大赛与技能培训 | 世赛备战、技能培训行动 |
| 🌍 国际合作与舆论宣传 | 国际交流、教育宣传会议 |

可根据实际资讯内容**动态增减或合并主题**，不强制使用全部主题。

---

#### 模式选择规则

| 用户表述 | 使用模式 |
|---------|---------|
| "速览""逐条""简单列一下""每条一句话" | 模式 A（速览） |
| "按主题""分类汇总""详细一点""输出Word" | 模式 B（主题） |
| 未明确指定 | 对话中用模式 A，生成文件时用模式 B |

### Step 5: 生成文件（当用户要求时）

当用户要求生成 Word/HTML 文件时：

1. **HTML**：所有链接使用 `<a href="..." target="_blank">` 标签，确保点击可打开
2. **Word**：**必须使用 python-docx 的 OxmlElement 创建可点击超链接**，严禁将 URL 输出为纯文本。实现方式：
   - 通过 `part.relate_to(url, RT.HYPERLINK, is_external=True)` 创建外部链接关系
   - 用 `OxmlElement('w:hyperlink')` + `qn('r:id')` 绑定链接
   - 给超链接文字设置蓝色 (#0563C1) 和下划线，字号8pt
   - 显示文字为「🔗 查看原文」而非裸 URL
3. **字体规范**：Word 文档必须设置中文字体，**禁止使用纯西文字体（如 Arial）**。所有 run 必须通过 `w:rFonts` 同时设置 `w:ascii`/`w:hAnsi`（西文=Calibri）和 `w:eastAsia`（中文=微软雅黑）。实现方式：
   ```python
   from docx.oxml.ns import qn
   from docx.oxml.shared import OxmlElement
   def set_run_font(run, ascii_font='Calibri', ea_font='微软雅黑', size=Pt(11), bold=False, color=None):
       run.font.size = size
       run.bold = bold
       run.font.name = ascii_font
       if color: run.font.color.rgb = color
       rPr = run._r.get_or_add_rPr()
       rFonts = rPr.find(qn('w:rFonts'))
       if rFonts is None:
           rFonts = OxmlElement('w:rFonts')
           rPr.insert(0, rFonts)
       rFonts.set(qn('w:ascii'), ascii_font)
       rFonts.set(qn('w:hAnsi'), ascii_font)
       rFonts.set(qn('w:eastAsia'), ea_font)
   ```
4. **所有文件**中的每条资讯都必须包含可点击的原文链接

---

## 信息源完整目录（17类）

### 一、近期动态与政策关注（国内官方）

| # | 信息源 | 网址 | 检索栏目 |
|---|--------|------|---------|
| 1 | 教育部官网 | http://www.moe.gov.cn | 新闻、公开、文献（统计公报） |
| 2 | 人社部官网 | https://www.mohrss.gov.cn/ | 新闻中心、政务公开、政策法规 |
| 3 | 中国职业技术教育网 | https://www.cvae.com.cn/ | 要闻、政策法规、学会工作、科研教研 |
| 4 | 国家发改委社会发展司 | https://www.ndrc.gov.cn/fzggw/jgsj/shs/ | 新闻动态、政务公开 |
| 5 | 教育部职业教育发展中心 | https://www.civte.edu.cn/ | 工作动态、决策服务、学术研究 |

### 二、国际前沿

| # | 信息源 | 网址 | 检索范围 |
|---|--------|------|---------|
| 6 | UNESCO-UNEVOC | https://unevoc.unesco.org/home/ | 全站 |
| 7 | OECD - 教育与技能 | https://www.oecd.org/ | Topics → Education and skills |
| 8 | 世界银行 - 教育与就业 | https://www.worldbank.org/ | What We Do, Research & Publications |
| 9 | 世界技能组织 | https://worldskills.org/ | 全站 |
| 10 | 德国 BIBB | https://www.bibb.de/en/index.php | 全站 |
| 11 | 欧洲 Cedefop | https://www.cedefop.europa.eu/en | Themes, Publications, News, Countries |
| 12 | 国际劳工组织 ILO | https://www.ilo.org/ | Research, Data, Standards |

### 三、实践案例

| # | 信息源 | 网址 | 检索栏目 |
|---|--------|------|---------|
| 13 | 现代高等职业技术教育网 | https://www.tech.net.cn/ | 院校动态、教学改革、校际合作 |
| 14 | 中国教育在线 - 职教频道 | https://www.eol.cn/ | 职教专题库、院校动态 |
| 15 | 世界职业院校技能大赛官网 | https://www.vcsc.org.cn/ | 赛事动态、获奖案例 |
| 16 | 重点职业院校（6所） | szpu.edu.cn, wxit.edu.cn, jhc.edu.cn, ntust.edu.tw, ntut.edu.tw, yuntech.edu.tw | 新闻动态、学校概况 |
| 17 | 主流媒体职教专栏 | 人民网/央视网/光明网 | 职业教育/职教强国专栏 |

---

## 使用示例

**用户**：`/职教资讯 最近一月国内职教政策有什么新动态？`

**执行**：
1. 时间范围：月（30天）
2. 意图：国内职教政策法规 → moe.gov.cn, mohrss.gov.cn, civte.edu.cn
3. 并行搜索（4源）：site:moe.gov.cn 职业教育 政策 / site:mohrss.gov.cn 职业技能 政策 / site:civte.edu.cn 工作动态 / site:ndrc.gov.cn 产教融合
4. 整理输出含标题、摘要、来源、日期的结构化结果

**用户**：`/职教资讯 最近一周教育新闻动态汇总`

**执行**：
1. 时间范围：周（7天）
2. 意图：综合教育动态 → 多源覆盖
3. 并行搜索（至少12源）：
   - 政策线：site:moe.gov.cn 教育 新闻 / site:mohrss.gov.cn 职业技能 / 人民网 教育
   - 案例线：site:tech.net.cn 高职 院校 / site:eol.cn 职教
   - **六校必搜**：site:szpu.edu.cn 新闻 / site:wxit.edu.cn 新闻 / site:jhc.edu.cn 新闻 / site:ntust.edu.tw 新聞 / site:ntut.edu.tw 新聞 / site:yuntech.edu.tw 新聞
4. 校验所有 URL + 修复死链
5. 整理输出

**用户**：`/职教资讯 最近一周国际职教有什么趋势？`

**执行**：
1. 时间范围：周（7天）
2. 意图：国际职教趋势 → unevoc.unesco.org, oecd.org, worldbank.org
3. 并行搜索英文关键词
4. 整理输出
