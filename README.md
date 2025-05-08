# 初探多模态 RAG 系统

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub stars](https://img.shields.io/github/stars/singularguy/MultimodalRAG?style=social)](https://github.com/singularguy/MultimodalRAG/stargazers)
[![Python Version](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/)
[![GitHub issues](https://img.shields.io/github/issues/singularguy/MultimodalRAG)](https://github.com/singularguy/MultimodalRAG/issues)
[![GitHub forks](https://img.shields.io/github/forks/singularguy/MultimodalRAG)](https://github.com/singularguy/MultimodalRAG/network)

---

## 项目简介

本项目演示了一个结合 **CLIP**、**Faiss** 和 **智谱 AI**，能够处理文本和图像数据的检索增强生成 (Retrieval-Augmented Generation, RAG) 系统。


---

✨ **欢迎关注我的分享渠道** ✨

*   小红书号: **AnthroSeekTheX** (Let's Seek The X!)
*   [**详细技术方案设计与思考 (飞书文档)**](https://jjrh0ec8rc.feishu.cn/docx/V5BrdafX1ovqL2xbiNlcDdsHnUh)

💡 **温馨提示 (Friendly Reminder)** 💡
> 想要快速获取本项目代码的 AI 解析？试试将浏览器地址栏中的 `github.com` 替换为 `deepwiki.com` 访问！ (例如: `https://github.com/singularguy/MultimodalRAG` -> `https://deepwiki.com/singularguy/MultimodalRAG`) *此功能依赖 Deepwiki 服务*

---

## 🚀 主要特性

*   **多模态索引**: 支持同时索引文本描述和关联图像。
*   **向量嵌入**: 使用 Hugging Face `transformers` 的 **CLIP** 模型 (`openai/clip-vit-base-patch32`) 为文本和图像生成统一的向量表示。
*   **高效检索**: 利用 **Faiss** (`IndexIDMap2` + `IndexFlatIP`) 实现快速相似性搜索。
*   **持久化存储**: 使用 **SQLite** 存储文档元数据，并将 Faiss 索引持久化到磁盘。
*   **上下文生成**: 结合检索到的信息，调用 **智谱 AI** (`glm-4-flash` 或可配置模型) 生成更精准的回答。
*   **灵活查询**: 支持纯文本、纯图像及文本+图像的多模态查询。
*   **模块化设计**: 清晰的代码结构 (`MultimodalEncoder`, `Indexer`, `Retriever`, `Generator`)。

## ⚙️ 系统要求

*   Python 3.9 或更高版本
*   智谱 AI API Key (可从 [智谱 AI 开放平台](https://open.bigmodel.cn/) 获取)
*   必要的 Python 库 (见 `requirements.txt`)

## 🛠️ 安装

我们推荐使用虚拟环境进行安装。

1.  **克隆仓库:**
    ```bash
    git clone https://github.com/singularguy/MultimodalRAG.git
    cd MultimodalRAG
    ```

2.  **创建并激活虚拟环境:**
    ```bash
    # 使用 conda (推荐)
    conda create -n multimodal_rag python=3.12 -y # 或你喜欢的 Python 3.9+ 版本
    conda activate multimodal_rag

    # 或者使用 venv
    # python -m venv venv
    # source venv/bin/activate  # Linux/macOS
    # .\venv\Scripts\activate  # Windows
    ```

3.  **安装依赖项:**
    ```bash
    pip install -r requirements.txt
    ```
    *注意: `requirements.txt` 已包含 `faiss-cpu`。如果您需要 GPU 支持且环境已配置 CUDA，可以考虑手动安装 `faiss-gpu` (可能需要先卸载 `faiss-cpu`)。*

## 🔑 配置

1.  **设置智谱 AI API Key:**
    脚本通过环境变量 `ZHIPUAI_API_KEY` 读取您的密钥。请在运行脚本前设置此环境变量：
    新建一个 `.env` 文件，并添加以下内容：
    ```dotenv
    ZHIPUAI_API_KEY=your_api_key
    ```
    *(确保你的代码能正确加载 `.env` 文件，例如使用 `python-dotenv` 库)*

## 🚀 运行

1.  **准备数据 (见下方 [数据准备](#-数据准备) 部分)。**

2.  **(可选) 修改脚本内配置:**
    您可以在主 Python 脚本 (`MultimodalRAG0428.ipynb`) 中直接修改以下变量的默认值：
    *   `json_data_path` (默认: `'data.json'`)
    *   `image_directory_path` (默认: `'images/'`)
    *   `db_file` (默认: `'multimodal_rag_data.db'`)
    *   `faiss_index_file` (默认: `'multimodal_rag_index.faiss'`)
    *   `CLIP_MODEL` (默认: `"openai/clip-vit-base-patch32"`)
    *   `LLM_MODEL` (默认: `"glm-4-flash"`) # 可根据需要修改为其他智谱模型

## 📁 数据准备

在项目根目录下准备以下数据：

1.  **`data.json`**: 一个 JSON 文件，包含文档对象的列表。每个对象需包含：
    *   `name`: 文档唯一 ID (用于关联图像，例如 `Bandgap1`)。
    *   `description`: 文档的文本内容。

    **示例 `data.json` (项目已包含):**
    ```json
    [
      {
        "name": "Bandgap1",
        "description": "一个基础的带隙基准电路图，展示了 BJT 晶体管和电阻。它用于产生一个温度不敏感的参考电压。"
      },
      {
        "name": "PTAT_Current",
        "description": "这个原理图演示了如何使用两个不匹配的 BJT 来产生与绝对温度成正比 (PTAT) 的电流。"
      },
      {
        "name": "SomeDocumentWithoutImage",
        "description": "这份文档只包含描述电压基准理论概念的文本内容。"
      }
    ]
    ```

2.  **`images/` 目录**: 包含与 `data.json` 中 `name` 字段对应的图像文件。
    *   文件名应为 `name` + 常见图片扩展名 (如 `Bandgap1.png`, `PTAT_Current.jpg`)。
    *   如果文档没有图像，则无需提供对应文件。
    *   *(项目已包含示例图片)*

## ▶️ 快速开始

1.  确保 **安装** 和 **配置** 步骤已完成。
2.  确保项目根目录下的 `data.json` 和 `images/` 目录包含你想要处理的数据（项目自带示例数据可直接运行）。
3.  运行主脚本：
    ```bash
    python yangrouchuan.py
    ```

脚本将自动执行以下步骤：
*   加载数据和图像。
*   初始化编码器、索引器、检索器和生成器。
*   **建立索引**: 为数据生成嵌入，存入 Faiss 和 SQLite。
*   **执行示例查询**: 演示文本、图像和多模态查询，并打印检索结果和 LLM 生成的响应。
*   **保存索引**: 将 Faiss 索引保存到磁盘 (`multimodal_rag_index.faiss`)。

## 🏗️ 代码结构

项目主要由以下几个类组成：

*   **`MultimodalEncoder`**: 负责加载 CLIP 模型并将文本/图像编码为向量。
*   **`Indexer`**: 管理 Faiss 索引和 SQLite 元数据存储，处理索引的创建和持久化。
*   **`Retriever`**: 接收查询，编码查询，搜索 Faiss 索引，并从数据库检索元数据。
*   **`Generator`**: 与智谱 AI API 交互，根据查询和检索到的上下文构建 Prompt 并生成响应。

主执行块 (`if __name__ == "__main__":` 在 `yangrouchuan.py` 中) 负责串联这些组件并运行示例流程。

## 💡 注意事项与未来改进

*   **性能**: 大规模数据集可考虑 `faiss-gpu` 或更高级的 Faiss 索引 (如 `IndexIVFFlat`)。
*   **多模态融合**: 当前是简单平均向量，可探索更复杂的融合策略。
*   **图像理解**: 若需 LLM 直接理解图像内容（而非基于文本描述），需切换到多模态 LLM (如 GLM-4V) 并修改 `Generator`。
*   **错误处理**: 增加更健壮的错误处理和日志。
*   **可扩展性**: 生产环境可考虑替换 SQLite 为更专业的数据库或向量数据库。
*   **提示工程**: 优化 `Generator` 中的 Prompt 可能提升效果。
*   **文本分块**: 长文本在索引前进行分块处理。

## 📅 更新日志 (Update Log)

*   **2024.04.28**: 将文本/图像的clip以及储存方式和检索方式进行修改。
*   **2024.04.27**: 新增多种技术方案的简单实现。
*   **2024.04.25**: 原始代码。

## 🤝 如何贡献

我们欢迎各种形式的贡献！如果您有任何建议、发现 Bug 或想改进代码，请随时：

*   提交 [Issues](https://github.com/singularguy/MultimodalRAG/issues)
*   创建 [Pull Requests](https://github.com/singularguy/MultimodalRAG/pulls)

<!-- 如果您希望更详细地说明贡献流程，可以创建一个 CONTRIBUTING.md 文件 -->

## 📄 License

本项目采用 [MIT License](LICENSE) 开源。