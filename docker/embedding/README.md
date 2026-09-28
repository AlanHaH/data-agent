# Embedding 模型

项目使用 `BAAI/bge-large-zh-v1.5`，向量维度为 1024。模型文件体积约 1.21 GiB，不提交到 Git；运行前下载到此目录下的 `bge-large-zh-v1.5/`。

在项目根目录执行以下 PowerShell 命令：

```powershell
cd data-agent
uv sync --locked
uv run hf download BAAI/bge-large-zh-v1.5 --revision 79e7739b6ab944e86d6171e44d24c997fc1e0116 --local-dir ../docker/embedding/bge-large-zh-v1.5
cd ..
```

这里固定了当前本地模型对应的版本，便于恢复相同的运行环境。下载完成后，目录中应包含 `config.json`、`tokenizer.json`、`pytorch_model.bin` 等模型文件。Docker Compose 会将此目录挂载到推理服务的 `/models/bge-large-zh-v1.5`。

模型来源和许可证见 [Hugging Face 模型页面](https://huggingface.co/BAAI/bge-large-zh-v1.5)。
