# 整体流程


main（） 
--> asyncio.run # 启动主线程 event loop
--> 获取 prompt，await agent.run

## Agent 继承关系

Manus Agent <--- BrowserAgent <--- ToolCallAgent <-- ReActAgent <--BaseAgent  

### BaseAgent 继承 pydantic BaseModel + ABC
属性： 
name/description

system_prompt
next_step_prompt

依赖的模块： 
llm - wrapper of external LLM model, 处理与配置的模型的交互，重要方法： ask， ask_with_images, ask_tool, token_count...
max_steps/curent_step 
memory - 存放输入的prompt信息
state - 状态： idle, running, finished, error

重要的方法： 
async def run(...) 
---> check status --> update moemory --> 启动 async loop： for each steps in steps, call self.step()

async def step:  需要子类override的方法

is_stuck: 检查是否陷入stuck

handle_stuck - Handle stuck state by adding a prompt to change strategy 需要进一步的研究。。。

## ReActAgent 继承BaseAgent + ABC

重要的方法： 

think() - abstract方法，处理当前状态，决定下一步 action， 需要子类重载

act （） - abstract方法，执行 action，需要子类重载

step （） - 非抽象方法，模板方法调用 think 来决定是否执行动作，然后 call self.act

## ToolCallAgent 继承 ReActAgent

属性: 
available_tools 获取可用的工具： chatCompletion, Terminate
tool_calls 

方法： 
think - 构建 prompt，然后 从 llm获取 可用的tool 以及参数项
act - 执行 tool call 并获取结果，添加到 memory 
execute_tool/_handle_special_tool - helper 函数执行 tool 调用

## BrowserAgent 继承 ToolCallAgent

属性：
available_tools - BrowserUser, Terminate

方法：
get_browser_state - 从 browser-use 工具获取状态
think -
--- 获取 browser-use的状态信息，如： url, title, tabs, screenshot; 
--- 获取下一个prompt
--- 调用父类的 think

## ManusAgent继承BrowserAgent 
属性：
available_tools - PythonExecute, BrowserTool, StrReplaceEditor, Teminate

方法： 
-- think :  检查最后3条在memory的browser activity，构建next prompt， call 父类的think

  
  


