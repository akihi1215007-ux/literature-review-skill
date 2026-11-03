# literature-review

科研文献检索、精读、引用核对与证据综合技能，版本 **2.0.2**。

[English](README.en.md) · [技能入口](skills/literature-review/SKILL.md) · [命令参考](skills/literature-review/references/search_and_citation.md)

更新记录见 [CHANGELOG.md](CHANGELOG.md)。2026-11-03本地54项离线测试通过。

普通查找、精读和核引可以直接交付文献表、阅读卡或书目。需要维护多源记录、增量更新或系统/范围综述时，再使用文献库工具。

## 能做什么

- 按问题检索文献，核对身份，记录来源、检索日期和覆盖边界。
- 阅读单篇或比较多篇论文，区分摘要与全文，保留证据定位、条件和局限。
- 核对 DOI、元数据及 BibTeX；把“文献存在”和“支持具体主张”分别判断。
- 离线合并多源记录，维护稳定编号、来源观察和版本关系，记录证据与筛选决定。
- 按需导出书目、证据表、阅读地图和检索日志。

检索结果不等于完整覆盖；脚本不自动确认主张支持、全文身份、撤稿现态或研究数量，也不自动执行 meta-analysis。

## 安装

运行脚本需要 **Python 3.10+**；阅读与写作指导本身无需安装 Python 依赖。克隆你当前所在的仓库，或选择 Code → Download ZIP 后解压。

把 `skills/literature-review` **整个目录**复制到用户级 `~/.agents/skills/`（Windows：`%USERPROFILE%\.agents\skills\`），或项目级 `.agents/skills/`。已有旧版 `$CODEX_HOME/skills` / `~/.codex/skills` 布局时，可沿用客户端实际识别的位置；二选一，不重复注册。若已有同名技能，先备份再替换。根据[官方技能文档](https://learn.chatgpt.com/docs/build-skills)，变化可自动发现；未显示时再重启客户端。

Crossref/arXiv 检索和离线文献库工具只用 Python 标准库。使用 OpenAlex、PubMed 或其余引用客户端时，在**仓库根目录**安装可选依赖：

```sh
python -m pip install -r requirements.txt
```

独立安装目录也保留同一份 `requirements.txt`；不必为普通阅读任务安装网络客户端。

采用兼容技能格式的其他客户端请参照其安装说明；本项目未单独验证它们的集成。

## 使用方式

安装后，可直接提出任务，例如：

> 用 literature-review 查找主动降噪的近期研究，说明检索范围和每篇的相关性。

> 精读这篇论文，梳理方法、关键公式、证据位置和局限；标明当前拿到的是摘要还是全文。

> 核对这份参考文献的 DOI 和元数据，单独列出仍需阅读原文判断的主张。

### 命令示例

以下命令从**仓库根目录**进入技能目录，后续相对路径都以技能目录为准。输出使用新目录；示例检索需要网络。

```sh
cd skills/literature-review
python -c "from pathlib import Path; Path('project').mkdir(exist_ok=True)"
python scripts/search_sources.py crossref "active noise control" --limit 20 -o project/crossref.json
python scripts/search_sources.py arxiv "all:acoustic AND all:control" --limit 20 -o project/arxiv.json
python scripts/literature_pipeline.py merge --input project/crossref.json --input project/arxiv.json --output project/library-v1.json
python scripts/literature_pipeline.py export --library project/library-v1.json --output-dir project/delivery --only references.bib review-map.md
```

Crossref/arXiv 每次只取一页，`--limit` 为 1–100，结果始终标记 `complete: false`。它们用于小规模发现，不提供全量检索承诺。仅查找文献时可在检索后直接整理结果，不必执行 merge/export。

需要证据记录时，从库中读取真实 `record_id` 和 `sources.observation_id`，按[数据合同](skills/literature-review/references/library_schema.md)创建 `project/evidence-items.json`，再运行：

```sh
python scripts/literature_pipeline.py evidence --library project/library-v1.json --input project/evidence-items.json --output project/library-v2.json
python scripts/literature_pipeline.py export --library project/library-v2.json --output-dir project/evidence-delivery --only evidence.csv references.bib
```

证据输入支持单对象或数组；全文证据需实际本地文件和定位，状态默认 `unresolved`。`export` 不带 `--only` 时导出六个文件：`records.csv`、`evidence.csv`、`screening.csv`、`review-map.md`、`references.bib`、`search-log.json`。默认拒绝覆盖；确需替换时显式使用 `--overwrite`。

无需联网的合成数据示例见[离线工作流](skills/literature-review/references/example_workflow.md)。

### 保留的引用工具

`scripts/citation_tools/` 保留五个 CLI：

| 工具 | 用途 |
| --- | --- |
| `search_openalex.py` | OpenAlex 检索 |
| `search_pubmed.py` | PubMed 检索 |
| `extract_metadata.py` | 按标识符获取元数据 |
| `doi_to_bibtex.py` | DOI 转 BibTeX |
| `validate_citations.py` | BibTeX 检查，可选联网核 DOI |

兼容入口 `scripts/verify_citations.py` 与 `scripts/search_databases.py` 也予以保留；后者仅转换本地记录，不联网检索。完整参数与来源边界见[命令参考](skills/literature-review/references/search_and_citation.md)，或运行对应脚本的 `--help`。

## 目录与验证

```text
.
├── README.md / README.en.md
├── LICENSE
├── CHANGELOG.md
├── requirements.txt
├── .github/workflows/test.yml
└── skills/literature-review/
    ├── SKILL.md
    ├── THIRD_PARTY_NOTICES.md
    ├── scripts/
    ├── references/
    ├── assets/
    └── tests/
```

在**仓库根目录**运行离线测试：

```sh
python -m unittest discover -s skills/literature-review/tests -v
```

本轮新增引用回归覆盖损坏尾条目、括号/嵌套引号、注释、不可读稿件、常用引用命令和书籍作者/编者规则；支持子集及显式拒绝的语法见命令参考。GitHub Actions 配置执行同类测试。离线测试不证明外部服务实时可用，也不证明文献结论正确；具体运行状态以实际输出和 Actions 页面为准。

## 许可与归属

本项目按 [MIT License](LICENSE) 发布，保留 K-Dense Inc. 等上游贡献者的归属。复用代码、独立改写的方法与来源说明见 [THIRD_PARTY_NOTICES.md](skills/literature-review/THIRD_PARTY_NOTICES.md)；再分发时请保留相应声明。
