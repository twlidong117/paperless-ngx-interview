# 第五步：检查运行 Paperless-ngx 的前置条件

日期：2026-09-08。由 Codex 基于当前仓库的开发说明、配置文件和只读环境检查辅助整理。

## 本步目标与原因

本步先确认 Paperless-ngx 提供哪些运行路径、每条路径需要哪些软件，再检查当前环境已经具备什么。这样可以在安装或启动前选出最小且可验证的下一步，避免把“命令存在”“依赖可用”和“应用运行成功”混为一谈。

本步只读取文件、系统信息、命令版本和软件包状态。没有安装软件，没有创建运行配置或目录，也没有启动 Paperless-ngx、容器、消息代理或数据库服务。

## 新名词

- **前置条件**：执行后续操作前必须具备的软件、配置或系统能力。
- **依赖**：项目运行或开发需要的其他软件或软件包。
- **容器**：把应用及其运行环境隔离打包的一种运行方式。
- **Docker**：创建和运行容器的平台；它包含命令行客户端以及通常在后台工作的服务端。
- **Docker Compose**：读取组合配置文件、协调多个容器共同运行的工具。
- **本地开发方式**：直接在当前操作系统中准备解释器、系统软件和项目依赖，再从源代码运行应用的方式；也常称裸机方式，但不表示一定是没有虚拟化的实体机器。
- **后端**：处理业务规则、数据库和后台任务的服务器端部分。Paperless-ngx 后端是 Django 应用。
- **前端**：浏览器中显示和交互的页面部分。Paperless-ngx 前端使用 Angular 构建。
- **解释器**：读取并执行某种编程语言代码的软件；这里主要指 Python。
- **包管理器**：下载和管理项目软件包的工具；本项目使用 `uv` 管理 Python 包和虚拟环境，使用 pnpm 管理前端包。
- **虚拟环境**：为单个项目隔离 Python 软件包的目录，通常名为 `.venv`。
- **消息代理**：在不同后台进程之间传递任务消息的软件。Paperless-ngx 可使用 Valkey 或 Redis 协议兼容的代理。
- **数据库**：持久保存用户、文档信息和配置等结构化数据的软件。SQLite 使用本地文件；PostgreSQL 和 MariaDB 是可选的独立数据库服务器。
- **JSON1 扩展**：SQLite 提供 JSON 数据查询能力的扩展；直接使用 SQLite 运行时需要它。
- **光学字符识别引擎**：从扫描图片提取文字的软件；本项目使用 Tesseract，并通过 OCRmyPDF 等组件处理文档。
- **锁定版本**：项目明确指定的工具或依赖版本，用于让不同环境得到更一致的结果。
- **迁移**：把数据库结构更新到当前代码要求状态的操作。
- **守护进程**：长期在后台运行并提供服务的程序，例如消息代理或 Docker 服务端。

## 三种状态必须分开

1. **软件已安装**：系统能够找到命令，或操作系统软件包数据库记录该软件包已安装。
2. **软件能够使用**：命令能够成功完成与目标相关的安全检查。本步最多验证版本查询、Python 导入或 SQLite JSON1 查询；没有验证完整业务协作。
3. **应用运行成功**：Paperless-ngx 的后端、后台任务和必要服务已经实际启动，并通过页面或健康检查证明可工作。本步没有启动应用，因此不能得出此结论。

一个例子：`python3 --version` 成功证明 Python 命令可执行；Django 无法导入说明当前 Python 环境尚未具备项目后端依赖；即使 Django 可以导入，也仍然不能证明 Paperless-ngx 已启动。

## 仓库提供的两条运行路径

### 路径一：Docker Compose 容器方式

`docs/setup.md` 第 40 至 66 行说明安装脚本和手工 Docker Compose 方式都以 Docker 与 Docker Compose 为前置条件。第 70 至 128 行说明还要准备组合配置和环境配置，拉取镜像后才会创建并启动容器。本步没有执行这些动作。

代表性的 `docker/compose/docker-compose.sqlite.yml` 第 24 至 45 行定义了两个服务：

- `broker` 使用 Valkey 容器传递后台任务消息；
- `webserver` 使用 Paperless-ngx 镜像提供应用，通过端口 8000 对外服务；
- SQLite 数据库存放在数据卷中，因此该组合不要求宿主机另装 PostgreSQL 或 MariaDB；
- 容器配置仍需要数据、媒体、导入和导出存储位置。

