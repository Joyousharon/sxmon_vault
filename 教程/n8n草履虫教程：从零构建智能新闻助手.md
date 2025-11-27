
本文档是《n8n教程》的配套文档，包含构建智能新闻助手所需的完整代码和提示词。通过本教程，您将学会如何使用n8n构建一个能够自动收集新闻、分析日程并提供智能建议的AI助手。

---

## 🎯 项目概述

构建一个"新闻报通"AI助手，能够：
- 自动收集AI领域最新新闻
- 分析您的日程安排
- 智能推荐感兴趣的内容和活动安排
- 通过飞书机器人推送个性化建议

---

## 📋 前置准备

### 1. 创建新闻收集多维表格
在飞书多维表格中创建包含以下6个字段的表单：
```
1. 新闻标题
2. 发布日期  
3. 发布媒体
4. 核心内容/摘要
5. 原文链接
6. 创建时间
```

---

## 🛠️ 环境搭建

### 第一步：安装Docker & n8n

**工具链接：**
- Docker官网：https://www.docker.com/
- n8n官网：https://n8n.io/

**终端命令：**

```bash
# 下载n8n
docker volume create n8n_data
docker run -d --name n8n --restart unless-stopped -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n

# 如果网络环境有问题，使用以下命令：
docker volume create n8n_data  
docker run -d --name n8n --restart unless-stopped -p 5678:5678 -v n8n_data:/home/node/.n8n n8nio/n8n

# 停止n8n
docker stop n8n

# 启动n8n
docker start n8n
```

安装完成后，访问 http://localhost:5678 即可使用n8n。

---

## 🔧 工作流构建

### 第二步：创建项目并设置触发器

1. 在n8n中创建新工作流
2. 添加"Schedule Trigger"（计划触发器）
3. 配置触发时间（如每天上午9点）

### 第三步：添加Agent节点

在工作流中添加：
- 选择 `AI` → `Agent` 节点

### 第四步：配置AI大脑

