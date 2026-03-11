# Subotiz Plugin

Subotiz 开发工具 Cursor 插件，提供支付、订阅管理等能力，方便在 Cursor 中接入 Subotiz 相关能力。

## 功能

- **支付**：与 Subotiz 支付能力集成
- **订阅管理**：订阅相关开发与调试支持
- **MCP 集成**：通过 MCP（Model Context Protocol）在对话中调用 Subotiz 工具

## 安装

1. 在 Cursor 中打开 **Settings → Cursor Settings → Plugins**
2. 选择「从本地安装」或按 Cursor 插件文档添加本插件目录
3. 启用 **subotiz-plugin**

安装并启用后，Subotiz MCP 工具会在对话中可用；若服务需要认证，按提示调用 `mcp_auth` 完成认证即可使用。

## 配置

- **插件元数据**：`/.cursor-plugin/plugin.json`（名称、版本、描述、关键词、Logo 等）
- **MCP 配置**：`/.mcp.json`（若需配置 MCP 服务器可在此填写）

## 作者

**Subotiz** · weihanlai@shoplazza.com

## 许可证

请以项目根目录或仓库中声明的许可证为准。
