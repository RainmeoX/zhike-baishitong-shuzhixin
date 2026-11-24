# 珠科百事通 —— 校务生活 RAG 智能问答平台

> 2026 年"数智新技艺"作品遴选 · 自主研发应用类参赛作品
>
> 给珠科 3.1 万在校生的校务问答助手：选课规则、报修电话、校巴时刻、奖助政策——一句话问，3 秒答，答案必带官方出处。

## 基本信息

| 项 | 内容 |
|---|---|
| 作品名称 | 珠科百事通 |
| 参赛类别 | 自主研发应用 |
| 作者 | 赵泽宇、汪诚睿、区志成（珠海科技学院 · 电子信息学院）|
| 作品仓库 | https://github.com/RainmeoX/zhike-baishitong-shuzhixin |

## 文档

| 文档 | 说明 |
|---|---|
| `docs/珠科百事通项目方案书.docx` | 选题论证、架构、功能清单、时间线 |

## 项目结构

```
crawler/      数据层：教务处/后勤处/招生办通知爬虫
retrieval/    检索层：FTS5 关键词路 + 语义向量路 + RRF 融合 + 重排 + 时效降权
api/          生成层：FastAPI 问答服务 + 引用强制校验
miniapp/      应用层：微信小程序学生端 — 已搭骨架（首页搜索 + 结果页）
data/         爬取数据落地（原始副本不入库）
evaluation/   评测集：100 道真实学生问答
docs/         技术文档
web/          网页测试台
```

## 团队

三名作者协作完成数据采集链路、混合检索、问答服务、Web 与小程序端、评测集的设计与实现；开发过程使用 AI 工具辅助编码，全程留痕可查。

## 研发节奏（作品版本 v2.1）

| 阶段 | 时间 | 里程碑 |
|---|---|---|
| 冲刺 1 | 10-02 ~ 10-05 | 开发环境搭建、爬虫跑通、数据入库 |
| 冲刺 2 | 10-06 ~ 10-12 | 检索双路 + 生成层 + 小程序骨架 |
| 冲刺 3 | 10-13 ~ 10-19 | 联调 + 100 问评测 ≥ 85% |
| 提交冲刺 | 10-20 ~ 10-23 | 每日提交迭代，10-23 提交 |

## 快速开始

环境：Python 3.12+（依赖见 `requirements.txt`）

```bash
# 方式一：一键启动（推荐）
python start.py

# 方式二：分步运行
python -m pip install -r requirements.txt          # 装依赖
python -m crawler.run --site jiaowu                 # 爬教务处
python -m crawler.run --site houqin                 # 爬后勤处
python -m crawler.run --site zsb                    # 爬招生办
python -m retrieval.index                           # 数据入库
python -m uvicorn api.main:app --port 8000          # 启 API
```

### Docker 部署（评委一键启动）

```bash
# 构建镜像
docker build -t zhike-baishitong .

# 运行（data/ 运行时挂载，密钥经 .env 注入）
docker run -p 8000:8000 --env-file .env -v ./data:/app/data zhike-baishitong
```

- 镜像基于 `python:3.12-slim`，内置 HEALTHCHECK（每 30s 探测 `/health`），`docker ps` 可见 healthy 状态
- `data/` 通过 `-v` 挂载进容器，首次启动 `start.py` 自动建库，已有数据可直接复用
- 大模型与 embedding 密钥通过 `--env-file .env` 注入，勿提交 `.env`

测试：

```bash
# 健康检查
curl http://127.0.0.1:8000/health

# 问答
curl -X POST http://127.0.0.1:8000/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"四六级考试什么时候报名","topk":5}'

# 评测（100 问）
python -m evaluation.runner --api http://127.0.0.1:8000
```

## 当前状态（v2.1）

| 层 | 状态 | 说明 |
|---|---|---|
| 数据层 | ✅ 254 篇 | 四个部门官方通知入库，条条可回溯原网页 |
| 检索层 | ✅ 混合路 | FTS5 关键词 + LIKE 中文子串 + 同义词扩展 + bge-m3 语义路 + RRF 融合 + rerank 精排 + 年份时效降权 |
| 生成层 | ✅ 生成式 | 只依据原文作答，引用强制校验，无出处即兜底；模型不可用时降级抽取式 |
| 应用层 | ✅ Web 三界面 + 小程序骨架 | Web 门户/智能问答/通知速览已落地；小程序两页骨架、接口已通，未上架 |
| 评测 | ✅ 100 问 | Hit Rate 87.1%、MRR 0.8357、拒答成功率 86.7% |

## 开发约定

- **全部核心模块由三名作者自主设计与实现**：数据采集管线、混合检索算法、问答服务、Web 与微信小程序端均为手写完成
- 开发过程按赛事要求使用 AI 工具辅助开发（IDE 插件），过程记录归档于提交材料
- 仓库：GitHub 公开仓 `RainmeoX/zhike-baishitong-shuzhixin`
- commit 信息写清楚做了什么；推送前本地跑通
- 数据文件（data/ 下爬取内容）不入库，只入库代码与配置
