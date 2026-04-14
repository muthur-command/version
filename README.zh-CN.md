# MCOS 版本通道数据

本仓库发布面向 **Supervisor**、**OS** 与 **mcio pre-pull** 消费方的通道元数据与签名文件。

## 发布文件

- 通道 JSON：`stable.json`、`beta.json`、`dev.json`
- 签名文件：`*.sig`（由工作流对签名目标自动生成）
- 静态配置：`apparmor*.txt`

## 工作流前置配置

在执行发布相关任务前，请先完成 `version/.github/workflows/version.yml` 与仓库 Secrets 配置。

### 环境变量（`version.yml` -> `env`）

| 变量名 | 用途 |
|---|---|
| `MCOS_VERSION_S3_BUCKET` | 目标桶名称。若使用 AList 的 S3 映射，此值需与映射暴露的桶名一致。 |
| `MCOS_VERSION_S3_PUBLIC_BASE` | S3 兼容 API 地址（传给 `aws s3 --endpoint-url`）。 |
| `MCOS_VERSION_PUBLIC_BASE` | 客户端下载 `stable.json` 与静态资源时使用的公开 HTTPS 基础地址。 |

> `MCOS_VERSION_S3_PUBLIC_BASE` 与 `MCOS_VERSION_PUBLIC_BASE` 可以相同，也可以分离（例如：S3 API 域名 + CDN/下载域名）。

### 必需的 GitHub Actions Secrets

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`

配置路径：**Settings -> Secrets and variables -> Actions -> New repository secret**

若任一 Secret 缺失，上传/签名步骤会因凭据错误失败（例如：`Unable to locate credentials`）。

## 触发行为（基于工作流）

- 向 `mc` 分支 `push`（当 `*.txt`、`*.json`、`*.png` 或工作流文件变更时）
- 向 `mc` 分支发起 `pull_request`（执行 lint/prepare 路径）
- 手动触发 `workflow_dispatch`（需提供 `files` 输入）

## TLS 故障排查

若出现 `SSL validation failed` 或 `TLSV1_UNRECOGNIZED_NAME`：

1. 确认 `MCOS_VERSION_S3_PUBLIC_BASE` 的主机名与证书 SAN/CN 匹配。
2. 确认反向代理 SNI 配置正确。
3. 优先使用受信任证书（如 Let's Encrypt）。

临时实验环境绕过（仅限排障）：

- 设置仓库变量 `MCOS_S3_NO_VERIFY_SSL=true`（Actions Variables）。
- 工作流将附带 `aws --no-verify-ssl` 参数。
- 生产环境请勿保留该配置。