容器方式把 Python、Tesseract 等许多应用内部依赖放进镜像管理，但宿主环境至少要能使用 Docker 和 Docker Compose。当前环境找不到 `docker` 命令，所以这条路径目前不具备启动前置条件。

### 路径二：源代码本地开发方式

`docs/development.md` 第 56 至 109 行要求先按本地安装说明准备系统依赖和 `uv`，再创建配置、数据目录、Python 依赖、数据库迁移和消息代理。后端启动还涉及 Django 开发服务器、文档消费者和 Celery 后台任务进程（第 111 至 132 行）。

所需条件及作用如下：

| 条件 | 作用 | 是否必需 |
| --- | --- | --- |
| Linux | 官方支持直接运行源代码的操作系统 | 本地方式必需 |
| Python 3.11、3.12、3.13 或 3.14 | 执行后端代码 | 本地方式必需 |
| `uv` | 创建 Python 虚拟环境并安装锁定依赖 | 开发说明要求 |
| Python 项目依赖 | 提供 Django、Celery、OCRmyPDF 等后端能力 | 后端必需 |
| 系统编译与文档工具 | 编译部分 Python 扩展，识别文件类型，转换便携式文档格式和执行光学字符识别 | 本地完整文档处理必需 |
| Valkey 或 Redis 协议兼容消息代理 | 向后台工作进程传递任务 | 运行核心后台处理必需 |
| SQLite、PostgreSQL 或 MariaDB | 保存结构化数据 | 三选一；新部署文档推荐 PostgreSQL |
| `paperless.conf` | 保存本地实例的调试和连接等配置 | 开发说明要求创建 |
| `consume` 与 `media` 目录 | 接收待导入文件及保存应用媒体文件 | 开发说明要求创建 |
| Node.js 24 或更高版本 | 执行前端构建工具 | 前端开发必需 |
| pnpm | 安装和运行前端依赖 | 前端开发必需 |

Tika 与 Gotenberg 用于可选的 Office 文档和电子邮件解析、转换，并非最小 SQLite 开发路径的必选条件。PostgreSQL 和 MariaDB 也是替代 SQLite 的数据库选择，不应把三种数据库都列为同时必需。

## 当前环境只读检查结果

执行位置均为项目最外层目录 `/workspace/paperless-ngx-interview`。

### 已具备且完成了有限可用性检查

| 条件 | 实际证据 | 能得出的结论 |
| --- | --- | --- |
| 操作系统 | `uname -s` 返回 `Linux`；系统是 Ubuntu 24.04.4 LTS | 满足本地方式的 Linux 条件 |
| 处理器架构 | `uname -m` 返回 `x86_64` | 与组合文件声明支持的 `amd64` 对应 |
| Git | `git --version` 返回 2.43.0 | Git 命令可用 |
| Python | `python3 --version` 返回 3.14.4 | 版本在官方列出的 3.11 至 3.14 范围内 |
| pip | `pip3 --version` 成功，来自 pyenv 的 Python 3.14.4 | pip 命令可用；不能据此证明项目依赖已安装 |
| `uv` | `uv --version` 返回 0.7.22 | `uv` 命令可用 |
| Node.js | `node --version` 返回 24.15.0 | 满足前端文档的 24 或更高版本条件 |
| pnpm | `pnpm --version` 返回 10.28.1 | 命令可用，但与 `src-ui/package.json` 声明的 `pnpm@11.15.1` 不一致 |
| SQLite | 命令行版本与 Python 使用的库均为 3.45.1 | SQLite 基础能力可用 |
| SQLite JSON1 | Python 内存数据库执行 `json_valid('{}')` 返回 1 | 当前 Python 的 SQLite 支持 JSON1 |
| 构建基础 | Ubuntu 记录 `build-essential` 已安装；`gcc` 与 `make` 版本查询成功 | 基础编译命令可用 |
| 部分系统库 | `gnupg`、`libpq-dev`、`default-libmysqlclient-dev`、`pkg-config`、`libxml2`、`zlib1g` 已安装 | 仅这些单项具备，不能代表完整依赖集合 |
| 前端依赖目录 | `src-ui/node_modules` 存在 | 只证明目录存在；未验证内容完整或构建成功 |

