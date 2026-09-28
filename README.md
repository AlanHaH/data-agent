# 掌柜 · Data Agent

基于自然语言查询业务数据的全栈项目。前端使用 Vue 3 和 Vite，后端使用 FastAPI、LangChain 和 LangGraph，结合 MySQL、Elasticsearch、Qdrant 与 Embedding 服务检索元数据、生成和执行 SQL，并通过流式接口返回结果。

## 项目结构

```text
data-agent/                     Python 后端
  app/                          API、Agent、服务与数据访问代码
  conf/                         应用配置示例和元数据定义
  prompts/                      SQL 生成等提示词
  pyproject.toml / uv.lock       Python 依赖与锁文件
data-agent-fronted/              Vue 前端（保留现有目录名称）
  src/                          页面与组件
  package.json / package-lock.json
docker/
  docker-compose.yaml           MySQL、ES、Kibana、Qdrant、Embedding
  elasticsearch/                Dockerfile 和 IK 插件安装包
  mysql/                        数据库初始化脚本与演示数据
  embedding/README.md            模型下载说明
```

## 环境要求

- Python 3.12 或更新版本，以及 [uv](https://docs.astral.sh/uv/)。
- Node.js 20.19+ 或 22.12+，以及 npm。
- Docker 与 Docker Compose；本项目使用 Linux 容器。
- 可用的 DeepSeek API 密钥，并在配置中选择账号支持的模型。

以下命令以 PowerShell 为例，从项目根目录开始执行。

## 1. 准备本地配置

```powershell
Copy-Item data-agent/conf/app_config.example.yaml data-agent/conf/app_config.yaml
Copy-Item docker/.env.example docker/.env
```

已有本地配置时跳过复制，避免覆盖。编辑这两个文件：

- 在 `docker/.env` 中设置 MySQL root 密码与应用用户密码。
- 在 `data-agent/conf/app_config.yaml` 中，将 `db_meta.password` 和 `db_dw.password` 设置为与 `MYSQL_PASSWORD` 相同的值，并填写 `llm.api_key`。
- 数据库初始化脚本为 `atguigu` 用户授权，因此默认保留该用户名。
- 后端默认访问本机端口；如果依赖服务在其他机器上，调整相应的 `host`。

这两个本地配置文件已被 Git 忽略。提交配置变更时，请更新不含真实凭据的示例文件。

## 2. 安装后端依赖并下载模型

```powershell
cd data-agent
uv sync --locked
uv run hf download BAAI/bge-large-zh-v1.5 --revision 79e7739b6ab944e86d6171e44d24c997fc1e0116 --local-dir ../docker/embedding/bge-large-zh-v1.5
cd ..
```

模型下载需要网络连接和额外磁盘空间，具体说明见 [模型下载说明](docker/embedding/README.md)。

## 3. 启动基础服务

```powershell
cd docker
docker compose up -d --build
docker compose ps
cd ..
```

服务启动后，需要等待 MySQL、Elasticsearch、Qdrant 和 Embedding 初始化完成，再执行下一步。首次创建 MySQL 数据卷时，容器自动执行 `docker/mysql/` 中的 SQL 脚本；已有数据卷不会重新执行这些脚本，修改 `.env` 也不会自动更改已有数据库的密码。

| 服务 | 本地端口 |
| --- | --- |
| MySQL | 3306 |
| Elasticsearch | 9200 |
| Kibana | 5601 |
| Qdrant HTTP / gRPC | 6333 / 6334 |
| Embedding | 8081 |

当前 Compose 配置用于本地开发，Elasticsearch 未启用身份认证。

## 4. 构建元知识库并启动后端

首次使用时构建元知识库：

```powershell
cd data-agent
uv run python -m app.scripts.build_meta_knowledge --conf conf/meta_config.yaml
uv run python main.py
```

后端地址为 `http://localhost:8000`，接口文档为 `http://localhost:8000/docs`。查询接口为 `POST /api/query`。

## 5. 启动前端

另开一个终端，从项目根目录执行：

```powershell
cd data-agent-fronted
npm ci
npm run dev
```

打开终端显示的地址，通常为 `http://localhost:5173`。开发服务器将 `/api` 请求代理到 `http://localhost:8000`。执行 `npm run build` 可生成前端静态产物；正式部署时需另行配置 API 反向代理。

## Git 仓库包含的内容

仓库包含前后端源码、提示词、元数据配置、依赖锁文件、Docker 配置、数据库初始化脚本和 ES IK 插件安装包。第三方插件与模型遵循各自的许可证。

本地凭据、Python 虚拟环境、`node_modules`、日志、编辑器配置、构建产物和下载的模型不提交。依赖通过锁文件恢复，模型通过固定版本的下载命令恢复。
