# 第七步：验收准备脚本安装的 Python 依赖

日期：2026-09-10。由 Codex 根据当前环境中的实际命令输出辅助整理。

## 本步目标与原因

本步只验收准备脚本安装 Python 依赖后的结果：确认检查命令实际使用项目 `.venv` 中的 Python，验证 Django 和 Celery 能被加载，并确认 `pyproject.toml` 与 `uv.lock` 没有被修改。这样可以区分“两个关键框架已经可导入”“整个锁定环境已通过一致性检查”和“应用已经启动”这三个不同结论。

本步不启动应用，不执行数据库迁移，也不通过降低版本或删除依赖掩盖问题。由于最后的锁定环境一致性检查失败，本步停在第一个有意义的错误，整体状态记为**未通过**。

## 本步出现的新名词及解释

- **虚拟环境（virtual environment）**：项目专用的 Python 与软件包目录，用于和系统环境隔离；本项目目录是 `.venv`。
- **`sys.executable`**：Python 运行时报告的解释器入口路径，用它可以确认当前命令由哪个 Python 执行。
- **导入（import）**：让 Python 查找并加载一个已安装的软件包；能成功导入是软件包基本可用的直接证据，但不等于整个应用可以运行。
- **依赖锁定文件**：记录依赖解析结果的文件；本项目使用 `uv.lock`，以便不同环境尽量安装一致的版本集合。
- **对象哈希**：Git 根据文件内容计算的标识；同一文件检查前后的哈希一致，说明内容没有变化。
- **网络隧道**：当前环境访问外部网络所经过的转发通道；隧道连接失败会阻止下载，但它与 Python 包本身的导入错误不是同一类问题。

## 具体操作

执行者：Codex。执行位置：仓库根目录 `/workspace/paperless-ngx-interview`。

### 1. 建立文件基线

先执行 `git status --short -- pyproject.toml uv.lock` 和 `git hash-object pyproject.toml uv.lock`。

- `git status --short` 只查看指定文件相对当前提交是否有变化；预期无输出。
- `git hash-object` 计算文件当前内容的 Git 对象哈希；用于检查前后比较。

实际无状态输出，初始哈希为：

```text
pyproject.toml  8364aacf07c827a7bbea7334d63573af9c8cdce6
uv.lock         68aa1b8499894bca1b50c3db4662914e0ec0cccb
```

### 2. 核对解释器并加载关键框架

直接运行 `.venv/bin/python`，在同一个 Python 进程中读取 `sys.executable`，再导入 `django` 与 `celery` 并打印版本和模块文件位置。预期解释器入口位于项目 `.venv`，两个模块文件也位于该虚拟环境。

实际关键信息为：

```text
sys.executable=/workspace/paperless-ngx-interview/.venv/bin/python
resolved_executable=/root/.pyenv/versions/3.12.13/bin/python3.12
django_version=5.2.16
django_module=/workspace/paperless-ngx-interview/.venv/lib/python3.12/site-packages/django/__init__.py
celery_version=5.6.3
celery_module=/workspace/paperless-ngx-interview/.venv/lib/python3.12/site-packages/celery/__init__.py
```

`.venv/bin/python` 最终解析到创建虚拟环境所用的基础 Python 文件是正常结构；`sys.executable` 保留项目入口，且两个模块实际从 `.venv` 加载，共同证明本次框架验证使用了项目虚拟环境。该组合检查退出状态为 `0`。

### 3. 只读检查锁定环境一致性

执行 `uv sync --check`。`sync` 表示对照项目声明和锁定结果检查虚拟环境，`--check` 表示只检查而不执行同步修改。预期命令返回 `0`；若返回非零状态，不能确认准备脚本安装的整个依赖环境已经通过验收。

该命令实际返回 `2`，第一个有意义的错误是：