**DeepSeek API配置：**
1. 访问 [DeepSeek开放平台](https://platform.deepseek.com/usage)
2. 获取API密钥
3. 在Agent节点中配置API凭证

### 第五步：记忆配置（本次跳过）

本项目中不需要配置长期记忆功能。

---

## 🔌 工具集成

### 第六步：配置飞书集成

**1. 安装飞书节点包**
```bash
n8n-nodes-feishu-lite
```

**2. 配置飞书凭证**
- 访问 [飞书开放平台](https://open.feishu.cn/?lang=zh-CN)
- 进入[开发者后台](https://open.feishu.cn/app)创建应用
- 获取 `appid` 和 `appsecret`

**3. 查询多维表格数据**

获取表格token和ID后，使用以下筛选条件查询最近两天的新闻：

```json
{
  "filter": {
    "conjunction": "or",
    "conditions": [
      {
        "field_name": "创建时间",
        "operator": "is", 
        "value": ["Today"]
      },
      {
        "field_name": "创建时间",
        "operator": "is",
        "value": ["Yesterday"]
      }
    ]
  }
}
```

**4. 获取飞书日历日程**

获取日历ID后，使用以下时间戳查询未来7天日程：

- **开始时间**（今天）：
```
{{ (Math.floor((Math.floor(Date.now()/1000) + 28800) / 86400) * 86400 - 28800) }}
```

- **结束时间**（7天后）：
```
{{ (Math.floor((Math.floor(Date.now()/1000) + 28800) / 86400) * 86400 - 28800) + 604800 + 86399 }}
```

**重要：** 记得创建并发布机器人，版本号按格式填写（如：0.0.1）

---

## 🤖 消息推送配置

### 第七步：配置飞书群聊机器人

**1. 添加webhook机器人**
- 在飞书群聊中添加自定义webhook机器人

**2. 配置HTTP请求**
- **请求头：**
  - `Content-Type: application/json`

- **请求体（卡片消息）：**
```json
{
  "msg_type": "interactive",
  "card": {
    "schema": "2.0",
    "config": {
      "update_multi": true,
      "style": {
        "text_size": {
          "normal_v2": {
            "default": "normal",
            "pc": "normal",
            "mobile": "heading"
          }
        }
      }
    },
    "body": {
      "direction": "vertical",
      "padding": "12px 12px 12px 12px",
      "elements": [
        {
          "tag": "markdown",
          "content": "{{ $json.output ? JSON.stringify($json.output).slice(1, -1) : '' }}",
          "text_align": "left",
          "text_size": "normal_v2",
          "margin": "0px 0px 0px 0px"
        }
      ]
    },
    "header": {
      "title": {
        "tag": "plain_text",
        "content": "AI News"
      },
      "subtitle": {
        "tag": "plain_text", 
        "content": ""
      },
      "template": "blue",
      "padding": "12px 12px 12px 12px"
    }
  }
}
```

---

## 🧠 Agent提示词设计

### 第八步：编写智能提示词

**提示词核心要素：**
- 🎭 **角色定义** - 明确AI的身份和专业领域
- 🎯 **任务目标** - 清晰描述要完成的任务
- 🛠️ **工具说明** - 列出可用的工具和功能
- 📝 **约束条件** - 设定行为规则和限制
- 📊 **输出格式** - 明确最终输出的格式要求

**完整提示词：**

```
你是我的专属AI助理"新闻报通"！你的使命是帮我洞察最新的AI动态，并结合我的工作日程，智能推荐感兴趣的内容和安排行程。在没有行业大事发生时，你也会关心我的生活，推荐放松娱乐活动。

最终你需要将所有分析和建议，整合为一个适合在飞书卡片中展示的Markdown格式文本块。保持乐观、敏锐、有创造力！

我有两个核心工具供你调遣：
- **news**：获取过去2天内飞书多维表格中的AI新闻，包含标题、日期、发布媒体、摘要和原文链接
- **daily**：查看我未来7天的飞书日程安排，包括日期、时间和事件标题

## 🚀 执行流程

### 第一步：信息收集
立即使用工具获取：
1. 最新的AI新闻列表
2. 未来7天的日程安排

### 第二步：智能分析与Markdown输出
生成单一、完整的Markdown文本作为最终输出：

### 🚀 AI圈今日速递与【专属建议】

**🌟 今日AI新闻看板：**
{{#if (tool_output.latest_news an_array_with_items)}}
{{#each tool_output.latest_news as |news_item|}}
* ---
* **标题：** {{news_item.title}}
* **发布日期：** {{news_item.date}} 
* **发布媒体：** {{news_item.source_or_media}}
* **核心摘要：** {{news_item.summary}}
* **原文链接：** [点击查看详情]({{news_item.link}})
{{/each}}
{{else}}
* 今天AI领域风平浪静，暂未捕获到新的AI大新闻。是时候出门活动活动了！
{{/if}}
* ---

**📅 我的近期日程概览：**
[基于工具返回的日程数据，列出未来几天的安排]

**💡 综合建议与排期参考：**
[结合新闻价值和日程空闲情况，给出具体建议：
- 有重要新闻 + 有空闲时间 → 推荐阅读优先级和安排建议
- 一般新闻 + 有空闲时间 → 适度关注建议  
- 无新闻 + 有空闲时间 → 休闲活动推荐
- 日程已满 → 专注工作提醒
]

## ⚡ 重要要求
- 输出必须是纯净的Markdown文本，不要包含```标记
- 确保所有信息准确来源于工具输出
- 建议要具体、有建设性，体现信息综合考量
- 语气积极、专业、充满洞察力
```

**注意：** 填写提示词后记得点击"固定"按钮！

---

## 📰 补充：新闻收集工作流

为了确保Agent有新闻可分析，需要建立新闻收集工作流：

1. 在n8n中通过 `Import from file` 导入提供的JSON工作流文件
2. 配置AI模型（参考第四步）
3. 配置飞书凭证和表格信息（参考第六步）

**推荐新闻源：**
- GitHub优质RSS源：https://github.com/weekend-project-space/top-rss-list
- 天行API：https://www.tianapi.com/

---

## 🎉 恭喜完成！

您已经成功学会了：
- ✅ n8n工作流搭建
- ✅ AI Agent构建
- ✅ 飞书集成配置
- ✅ 智能提示词编写

**扩展资源：**
- n8n社区节点：https://github.com/restyler/awesome-n8n
- n8n工作流模板：https://n8n.io/workflows/

现在您的"新闻报通"AI助手已经准备就绪，开始享受智能化的信息管理体验吧！🚀