### 缺失或尚未达到要求

| 条件 | 实际证据 | 影响 |
| --- | --- | --- |
| Docker 与 Docker Compose | 找不到 `docker`；两个版本查询均以 127 退出 | 当前不能采用官方 Docker Compose 路径 |
| 消息代理 | 找不到 `redis-server`、`valkey-server` 和 `redis-cli` | 本地核心后台处理缺少消息代理；也没有 Docker 可临时提供代理 |
| 项目虚拟环境 | `.venv` 不存在 | 没有项目隔离的 Python 环境证据 |
| Python 项目依赖 | Django、Celery、Redis 客户端、OCRmyPDF、python-magic、PostgreSQL 与 MariaDB 驱动均无法由当前 Python 找到 | 后端尚不能按当前 Python 环境直接运行 |
| 开发配置 | `paperless.conf` 不存在，只有示例文件 | 尚未完成首次开发配置 |
| 数据目录 | `consume` 与 `media` 不存在 | 尚未完成开发说明要求的首次目录准备 |
| pnpm 锁定版本 | 当前 10.28.1，项目声明 11.15.1 | 不能认定前端包管理环境与仓库要求一致 |
| 光学字符识别与文档工具 | Tesseract、Ghostscript、qpdf、unpaper、ImageMagick、Poppler 工具和 libmagic 命令均未找到 | 本地方式无法完成完整文档识别与转换 |
| 多项系统开发包 | `python3-dev`、Ubuntu 的 `python3-pip`、`python3-setuptools`、`python3-wheel`、`fonts-liberation`、`libmagic-dev` 等未记录为已安装 | 本地依赖安装前置集合不完整 |
| 独立数据库客户端 | `psql` 与 `mariadb` 均未找到 | 不能使用客户端检查 PostgreSQL 或 MariaDB；最小 SQLite 路径不要求这两者 |

`pip3` 命令存在而 Ubuntu 的 `python3-pip` 软件包记录缺失并不矛盾：前者来自 pyenv 管理的 Python，后者是 Ubuntu 软件包管理器中的特定软件包。应以实际选择的安装方式判断，不能只看一个软件包名称。

## 具体操作

执行者：Codex。执行位置：`/workspace/paperless-ngx-interview`。

1. 读取 `AGENTS.md`、`learning/PROGRESS.md` 和 `learning/steps/04-business-workflow.md`，确认学习规则与第四步边界。
2. 使用 `find` 和 `rg` 定位 `docs/development.md`、`docs/setup.md`、`pyproject.toml`、`src-ui/package.json` 和 Docker Compose 配置中的前置条件依据。
3. 使用 `uname` 和 `/etc/os-release` 读取操作系统与架构。
4. 使用 `command -v` 以及版本参数检查现有命令，不启动其服务。
5. 使用 Python 内存数据库查询 SQLite JSON1；内存数据库不会写入项目文件。
6. 使用 Python 模块定位检查关键项目包是否可导入，但不安装和导入运行应用。
7. 使用 `dpkg-query` 读取 Ubuntu 软件包数据库。
8. 首次组织软件包查询命令时，命令编排层把 `${...}` 误作文本替换并产生 `SyntaxError`；该命令未开始执行。改为原样字符串后，同一只读检查成功完成。
9. 尝试核验相关官方网页时，网页读取工具返回 `401 Unauthorized`，随后 `curl` 返回 `403 Forbidden`。因此相关页面的内容依据来自当前仓库的官方文档源文件，不声称外部网页正文复核成功。

所有版本查询和状态查询都没有启动 Paperless-ngx。没有执行 `uv sync`、`pnpm install`、`docker compose up`、数据库迁移或任何服务启动命令。

## 验收

本步已达到“识别运行路径、解释软件职责、只读确认当前前置条件”的目标：

- Docker Compose 路径因 Docker 缺失而尚不可用；
- 本地开发路径已有 Linux、受支持 Python、`uv`、Node.js、SQLite JSON1 和部分构建库；
- 本地路径仍缺项目 Python 依赖、消息代理、主要文档处理工具、首次配置和目录；
- pnpm 可执行但版本与项目声明不一致；
- 应用没有启动，所以没有应用运行成功证据。

## 下一步最小操作

