
---
 本教程将详细介绍如何利用 Cloudflare 强大的全球边缘网络，免费搭建一套支持 **VLESS / Trojan / Shadowsocks** 的高性能代理服务。

相比市面上仅支持 VLESS 的单一脚本，本项目的核心优势在于其**多协议支持**与**高度可定制性**，能够完美适配各种复杂的网络环境。

### 核心优势一览

- **⚡️ 零成本部署**：基于 Cloudflare Workers/Snippets，利用其免费额度即可使用，无需购买 VPS。
    
- **🔄 三合一协议**：
    
- 同时支持 **VLESS、Trojan、Shadowsocks**。
    
- 支持 **VLESS + Trojan** 双协议共存，灵活切换，抗干扰能力更强。
    
- **🌍 高性能与稳定性**：
    
- 利用 Cloudflare 全球边缘网络，自带负载均衡与故障转移。
    
- **自定义 ProxyIP**：轻松解决 CF IP 被阻断或访问慢的问题。
    
- **📱 全平台兼容**：一键生成通用订阅链接，完美支持 Shadowrocket, Clash, v2rayN, Sing-box 等主流客户端。
    
- **🛡 安全便捷**：支持仪表盘（Dashboard）密码保护，防止配置泄露。
    

---

### 🛠 第一步：获取项目代码

首先，我们需要获取核心的部署脚本。

1. 访问项目官方地址：**[点击打开项目主页]** 
    
2. 在文件列表中找到 `_worker.js` 或发布页面的下载选项。
    
3. **下载/复制**该脚本的全部代码内容（通常选择第一个下载选项）。
    

---

### ☁️ 第二步：部署至 Cloudflare Workers

请确保你已经拥有一个 Cloudflare 账号。如果没有，请先注册。

1. **登录 Cloudflare**：点击进入 [Cloudflare 官网](https://dash.cloudflare.com/) 并登录。
    
2. **进入 Workers 面板**：
    

- 在左侧菜单栏点击 **"Workers & Pages" (Workers 和 Pages)**。
    
- 点击右上角的 **"Create Application" (创建应用程序)**。
    
- 点击 **"Create Worker" (创建 Worker)**。
    

3. **初始化 Worker**：
    

- 你可以自定义 Worker 的名称（例如 `my-proxy-node`），也可以保持默认。
    
- 点击 **"Deploy" (部署)**。此时你会得到一个默认的 Hello World 程序。
    

4. **编辑代码**：
    

- 点击 **"Edit code" (编辑代码)** 进入在线编辑器。
    
- **清空** 编辑器中原有的所有代码。
    
- **粘贴** 你在第一步中下载/复制的 `_worker.js` 完整代码。
    

5. **保存并部署**：
    

- 点击编辑器右上角的 **"Deploy" (部署)** 按钮，再次确认部署。
    

---

### ⚙️ 第三步：环境配置与 UUID 设置

为了让节点能够安全运行，我们需要配置唯一的 UUID（用户 ID）。

1. **生成 UUID**：
    

- 你需要一个 UUID 作为连接密码。你可以在电脑终端输入 `uuidgen` 生成，或者访问在线 UUID 生成网站获取一个。
    
- *示例 UUID：`84307221-502a-4467-9c56-655677889900*`
    

2. **修改配置 (方式一：直接修改代码)**：
    

- 在代码顶部找到 `userID` 或 `UUID` 相关的变量声明。
    
- 将默认值替换为你生成的 UUID。
    

3. **修改配置 (方式二：环境变量 - 推荐)**：
    

- 返回 Cloudflare Worker 的详细设置页面。
    
- 点击 **"Settings" (设置)** -> **"Variables" (变量)**。
    
- 添加变量：
    
- **变量名**：`UUID`
    
- **值**：`你的UUID字符串`
    
- _(可选)_ 添加 `PROXYIP` 变量来指定优选 IP，或添加 `PASSWORD` 变量来设置访问面板的密码。
    
- 保存后，回到 "Deployments" 重新部署一次以生效。
    

---

### 🔗 第四步：获取订阅与客户端连接

部署完成后，你将获得一个专属的 Worker 域名（例如 `https://your-worker.workers.dev`）。

1. **访问管理面板**：
    

- 在浏览器地址栏输入：`https://你的域名/你的UUID`
    
- *例如：`https://my-proxy-node.workers.dev/84307221-502a-4467-9c56-655677889900*`
    

2. **获取节点订阅**：
    

- 进入页面后，你将看到 VLESS、Trojan、Shadowsocks 的节点信息。
    
- 页面通常提供 **"一键复制订阅链接"** 或 **"Clash/Sing-box 配置下载"**。
    

3. **导入客户端**：
    

- **iOS (Shadowrocket/Stash)**：点击右上角 `+` 号 -> 类型选择 `Subscribe` -> 粘贴订阅链接。
    
- **Windows (v2rayN/Clash)**：直接添加订阅地址并更新。
    
- **Android (v2rayNG/Surfboard)**：同上。
    

> **💡 提示**：该项目支持自动化配置，无需手动填写复杂的加密方式或传输协议，导入订阅即可使用。

---

### 🚀 第五步：测试与验证

导入节点后，请进行以下测试：

1. **真连接延迟测试**：在客户端中点击“连通性测试”或“Tcping”。如果显示具体的毫秒数（如 150ms），说明节点搭建成功。
    
2. **速度体验**：
    

- 打开 YouTube 4K 视频进行测试。
    
- 由于 Cloudflare 的机制，部分地区的非优选 IP 可能延迟较高，但带宽通常非常充足。
    

---

### 🌟 进阶优化：自定义 ProxyIP

如果遇到节点无法连接或速度慢，通常是因为 Cloudflare 的出口 IP 被目标网站（如 Google）暂时屏蔽或限速。

- **解决方案**：在 Worker 的环境变量中设置 `PROXYIP`。
    
- **参数支持**：你可以填入专门筛选过的 **优选 IP** 或 **优选域名**。这能显著降低延迟并提高连接成功率。
    

---

希望这篇教程能帮你顺利搭建属于自己的多协议高速节点！

**Would you like me to explain how to find and configure a "Best ProxyIP" (优选 IP) to further improve your connection speed?**![](assets/全能三合一：基于%20Cloudflare%20Workers%20的免费多协议节点搭建指南/file-20260131224103863.png)