```text
error: Failed to generate package metadata for `psycopg-c==3.3.4 @ direct+https://github.com/paperless-ngx/builder/releases/download/psycopg-trixie-3.3.4/psycopg_c-3.3.4-cp312-cp312-linux_aarch64.whl`
Caused by: Failed to fetch: `https://github.com/paperless-ngx/builder/releases/download/psycopg-trixie-3.3.4/psycopg_c-3.3.4-cp312-cp312-linux_aarch64.whl`
Caused by: tunnel error: unsuccessful
```

`psycopg-c` 是 PostgreSQL 数据库驱动 Psycopg 的 C 语言加速组件。错误发生在 uv 获取项目指定构建产物、生成软件包元数据时；网络隧道连接失败使检查无法完成。这不等于 Django 或 Celery 导入失败，也不能据此确认或否定其余所有已安装软件包，但足以使“整个锁定环境检查成功”这一验收项失败。

遵照本步规则，记录该错误后不再运行新的依赖或应用检查；没有降级或删除任何依赖。

### 4. 复核配置和锁定文件

失败后比较检查前后的哈希，并执行 `git diff --exit-code -- pyproject.toml uv.lock`。`--exit-code` 在没有差异时返回 `0`。

两个文件的前后哈希分别保持为 `8364aacf07c827a7bbea7334d63573af9c8cdce6` 和 `68aa1b8499894bca1b50c3db4662914e0ec0cccb`；指定文件的 Git 状态仍无输出，差异检查返回 `0`。

## 验收

| 验收项 | 实际结果 | 判断 |
| --- | --- | --- |
| 使用项目 `.venv` 的 Python | `sys.executable` 为项目 `.venv/bin/python` | 通过 |
| Django 加载 | 从项目 `.venv` 导入 `5.2.16` | 通过 |
| Celery 加载 | 从项目 `.venv` 导入 `5.6.3` | 通过 |
| 整个锁定环境检查 | `uv sync --check` 因外部构建产物的网络隧道错误返回 `2` | 未通过 |
| `pyproject.toml` 未修改 | 前后哈希相同、Git 无差异 | 通过 |
| `uv.lock` 未修改 | 前后哈希相同、Git 无差异 | 通过 |
| 应用启动 | 未执行 | 不在本步范围 |

本步整体**未通过**。准确边界是：关键框架在项目虚拟环境中可加载，配置与锁定文件未修改；但 `uv sync --check` 未完成，所以不能宣称准备脚本安装的全部 Python 依赖已经验收成功。没有启动应用。

## 知识点总结

- `sys.executable` 和模块的 `__file__` 应结合使用，分别证明解释器入口与包的实际加载位置。
- 成功导入 Django 和 Celery 只验证两个关键包，不代表锁定文件描述的完整环境一致。
- `uv sync --check` 的非零退出状态意味着一致性验收没有成功；应保留原始因果链，而不是把网络失败误报为框架加载失败。
- 文件前后哈希、Git 状态和 Git 差异检查共同提供“配置及锁定文件未改”的可复查证据。
- 验证失败时不应靠降低版本或删除依赖获得表面成功，也不能把“未启动”写成“应用可运行”。

## 相关文档

- [Python 官方 `venv` 文档](https://docs.python.org/3/library/venv.html)：用于了解虚拟环境如何隔离解释器环境及常见目录结构。
- [Django 官方工具参考](https://docs.djangoproject.com/en/5.2/ref/utils/)：用于查阅 Django 工具接口；本次版本值由实际安装包的 `django.get_version()` 给出。
- [Celery 官方入门介绍](https://docs.celeryq.dev/en/stable/getting-started/introduction.html)：用于了解 Celery 在任务队列中的角色；本次只验证 Python 包导入，不启动 worker（任务执行进程）。
- [uv 官方项目同步文档](https://docs.astral.sh/uv/concepts/projects/sync/)：用于理解项目环境同步、锁定文件与检查行为。

2026-09-10 已尝试核验以上页面正文。网页检索接口返回 `401 Unauthorized`；命令行 `curl --location --fail --silent --show-error --max-time 20` 访问四个官方页面均返回 HTTP 403。因此这里只保留可复查的官方页面及阅读重点，并明确记录本环境未能读取正文；没有虚构网页内容。当前步骤的技术结论全部来自仓库文件和实际命令输出。

## 面试题与参考答案

**问题一：为什么只看终端中的环境名称不足以证明使用了项目虚拟环境？**

终端提示符可以被手工设置，也可能过时。读取 `sys.executable` 能看到 Python 自己报告的入口，再检查 Django 和 Celery 的 `__file__`，可以证明解释器入口和实际包文件都属于项目 `.venv`。

**问题二：Django 和 Celery 都能导入，为什么本步仍判定未通过？**

因为目标还包括检查准备脚本安装的完整锁定环境。两个关键包导入成功只是局部证据；`uv sync --check` 返回非零状态，完整一致性没有得到证明，所以整体不能通过。

**问题三：遇到 `tunnel error: unsuccessful` 应先排查什么？**

先保留完整错误链，确认失败发生在访问哪个地址以及代理或网络隧道是否允许访问，而不是先修改依赖版本。还应确认验证命令没有改动 `pyproject.toml` 或 `uv.lock`。本步在错误后停止，没有尝试绕过。

**问题四：如何证明只读验收没有改动依赖声明？**

在检查前后计算 `pyproject.toml` 和 `uv.lock` 的 Git 对象哈希，再查看指定文件的 Git 状态和差异。哈希相同、状态无输出且 `git diff --exit-code` 返回 `0`，说明文件内容未变化。

## 可选的复述或判断练习

判断下面说法是否正确，并说明原因：

> Celery `5.6.3` 已经能导入，所以准备脚本安装的全部依赖已经验收成功。

参考判断：错误。它只证明 Celery 这一软件包能从项目虚拟环境加载；完整锁定环境检查本次因网络隧道错误未完成。

## 本步完成状态及下一步预告

第七步已执行并保存，但验收整体未通过：项目 `.venv`、Django、Celery 和文件未修改四类证据均已取得，完整依赖一致性检查停在外部构建产物的网络隧道错误。下一步只预告为：用户要求继续时，先重新确认网络访问条件，再重跑本步骤中失败的检查；在此之前不启动应用。
