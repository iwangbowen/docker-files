# Gotenberg 文档转换服务

基于 [Gotenberg](https://github.com/gotenberg/gotenberg) 的文档转 PDF 服务。

## 功能特性

- 支持 Word (.doc, .docx)、Excel (.xls, .xlsx)、PPT (.ppt, .pptx) 等格式转换为 PDF
- 基于 LibreOffice 的无头转换
- RESTful API 接口
- 轻量级 Docker 镜像

## 启动服务

```bash
docker compose up -d
```

## 查看日志

```bash
docker compose logs -f
```

## API 使用

### 基本转换

```bash
# 将 Word 文档转换为 PDF
curl -X POST http://localhost:3434/forms/libreoffice/convert \
  -F "file=@/path/to/your/document.doc" \
  --output output.pdf

# 将 Excel 转换为 PDF
curl -X POST http://localhost:3434/forms/libreoffice/convert \
  -F "file=@/path/to/your/spreadsheet.xlsx" \
  --output output.pdf

# 将 PPT 转换为 PDF
curl -X POST http://localhost:3434/forms/libreoffice/convert \
  -F "file=@/path/to/your/presentation.pptx" \
  --output output.pdf
```

### Windows PowerShell 示例

```powershell
# 转换单个文件
Invoke-RestMethod -Method POST -Uri "http://localhost:3434/forms/libreoffice/convert" `
  -Form @{file = Get-Item "C:\path\to\document.doc"} `
  -OutFile "C:\path\to\output.pdf"
```

## 端口映射

| 容器端口 | 主机端口 | 说明 |
|:--------:|:--------:|:-----|
| 3000 | 3434 | API 服务端口 |

## 数据持久化

使用 Docker 命名卷进行数据持久化：

| 卷名称         | 容器路径         | 说明         |
|:---------------|:-----------------|:-------------|
| gotenberg_data | /var/task/data   | 临时工作目录 |
| gotenberg_logs | /var/task/logs   | 日志目录     |

### 管理命名卷

```bash
# 查看卷列表
docker volume ls | grep gotenberg

# 查看卷详情
docker volume inspect gotenberg_data

# 删除卷（需要先停止服务）
docker compose down --volumes
```

## 健康检查

```bash
# 检查服务是否正常运行
curl http://localhost:3434/health
```

## 注意事项

- 首次启动需要下载镜像，约 500MB
- 建议在生产环境中配置适当的超时时间
- 大文件转换可能需要较长时间

## 官方文档

更多配置选项请参考 [Gotenberg 官方文档](https://gotenberg.dev/)
