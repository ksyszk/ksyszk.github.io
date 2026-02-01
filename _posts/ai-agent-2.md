---
layout: post
title: "AI智能体学习笔记"
date: 2026-02-01
categories: [ai-agent]
---

# AI智能体学习笔记 (二)：工具调用与任务处理模式

在本篇笔记中，我们将深入探讨 AI 智能体执行任务的四种主流模式：自然语言描述、Function Calling、ReAct 以及 Skills。

--------------------------------------------------------------------------------
## 1. 自然语言描述 (Natural Language Description)
这是最原始、最简单的模式。开发者直接在系统提示词 (System Prompt) 中用普通话或英语告诉 AI 有哪些工具可用，以及应该以什么格式输出。

- 特点：编写门槛极低，只要 AI 能读懂即可。

- 缺点：由于大模型本质是概率模型，AI 可能会由于理解偏差输出错误的格式，导致程序无法解析，可靠性较低。

- **示例：**

  ```python
  system_prompt = """
  你是一个天气助手，可以查询天气信息。
  
  你可以使用的工具：
  1. 查询天气：输入城市名称，返回天气信息。你需要按照以下格式输出：
     动作：查询天气
     城市：[城市名称]
  
  2. 查询空气质量：输入城市名称，返回AQI指数。你需要按照以下格式输出：
     动作：查询空气质量
     城市：[城市名称]
  
  用户现在的问题是：{user_query}
  请根据问题决定使用哪个工具，并严格按照上述格式输出。
  """
  
  # 用户查询
  user_query = "北京今天天气怎么样？"
  
  # AI可能的回复（不稳定的情况）
  # 理想情况：
  """
  动作：查询天气
  城市：北京
  """
  
  # 但AI也可能回复：
  """
  用户想查询北京天气，我应该使用查询天气工具。
  动作：查询天气
  城市：北京市
  """
  # 注意："北京市"可能无法被解析，因为程序期待的是"北京"
  ```

  


## 2. 函数调用 (Function Calling)
为了解决自然语言描述的不稳定性，各大模型厂商（如 OpenAI, Claude）推出了 Function Calling。它将工具描述标准化，不再混杂在系统提示词中，而是放在独立的 JSON 字段里。

- 特点：格式统一、描述标准。AI 专门针对这种 JSON 格式进行过训练，调用准确度极高。

- 优势：如果 AI 输出格式有误，服务器端可以自动检测并重试，开发者无需手动编写重试逻辑，节省 Token 成本。

- **示例：**

  ```python
  import openai
  import json
  
  # 定义工具（函数）的JSON Schema
  tools = [
      {
          "type": "function",
          "function": {
              "name": "get_weather",
              "description": "获取指定城市的天气信息",
              "parameters": {
                  "type": "object",
                  "properties": {
                      "location": {
                          "type": "string",
                          "description": "城市名称，如：北京、上海"
                      },
                      "unit": {
                          "type": "string",
                          "enum": ["celsius", "fahrenheit"],
                          "description": "温度单位，默认为摄氏度"
                      }
                  },
                  "required": ["location"]
              }
          }
      },
      {
          "type": "function",
          "function": {
              "name": "get_aqi",
              "description": "获取空气质量指数",
              "parameters": {
                  "type": "object",
                  "properties": {
                      "location": {
                          "type": "string",
                          "description": "城市名称"
                      }
                  },
                  "required": ["location"]
              }
          }
      }
  ]
  
  # 用户消息
  messages = [
      {"role": "user", "content": "北京和上海的天气怎么样？"}
  ]
  
  # 调用API
  response = openai.chat.completions.create(
      model="gpt-4",
      messages=messages,
      tools=tools,
      tool_choice="auto"  # 让模型自动决定是否调用工具
  )
  
  # 解析响应
  response_message = response.choices[0].message
  
  # 检查是否有工具调用
  if response_message.tool_calls:
      print("AI决定调用工具：")
      for tool_call in response_message.tool_calls:
          function_name = tool_call.function.name
          function_args = json.loads(tool_call.function.arguments)
          
          print(f"工具名称：{function_name}")
          print(f"参数：{function_args}")
          
          # 这里你会实际调用相应的函数
          if function_name == "get_weather":
              location = function_args.get("location")
              unit = function_args.get("unit", "celsius")
              # 调用实际的天气API
              weather_data = call_weather_api(location, unit)
              
              # 将结果返回给AI继续对话
              messages.append(response_message)
              messages.append({
                  "role": "tool",
                  "tool_call_id": tool_call.id,
                  "content": json.dumps(weather_data)
              })
  ```

  

    AI的响应会是这样的结构化格式：

  ```json
  {
    "id": "chatcmpl-xxx",
    "choices": [{
      "index": 0,
      "message": {
        "role": "assistant",
        "content": null,
        "tool_calls": [{
          "id": "call_abc123",
          "type": "function",
          "function": {
            "name": "get_weather",
            "arguments": "{\"location\": \"北京\", \"unit\": \"celsius\"}"
          }
        }, {
          "id": "call_def456",
          "type": "function",
          "function": {
            "name": "get_weather",
            "arguments": "{\"location\": \"上海\", \"unit\": \"celsius\"}"
          }
        }]
      }
    }]
  }
  ```



