# 掌柜问数 · 数据问答 Agent

大模型项目：基于 **RAG（检索增强生成）** 的数据智能问答系统。用户用自然语言提问，系统自动定位相关数据并给出答案。

## 项目架构

| 服务 | 技术 | 端口 | 作用 |
|------|------|------|------|
| mysql | MySQL 8.0 | 3306 | 业务数据 + 数据库元数据（table_info / column_info） |
| elasticsearch | ES 8.19.10 + IK 中文分词 | 9200 | 元数据全文检索 |
| kibana | Kibana 8.19.10 | 5601 | ES 可视化控制台 |
| qdrant | Qdrant v1.16 | 6333/6334 | 向量数据库 |
| embedding | text-embeddings-inference + bge-large-zh-v1.5 | 8081 | 文本向量化 |

### 检索链路

```
用户提问 ──► ES 全文检索元数据（IK 分词）
         └► Qdrant 向量相似度检索（bge 向量化）
                │
                ▼
        定位相关表/字段 ──► 大模型生成 SQL ──► MySQL 查数 ──► 回答
```

## 快速开始

```bash
cd docker
docker compose up -d
```

### 前置：下载向量模型

bge 模型体积过大（1.3GB，超过 GitHub 限制），需自行下载后放入指定目录：

```bash
# 从 HuggingFace 下载 bge-large-zh-v1.5 到以下目录
docker/embedding/bge-large-zh-v1.5/
```

## 国内网络说明

拉取 Docker 镜像如遇超时，可在 Docker 配置镜像加速器（已内置可用源）：
`~/.docker/daemon.json` 中配置 `registry-mirrors`，例如：

```json
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://docker.1ms.run"
  ]
}
```

## 项目结构

```
├── docker/
│   ├── docker-compose.yaml      # 全部服务编排
│   ├── mysql/                   # 初始化 SQL（业务数据 + 元数据）
│   ├── elasticsearch/           # ES 镜像构建 + IK 插件
│   └── embedding/               # 向量化模型目录（需自行下载）
├── pyproject.toml               # Python 依赖（uv 管理）
├── uv.lock
└── 尚硅谷大模型项目之掌柜问数.md   # 完整开发文档
```
