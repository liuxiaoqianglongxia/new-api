# relay/channel — 上游 Provider 适配器

37 个 AI provider 适配器，将各厂商 API 转换为统一 OpenAI 兼容格式。

## OVERVIEW

每个 `relay/channel/<provider>/` 实现 `Adaptor` 或 `TaskAdaptor` 接口，负责：请求转换 → 上游调用 → 响应转换 → 用量提取。

## STRUCTURE

```
relay/channel/
├── adapter.go          # Adaptor / TaskAdaptor 接口定义 + 通道注册表
├── api_request.go      # 通用 HTTP 请求封装 (DoAPIRequest)
├── openai/             # 参考实现 — OpenAI 兼容适配器
│   ├── adaptor.go      # 接口实现
│   ├── relay-openai.go # Chat Completions 主逻辑
│   ├── relay_image.go  # 图片生成
│   ├── relay_realtime.go # Realtime API
│   └── relay_responses.go # Responses API
├── claude/             # Anthropic Claude (Messages API)
├── gemini/             # Google Gemini
├── aws/                # AWS Bedrock
├── azure/              # Azure OpenAI (通过 relay/aztorelay/)
├── deepseek/           # DeepSeek
├── ...                 # 其余 30+ provider
└── submodel/           # 子模型路由 (特殊 — 内部 fan-out)
```

## WHERE TO LOOK

| 任务 | 文件/目录 | 说明 |
|------|----------|------|
| 添加新 provider | 新建 `<provider>/adaptor.go` | 实现 `Adaptor` 接口 |
| 注册新通道 | `adapter.go` → `GetAdaptor()` | 添加 case 分支 |
| 通道类型常量 | `constant/channel.go` | `ChannelType*` + `ChannelTypeNames` |
| StreamOptions 支持 | `relay/common/relay_info.go` → `streamSupportedChannels` | 支持流式选项的通道白名单 |
| 通用 HTTP 请求 | `api_request.go` | `DoAPIRequest()` — 大部分 provider 复用 |
| OpenAI 兼容参考 | `openai/` | 最完整的适配器实现，新 provider 首选参考 |
| Claude 参考 | `claude/` | Messages API 格式适配参考 |
| Gemini 参考 | `gemini/` | Gemini 原生格式适配参考 |
| 中间请求转换 | `relay/relay_task.go` | TaskAdaptor 类型的调度逻辑 |

## 添加新 Provider 步骤

1. 创建 `relay/channel/<provider>/adaptor.go`，实现 `Adaptor` 接口：
   ```
   Init()          — 初始化 (从 RelayInfo 读取配置)
   GetRequestURL() — 构造上游请求 URL
   SetupRequestHeader() — 设置请求头 (API Key 等)
   ConvertOpenAIRequest() — OpenAI → 上游格式
   DoRequest()     — 发送请求 (通常复用 api_request.go)
   DoResponse()    — 解析响应、提取 usage
   GetModelList()  — 返回支持的模型列表
   GetChannelName() — 返回通道名称字符串
   ```
2. 若需支持其他 relay 模式，实现对应 `Convert*Request()` 方法
3. 在 `adapter.go` 的 `GetAdaptor()` 中注册
4. 在 `constant/channel.go` 添加 `ChannelType*` 常量和 `ChannelTypeNames` 映射
5. 若支持 StreamOptions，在 `relay/common/relay_info.go` 的 `streamSupportedChannels` 中添加

## 适配器模式

```go
// 核心接口 (adapter.go)
type Adaptor interface {
    Init(info *RelayInfo)
    GetRequestURL(info *RelayInfo) (string, error)
    SetupRequestHeader(c *gin.Context, req *http.Header, info *RelayInfo) error
    ConvertOpenAIRequest(c *gin.Context, info *RelayInfo, request *dto.GeneralOpenAIRequest) (any, error)
    DoRequest(c *gin.Context, info *RelayInfo, requestBody io.Reader) (any, error)
    DoResponse(c *gin.Context, resp *http.Response, info *RelayInfo) (usage any, err *types.NewAPIError)
    GetModelList() []string
    GetChannelName() string
    // 可选：其他格式转换
    ConvertClaudeRequest(...)
    ConvertGeminiRequest(...)
    ConvertRerankRequest(...)
    ConvertEmbeddingRequest(...)
    ConvertAudioRequest(...)
    ConvertImageRequest(...)
    ConvertOpenAIResponsesRequest(...)
}

type TaskAdaptor interface {
    Init(info *RelayInfo)
    ValidateRequestAndSetAction(c *gin.Context, info *RelayInfo) *dto.TaskError
    EstimateBilling(c *gin.Context, info *RelayInfo) map[string]float64
    AdjustBillingOnSubmit(c *gin.Context, info *RelayInfo) map[string]float64
    FetchTaskStatus(c *gin.Context, info *RelayInfo, taskID string) (*dto.TaskStatus, *dto.TaskError)
    DoResponse(c *gin.Context, resp *http.Response, info *RelayInfo) (usage any, err *types.NewAPIError)
    GetChannelName() string
}
```

`TaskAdaptor` 用于异步任务类 provider (Midjourney, Suno, 视频生成等)，额外支持计费和任务状态轮询。

## CONVENTIONS

- **adaptor.go** 是每个 provider 的入口文件，必须包含结构体 + `Init()` 方法
- 通用 HTTP 请求使用 `api_request.DoAPIRequest()`，不要自己封装 HTTP client
- Usage 提取在 `DoResponse()` 中完成，返回 `*types.Usage` 结构
- 错误处理返回 `*types.NewAPIError`，不要 panic
- 模型列表在 `GetModelList()` 中硬编码或从配置读取
- Channel name 使用 `GetChannelName()` 返回，与 `ChannelTypeNames` 保持一致

## ANTI-PATTERNS

- **不要** 在 adaptor 中直接导入 `encoding/json` — 使用 `common.Marshal`/`common.Unmarshal`
- **不要** 在 adaptor 中直接操作数据库 — 通过 `service` 层
- **不要** 忘记注册到 `adapter.go` 的 `GetAdaptor()` — 否则通道无法使用
- **不要** 忘记添加 `ChannelType` 常量和名称映射到 `constant/channel.go`
- **不要** 用非指针标量 + `omitempty` 定义 relay DTO 可选字段 — 必须用指针类型 (`*int`, `*bool`)

## 流式处理

流式响应的处理路径不同：`DoResponse()` 中判断 `info.IsStream`，走 SSE 逐行解析。
非流式直接 `json.Unmarshal` 完整响应体。

## 参考实现优先级

| 新 Provider 类型 | 参考实现 |
|-----------------|---------|
| OpenAI 兼容 upstream | `openai/` (最完整) |
| Claude Messages | `claude/` |
| Google Gemini | `gemini/` |
| 异步任务 (图片/视频) | `midjourney/`, `suno/`, `jimeng/` |
| 自定义格式 | `baidu_v2/`, `zhipu/` |
| AWS Bedrock | `aws/` |
