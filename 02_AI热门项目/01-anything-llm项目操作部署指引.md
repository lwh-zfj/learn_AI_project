lwh-zfj:
请告诉我这个hithub项目： `https://github.com/Mintplex-Labs/anything-llm.git`  在本机如何安装部署。


智能AI-Builder：
          
**项目简介**
- AnythingLLM 是一体化的本地/服务器端 AI 应用，支持对话、RAG、智能体、无代码 Agent 构建、MCP 兼容等。
- 既可接入商用模型（OpenAI、Anthropic、Cohere、Azure/OpenAI 等），也可对接本地推理（如 Ollama、LM Studio）。
- 内置向量库（LanceDB）与私有嵌入器，文档分片与工作区管理开箱可用，适合搭建“私有版 ChatGPT”。

**本机安装部署（macOS）—推荐：Docker 方式**
- 前置要求：已安装 Docker（建议 v18.03+），内存≥2GB，磁盘至少预留 10GB。
- 持久化运行，数据升级不丢失：

```bash
# 1) 拉取镜像
docker pull mintplexlabs/anythingllm

# 2) 准备本地存储与环境文件
export STORAGE_LOCATION=$HOME/anythingllm
mkdir -p $STORAGE_LOCATION
touch "$STORAGE_LOCATION/.env"

# 3) 启动容器（暴露 3001 端口）
docker run -d -p 3001:3001 \
  --cap-add SYS_ADMIN \
  -v ${STORAGE_LOCATION}:/app/server/storage \
  -v ${STORAGE_LOCATION}/.env:/app/server/.env \
  -e STORAGE_DIR="/app/server/storage" \
  mintplexlabs/anythingllm

# 4) 浏览器访问
# http://localhost:3001 完成初始化与登录
```

- 首次进入后在 Settings 中选择模型来源：
  - 使用 OpenAI：在界面中填入 OpenAI API Key，选择对话与嵌入模型。
  - 使用本地 Ollama：将 Base URL 配置为 http://host.docker.internal:11434（Docker 容器访问宿主机时需要这个地址）。

**可选：Docker Compose（便于集中配置）**
- 将以下内容保存为 docker-compose.yml，并把 volumes.device 改为你本地存储路径：

```yaml
version: '3.8'
services:
  anythingllm:
    image: mintplexlabs/anythingllm
    container_name: anythingllm
    ports:
      - "3001:3001"
    cap_add:
      - SYS_ADMIN
    environment:
      - STORAGE_DIR=/app/server/storage
      - JWT_SECRET=please-change-me-20plus-characters
      - LLM_PROVIDER=ollama
      - OLLAMA_BASE_PATH=http://host.docker.internal:11434
      - OLLAMA_MODEL_PREF=llama3
      - EMBEDDING_ENGINE=ollama
      - EMBEDDING_BASE_PATH=http://host.docker.internal:11434
      - EMBEDDING_MODEL_PREF=nomic-embed-text:latest
      - VECTOR_DB=lancedb
    volumes:
      - anythingllm_storage:/app/server/storage
    restart: always

volumes:
  anythingllm_storage:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /Users/<你的用户名>/anythingllm
```

- 启动与访问：

```bash
docker compose up -d
# 浏览器访问 http://localhost:3001
```

**对接本地模型（Ollama 示例）**
- 安装并启动 Ollama（默认监听 127.0.0.1:11434）。
- 拉取推理与嵌入模型：

```bash
ollama pull llama3


ollama pull nomic-embed-text
```

- 在 AnythingLLM 设置里选择 LLM Provider=Ollama，Base URL 用 http://host.docker.internal:11434，模型选择你拉取的名称；嵌入引擎同理。

**从源码构建（不推荐日常使用）**
- 仍以 Docker 构建为主，但从仓库初始化步骤如下：

```bash
git clone https://github.com/Mintplex-Labs/anything-llm.git
cd anything-llm
touch server/storage/anythingllm.db
cd docker
cp .env.example .env
docker-compose up -d --build
# 访问 http://localhost:3001
```

**常见问题与排查**
- 容器访问宿主机服务：在配置里用 http://host.docker.internal:端口（macOS/Windows），Linux 需添加 --add-host 或用 172.17.0.1:端口。
- 权限问题：容器内默认 UID/GID 为 1000；若宿主机用户 ID 不同，可能需调整挂载目录权限或在 .env 中设置匹配的 UID/GID。
- LLM 显示 offline 或登录失败：如果不是本机访问，确保前端 API Base 指向实际可达地址（例如 http://<你的IP>:3001/api）。
- 端口占用：修改 -p 3001:3001 的左侧宿主机端口即可，例如 -p 8080:3001。

**下一步建议**
- 先用 Docker 快速跑起来，确认能导入文档、创建 Workspace、完成一次 RAG 检索与对话。
- 再按需切换模型来源（OpenAI 或 Ollama）与嵌入引擎，测试不同效果与成本。