下一步只选择并准备一种运行路径，不同时尝试两套方案。针对当前以源代码学习和面试材料积累为目标的仓库，建议第六步先**制定并验收本地开发依赖安装方案**：根据 Ubuntu 24.04 和项目文档整理准确的软件包清单，先解决 pnpm 版本与项目声明的差异，再经用户明确要求后才执行安装。

这只是下一步预告。本步不安装依赖、不复制 `paperless.conf`、不创建目录、不执行数据库迁移，也不启动消息代理或应用。

## 知识点总结

- 项目支持 Docker Compose 和源代码本地开发两条路径；两条路径的宿主机前置条件不同，不能把清单混在一起。
- 命令能够报告版本，只证明有限可用性；只有实际启动并通过有效检查，才能声称应用运行成功。
- Python 版本符合要求不代表 Django、Celery 和 OCRmyPDF 等项目依赖已经安装。
- SQLite、PostgreSQL 与 MariaDB 是替代选择；Tika 和 Gotenberg也是可选扩展，不需要为最小路径全部安装。
- 前端要求 Node.js 24 或更高版本和 pnpm；当前 pnpm 虽可用，但没有达到仓库声明的锁定版本。

## 相关官方文档

- [Paperless-ngx 开发说明](https://docs.paperless-ngx.com/development/)：查看首次开发配置、`uv`、消息代理以及前后端开发命令；本仓库对应源文件是 `docs/development.md`。
- [Paperless-ngx 安装说明](https://docs.paperless-ngx.com/setup/)：查看 Docker Compose 和本地安装两条路径、系统软件及数据库选择；本仓库对应源文件是 `docs/setup.md`。
- [Docker Engine 的 Ubuntu 安装说明](https://docs.docker.com/engine/install/ubuntu/)：下一步若选择容器路径，用于核对 Ubuntu 支持范围和官方安装步骤。
- [`uv` 官方安装说明](https://docs.astral.sh/uv/getting-started/installation/)：下一步若需要重建 Python 工具环境，用于核对官方安装及版本检查方法。

上述链接与当前仓库官方文档中的引用或工具官网对应。2026-09-08 外部页面读取受当前环境的 `401` 和 `403` 限制；本步已直接核验仓库内的 Paperless-ngx 官方文档源文件，不虚构外部页面读取结果。

## 面试题与参考答案

**问题一：为什么 Python 3.14.4 能执行，还不能说 Paperless-ngx 后端可以运行？**

Python 解释器只是基础前置条件。后端还需要 Django、Celery、Redis 客户端、OCRmyPDF 等 Python 包，以及消息代理、配置和数据目录。本次关键包定位均失败，因此不能从 Python 版本成功推导出应用可运行。

**问题二：Docker Compose 路径和本地开发路径的主要取舍是什么？**

Docker Compose 把应用及大量依赖放在容器中统一管理，宿主机准备项较少，但必须先具备可用的 Docker 和 Docker Compose。本地开发路径便于直接调试源代码，但需要单独准备 Python、系统文档工具、消息代理和项目依赖，环境差异也更多。

**问题三：检查到 `redis-server` 不存在时，应怎样排查而不立即下结论？**

先确认选择的是哪条路径。容器方式可以由组合文件启动 Valkey 容器，不要求宿主机有 `redis-server`；本地方式则要安装 Valkey、Redis 或使用容器提供兼容代理。还要区分命令不存在、服务未启动和连接配置错误，这三类问题的处理不同。

**问题四：为什么 pnpm 10.28.1 可执行仍被记录为未完全满足？**

因为 `src-ui/package.json` 声明 `pnpm@11.15.1`。版本查询成功只证明现有 pnpm 能运行，不能证明它符合项目锁定版本，也不能证明现有 `node_modules` 可以成功构建前端。

## 可选练习

判断下面说法是否正确，并说明原因：

> `python3 --version` 和 `node --version` 都成功，所以 Paperless-ngx 已经可以运行。

参考判断：错误。两个命令只证明解释器和前端运行时能够报告版本；项目依赖、消息代理、文档工具和配置仍不完整，而且应用没有启动。

## 本步完成状态与下一步预告

第五步前置条件检查已完成；应用未安装、未启动、未进行功能验证。下一步由用户明确要求后，再单独制定并验收一种运行路径的依赖安装方案。