## 3. ReAct (Reasoning + Acting)
ReAct 是一种思维模式，它让 AI 像人类一样“边思考边行动”。AI 不会一次性给结果，而是按照 “思考 (Reasoning) -> 行动 (Acting) -> 观察 (Observation)” 的循环运行。

- 特点：适用于需要多步拆解的复杂任务。AI 在每一步都会根据上一步的观察结果决定下一步做什么。

-  **示例：**

  用户：我想做千层面，但家里没有Ricotta奶酪，怎么办？

  AI的思考过程：

  ```
  思考1: "用户想做千层面但缺关键食材Ricotta奶酪"
  行动1: 搜索"Ricotta奶酪在千层面的作用"
  观察1: "Ricotta提供奶油口感，是传统配料"
  
  思考2: "需要寻找合适的替代品"
  行动2: 搜索"Ricotta奶酪替代品 烹饪"
  观察2: "可用Cottage Cheese、Mascarpone或混合酸奶替代"
  
  思考3: "需要考虑口感和烹饪效果差异"
  行动3: 搜索"千层面用Cottage Cheese替代效果"
  观察3: "口感略不同但可接受，需减少水分"
  
  思考4: "给用户具体建议"
  最终回答: "可以用Cottage Cheese替代，建议..."
  ```

## 4. Skills
Skill 是管理大规模提示词和工具的进阶方案。它将特定的 Prompt、脚本和工具打包成一个文件夹，通过动态加载来解决 Token 浪费和 AI 迷茫的问题。

### 组织结构：文件夹式管理
  每个 Skill 都是一个独立的物理文件夹，支持嵌套关系，以便 AI 深度探索。

  - Skill 文件夹结构示例：

    ```
    📂 cooking-assistant-skills/
    │
    ├── 📁 recipe-finder/          # 菜谱发现技能
    │   ├── 📄 skill.md           # 技能说明文件，包含Metadata和Instruction
    │   │   ├─ 能力：按食材找菜谱
    │   │   ├─ 工具：搜索、筛选、排序
    │   │   └─ 数据：引用外部文件
    │   ├── 📁 templates/         # 模板文件夹
    │   │   ├─ 📄 italian.md     # 意大利菜模板
    │   │   └─ 📄 french.md      # 法国菜模板
    │   └── 📄 common_pairs.json # 常见食材搭配数据
    │
    ├── 📁 meal-planner/          # 餐饮规划技能
    │   ├── 📄 skill.md
    │   └── 📄 nutrition_guide.md
    │
    └── 📁 grocery-helper/        # 购物助手技能
        ├── 📄 skill.md
        └── 📄 price_checker.py   # 价格检查脚本
    ```

### 核心工作流
Skill 的运行分为三个关键阶段：发现、激活、执行。

| 阶段              | 动作说明 | 交互流程                                                     |
| ----------------- | -------- | ------------------------------------------------------------ |
| 发现 (Discovery)  | 简介分发 | 客户端收集所有 Skill 文件夹中的 Metadata，连同 User Prompt 发给大模型。AI 此时仅知道有哪些技能可用。 |
| 激活 (Activation) | 全量读取 | AI 通过语义理解判断当前任务需要哪个 Skill，发出特殊指令（如执行 cat recipe-finder/skill.md）要求读取完整内容。 |
| 执行 (Execution)  | 深度交互 | 客户端将完整的指令和关联文件传给 AI。AI 根据指令给出回复，甚至可能动态编写脚本或直接运行文件夹内的程序来解决问题。 |

**示例：**

用户请求：帮我规划一周的健康晚餐，要低卡路里、快速烹饪

```
第1步：技能发现
AI看到技能清单：
1. recipe-finder（找菜谱） ✓ 需要
2. meal-planner（规划餐饮） ✓ 需要  
3. nutrition-calculator（计算营养） ✓ 需要
4. grocery-shopper（购物） ✗ 暂时不需要

第2步：技能激活
AI请求："加载meal-planner技能的全部内容"
→ 系统读取skill.md文件
→ 发现需要关联nutrition-guide.md
→ 一并加载所有相关文件

第3步：技能执行
AI使用加载的技能：
• 用meal-planner规划每日菜谱
• 用recipe-finder寻找具体菜谱
• 用nutrition-calculator确保低卡路里
• 整合结果生成一周晚餐计划
```



### 核心优势

- 无限扩展：Skill 可以引用其他 md 文件，形成无限嵌套的知识网络，AI 会根据需要逐层读取。
- 行动合一：配合 Execution 能力，AI 不仅能读，还能直接调用电脑上的函数库或工具（如处理 PDF 的 Python 库）完成实操任务。
