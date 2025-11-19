# PaperMirror 部署与安全指南

本项目经过深度优化，采用 **纯前端架构 (Client-Side Architecture)**。这意味着您不需要部署任何后端服务器（如 Node.js 或 Python），所有逻辑都直接在用户的浏览器中运行。

---

## ❓ 为什么移除 Vercel 后端？

您可能在寻找后端代码（如 `api/proxy.ts`），但本项目**有意移除了它**。

*   **原因**：Vercel (Hobby Plan) 的 Serverless 函数有 **10秒** 的硬性超时限制。
*   **冲突**：PaperMirror 使用的 **Gemini 2.5 / 3.0 Pro** 模型在进行“思考模式”和长文本重写时，响应时间通常在 **30秒到 90秒** 之间。
*   **后果**：如果使用后端代理，请求会因为超时被强制切断 (504 Gateway Timeout)，导致应用不可用。

**结论：前端直连 Google API 是目前免费处理长文本 AI 任务唯一稳定的方案。**

---

## 🛡️ 核心安全警告：如何保护 API Key

由于是纯前端应用，**API Key 最终会被打包进 JavaScript 文件中**，用户通过浏览器开发者工具可以看到它。

**但是，只要配置了 Referrer 限制，这依然是安全的。**

### 安全原理
这就好比您把车钥匙（API Key）放在了车顶上，但是给车装了“地理围栏”（HTTP Referrer）。只有在您家车库（您的域名）里，这把钥匙才能发动汽车。黑客偷走了钥匙，在他自己的电脑上是无法使用的。

---

## 🚀 GitHub Pages 部署步骤

### 1. 获取 API Key
访问 [Google AI Studio](https://aistudio.google.com/app/apikey) 获取 Key。

### 2. 配置安全限制 (必做!)
在 Google AI Studio 的 API Key 设置页面，找到 **Client restriction** 或 **Website restrictions**：
1.  点击 **Add**。
2.  添加本地开发地址：`http://localhost:5173`
3.  添加生产环境地址：`https://<您的GitHub用户名>.github.io`
4.  保存。

### 3. 设置 GitHub Secrets
为了让 Vite 在构建时能注入 Key：
1.  Fork 本仓库到您的 GitHub。
2.  进入仓库 **Settings** -> **Secrets and variables** -> **Actions**。
3.  点击 **New repository secret**。
    *   Name: `API_KEY`
    *   Value: 您的 API Key 字符串 (例如 `AIzaSy...`)

### 4. 自动部署
1.  进入仓库的 **Actions** 标签页。
2.  选择左侧的 **Deploy to GitHub Pages** 工作流。
3.  点击 **Run workflow**。

等待几分钟后，您的应用将在 `https://<您的用户名>.github.io/paper-mirror/` 上线运行。

---

## ⚠️ 常见错误排查

*   **Error 403 (Permission Denied)**: 
    *   检查 Google AI Studio 中的 URL 限制是否填写正确。注意 `https` 和结尾的 `/`。
    *   确保 GitHub Secrets 中的 Key 没有多余空格。
*   **Error 404 (Not Found) / White Screen**:
    *   确保 `vite.config.ts` 中的 `base: './'` 配置存在。
*   **JSON Parsing Error**:
    *   这是模型偶尔输出了非标准格式。请重试，或检查控制台日志。