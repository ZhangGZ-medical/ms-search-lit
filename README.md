# ms-search-lit

文献检索与引文管理技能，面向医学研究。**每条引用都必须经 API 验证，禁止凭记忆生成。**

## 能力

| 数据库 | 用途 |
|---|---|
| **PubMed** | 按 MeSH 词、布尔查询检索；取文章元数据；找相关文章；按题录反查；ID 互转 |
| **Semantic Scholar** | 语义检索、引文网络扩展 |
| **bioRxiv / medRxiv** | 预印本检索 |

## 结构

```
ms-search-lit/
├── SKILL.md                       # 工作流与反幻觉协议
├── skill.yml
└── references/
    ├── pubmed_eutils.sh           # NCBI E-utilities 封装（Bash）
    ├── parse_pubmed.py            # 响应解析：esearch / esummary / efetch / bibtex
    ├── snowball.py                # 引文网络滚雪球扩展（S2 Graph API）
    └── snowball_challenge/        # 离线确定性校验卡（fixture + verify.sh）
```

## 主通道与降级

优先使用 PubMed MCP 工具；仅当 MCP 不可用（session 超时、`No such tool available`）时，才降级到 `pubmed_eutils.sh`。**一旦降级，本会话后续全部走 E-utilities，不再重试 MCP。**

```bash
EUTILS="references/pubmed_eutils.sh"
PARSER="references/parse_pubmed.py"

bash "$EUTILS" search "ischemic stroke" 20 | python3 "$PARSER" esearch
bash "$EUTILS" fetch_json "16168343,16085191" | python3 "$PARSER" esummary
bash "$EUTILS" fetch "16168343" | python3 "$PARSER" efetch
bash "$EUTILS" cite_lookup "Bivariate analysis of sensitivity" | python3 "$PARSER" esearch
```

限速：无 API key 时 3 req/s，脚本内置 350 ms 间隔。

## 反幻觉协议（强制）

1. **绝不**凭记忆生成参考文献——每条必须来自 API 检索结果
2. **绝不**编造 DOI / PMID；查不到就标 `[UNVERIFIED - NEEDS MANUAL CHECK]`
3. 逐字段交叉核对：作者（首尾）、年份、期刊、标题（精确匹配）、卷期页码
4. 生成 BibTeX 时附 `verified` / `verified_by` / `verified_on` 字段

## 检索模式

- **Manuscript Reference Pool**：为原创研究返回 25–40 条候选，覆盖六大类别（背景、缺口定义、对照队列、方法学原始文献、报告规范、机制支撑）
- **Systematic Search**（PRISMA 合规）：记录检索式、日期、命中数、逐级筛选
- **Quick Cite** / **Related Papers**：单篇定位、从已知文献扩展

## 依赖

- Python 3.10+（stdlib only）
- 可选：`NCBI_API_KEY` 环境变量（限速提升至 10 req/s）
- 网络：需访问 `eutils.ncbi.nlm.nih.gov`

## 来源

属于 `medsci-skills` 技能套件（`/search-lit` 模块）。
