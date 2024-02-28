[TOC]


### 1-列出可用的模型列表

```html
<script>
import OpenAI from 'openai'

export default {
  data() {
    return {
      openai: new OpenAI({
        apiKey: '...',
        dangerouslyAllowBrowser: true
      })
    }
  },
    
  methods: {
  	// 列出可用的模型列表
    async getModelsList () {
      const list = await this.openai.models.list()
      console.log(list.data)
      // 示例
      // [
      //   {
      //     "id": "gpt-4-vision-preview",
      //     "object": "model",
      //     "created": 1698894917,
      //     "owned_by": "system"
      //   },
      //   {
      //     "id": "dall-e-3",
      //     "object": "model",
      //     "created": 1698785189,
      //     "owned_by": "system"
      //   },
      //   {
      //     "id": "gpt-3.5-turbo-0613",
      //     "object": "model",
      //     "created": 1686587434,
      //     "owned_by": "openai"
      //   },
      //   {
      //     "id": "text-embedding-3-large",
      //     "object": "model",
      //     "created": 1705953180,
      //     "owned_by": "system"
      //   },
      //   {
      //     "id": "gpt-3.5-turbo-instruct-0914",
      //     "object": "model",
      //     "created": 1694122472,
      //     "owned_by": "system"
      //   },
      //   {
      //     "id": "dall-e-2",
      //     "object": "model",
      //     "created": 1698798177,
      //     "owned_by": "system"
      //   }
      //   ...
      // ]
    }
  }
}
</script>
```







### 2-同步通讯

```html
<script>
import OpenAI from 'openai'

export default {
  data() {
    return {
      openai: new OpenAI({
        apiKey: '...',
        dangerouslyAllowBrowser: true
      })
    }
  },
    
  methods: {
  	// 同步通讯
    async syncCommunicate() {
      const completion = await this.openai.chat.completions.create({
        messages: [{ role: 'user', content: '写一篇100字左右关于春天的作文' }],
        model: 'gpt-4-turbo-preview',
      })

      console.log(completion?.choices[0]?.message?.content || '')
      // 示例
      // '春天，是一年之中最充满生机的季节。万物复苏，斑斓的色彩重新涂抹大地。明媚的阳光洒落，暖洋洋地拥抱着每一寸土地，让人心情舒畅。树木抽出嫩绿的新芽，花朵竞相开放，散发出阵阵芬芳，吸引着蜜蜂与蝴蝶翩翩起舞。小溪也开始欢快地唱歌，伴随着鸟儿清脆悦耳的鸣叫声，为春天奏响生命的赞歌。在这个生机盎然的季节里，人们的心情也变得轻松愉悦，充满了对美好未来的向往与期待。春天，是希望的季节，是新生的象征。'
    }
  }
}
</script>
```







### 3-回流通讯

```javascript
// 回流通讯 || 异步通讯
async streamCommunicate() {
  const stream = await this.openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{ role: 'user', content: '写一篇100字左右关于春天的作文' }],
    stream: true
  })

  for await (const chunk of stream) {
    console.log(chunk.choices[0]?.delta?.content || '')
    // 示例
    // '春'
    // '天'
    // ','
    // '是'
    // '一个'
    // '万'
    // '物'
    // '复'
    // '苏'
    // ...
  }
}
```







### 4-上传文件

```html
<!--
	I.单个文件的大小最大为 512MB
	II.一个组织上传的所有文件大小最大为 100 GB
	III.支持的文件种类: 参考"附录"
-->
<template>
  <div>
  	<input type="file" @change="handleFileChange" />
  </div>
</template>

<script>
import OpenAI from 'openai'
    
export default {
  data() {
    return {
      openai: new OpenAI(...)
    }
  },
  
  methods: {
  	async handleFileChange(e) {
      if (e.target.files.length !== 0) {  // 用户选择了文件
        
        // 获取文件对象
        let targetFile = e.target.files[0]

        // 上传具有“助手”用途的文件
        const file = await this.openai.files.create({
          file: targetFile,
          purpose: 'assistants'
        })
        
        console.log(file)
	    // 示例
        // {
        //    "object": "file",
        //    "id": "file-XuqA6ts1Y9EN9ZIzBNZ6IpOI",
        //    "purpose": "assistants",
        //    "filename": "精通正则表达式（第3版）]中文版.pdf",
        //    "bytes": 47404541,
        //    "created_at": 1708930855,
        //    "status": "processed",
        //    "status_details": null
        // }
      }
    }
  }
</script>
```









### 5-列出文件列表

```javascript
async getFilesList () {
	const list = await this.openai.files.list()
	console.log(list.data)
    // 示例
    // [
    //   {
    //     "object": "file",
    //     "id": "file-XuqA6ts1Y9EN9ZIzBNZ6IpOI",
    //     "purpose": "assistants",
    //     "filename": "精通正则表达式（第3版）中文版.pdf",
    //     "bytes": 47404541,
    //     "created_at": 1708930855,
    //     "status": "processed",
    //     "status_details": null
    //   },
    //   {
    //     "object": "file",
    //     "id": "file-XvEf2taaDHQaWBr9YVNfVSlX",
    //     "purpose": "assistants",
    //     "filename": "G38 SP12 Cell displacement HVT Work Instruction_CN_V5_0511.docx",
    //     "bytes": 11077538,
    //     "created_at": 1708929130,
    //     "status": "processed",
    //     "status_details": null
    //   }
    // ]
}
```









### 6-删除文件

```javascript
// 删除文件
async deleteFile () {
    const file = await this.openai.files.del('file-jFes5XzTIXCtliZ2uz9GrKmI')
    console.log(file)
    // 示例
    // {
    //   "object": "file",
    //   "deleted": true,
    //   "id": "file-jFes5XzTIXCtliZ2uz9GrKmI"
    // }
}
```







### 7-创建助手

```javascript
// 创建一个带有模型和说明的助手
async createAssistant () {

  const assistant = await this.openai.beta.assistants.create({
    instructions: '你是客户支持聊天机器人，需要使用中文回答客户的询问。',
    model: 'gpt-4-0125-preview',
    // 开启'code_interpreter'后，助手会解释附加的文件
    tools: [ { type: 'code_interpreter' } ],
    file_ids: []  // 可选的
  })

  console.log(assistant)
  // 示例
  // {
  //   "id": "asst_jdUvFkWIo4iIXKwD8FRe4lWR",
  //   "object": "assistant",
  //   "created_at": 1708932851,
  //   "name": null,
  //   "description": null,
  //   "model": "gpt-4-0125-preview",
  //   "instructions": "你是客户支持聊天机器人，需要使用中文回答客户的询问。",
  //   "tools": [
  //     {
  //       "type": "code_interpreter"
  //     }
  //   ],
  //   "file_ids": [],
  //   "metadata": {}
  // }

}
```







### 8-删除助手

……







### 9-修改助手

……







### 10-查询助手

……







### 11-列出助手列表

```javascript
// 检索助手列表 
async getAssistantList () {
  const myAssistants = await this.openai.beta.assistants.list({
    order: "desc",  // 降序
    limit: "20"  // 只输出20条
  })

  console.log(myAssistants.data)
  // 示例
  // [
  //   {
  //     "id": "asst_jdUvFkWIo4iIXKwD8FRe4lWR",
  //     "object": "assistant",
  //     "created_at": 1708932851,
  //     "name": null,
  //     "description": null,
  //     "model": "gpt-4-0125-preview",
  //     "instructions": "你是客户支持聊天机器人，需要使用中文回答客户的询问。",
  //     "tools": [
  //         {
  //           "type": "code_interpreter"
  //         }
  //     ],
  //     "file_ids": [
  //       "file-XuqA6ts1Y9EN9ZIzBNZ6IpOI"
  //     ],
  //     "metadata": {}
  //   },
  //   {
  //     "id": "asst_78scnFhFxd42rDHfgPXFnj2j",
  //     "object": "assistant",
  //     "created_at": 1708656500,
  //     "name": null,
  //     "description": null,
  //     "model": "gpt-4-0125-preview",
  //     "instructions": "你是客户支持聊天机器人，需要使用中文回答客户的询问。",
  //     "tools": [
  //         {
  //           "type": "code_interpreter"
  //         }
  //     ],
  //     "file_ids": [
  //       "file-jFes5XzTIXCtliZ2uz9GrKmI"
  //     ],
  //     "metadata": {}
  //   }
  // ]
}
```







### 12-附加文件到助手

```javascript
// 通过将File附加到助手来创建助手文件
// 不支持使用file_ids传递Array，创建助手时支持使用file_ids传递Array
async createAssistantFile () {
  const myAssistantFile = await this.openai.beta.assistants.files.create(
    'asst_jdUvFkWIo4iIXKwD8FRe4lWR',
    {
      file_id: 'file-XuqA6ts1Y9EN9ZIzBNZ6IpOI'
    }
  )
  console.log(myAssistantFile)
  // 示例
  // {
  //   "id": "file-XuqA6ts1Y9EN9ZIzBNZ6IpOI",
  //   "object": "assistant.file",
  //   "created_at": 1708933195,
  //   "assistant_id": "asst_jdUvFkWIo4iIXKwD8FRe4lWR"
  // }
}
```







### 13-删除助手文件

……







### 14-查询助手文件

……







### 15-列出助手文件的列表

```javascript
async getAssistantFilesList () {
  const assistantFiles = await this.openai.beta.assistants.files.list(
    'asst_jdUvFkWIo4iIXKwD8FRe4lWR'
  )
  console.log(assistantFiles.data)
  // 示例
  // [
  //   {
  //       "id": "file-XvEf2taaDHQaWBr9YVNfVSlX",
  //       "object": "assistant.file",
  //       "created_at": 1708936284,
  //       "assistant_id": "asst_jdUvFkWIo4iIXKwD8FRe4lWR"
  //   },
  //   {
  //       "id": "file-XuqA6ts1Y9EN9ZIzBNZ6IpOI",
  //       "object": "assistant.file",
  //       "created_at": 1708933195,
  //       "assistant_id": "asst_jdUvFkWIo4iIXKwD8FRe4lWR"
  //   }
  // ]
}
```







### 16-核心对象

![](https://github.com/helloXuYifan/test/blob/main/assets/images/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87-2.png?raw=true)

| 对象      | 作用                                                         |
| :-------- | ------------------------------------------------------------ |
| Assistant | 助手，可以使用指定模型根据的一个实体，如果把助手比作某个人的话，这里就是指具备某些能力的一个具体的人 |
| Thread    | 线程，可以认为这个是和助手沟通的上下文对话信息， 就好比你和某宝客服沟通，整个对话就可以认为是一个线程 |
| Run       | 运行， 可以认为是你向助手发起一次对话，整个对话响应的过程及工程中的状态变化，就可以当成一个Run，一个Run里不仅仅可以有模型的回复，还可以有函数调用、代码解释器调用…… |
| Run Step  | Run各个步骤的详情，可以看到整个助手的运行过程，主要是方便问题排查和助手优化 |







### 17-创建线程

```javascript
async createThread () {
  const emptyThread = await this.openai.beta.threads.create({
    messages: [
      {
        // 仅user支持
        role: 'user',
        // 消息的内容
        content: '帮我整理下附加文件的内容，有条理一些。'
      }
    ]
  })
  console.log(emptyThread)
  // 示例
  // {
  //   "id": "thread_tMpHMjMfDBds2nBiZKdqTVfT",
  //   "object": "thread",
  //   "created_at": 1708936835,
  //   "metadata": {}
  // }
}
```







### 18-删除线程

……







### 19-修改线程

……







### 20-查询线程

……







### 21-创建运行

```javascript
async createRun () {
  const run = await this.openai.beta.threads.runs.create(
    // 线程id: 必需
    thread_Id,
    // 助手id: 必需
    { assistant_id: '...' }
  )
  console.log(run)
  // 示例
  // {
  //   "id": "run_nxjSIbcIgkhuMqGzDUPjzVA6",
  //   "object": "thread.run",
  //   "created_at": 1708937946,
  //   "assistant_id": "asst_2g2E8WwrZ35IndwNHU0fRoTM",
  //   "thread_id": "thread_5fDgBDbSCNsUsfZoLfK6hJUa",
  //   "status": "queued",
  //   "started_at": null,
  //   "expires_at": 1708938546,
  //   "cancelled_at": null,
  //   "failed_at": null,
  //   "completed_at": null,
  //   "required_action": null,
  //   "last_error": null,
  //   "model": "gpt-4-0125-preview",
  //   "instructions": "你是客户支持聊天机器人，需要友好地回答客户的询问。",
  //   "tools": [
  //     {
  //       "type": "code_interpreter"
  //     }
  //   ],
  //   "file_ids": [
  //     "file-jFes5XzTIXCtliZ2uz9GrKmI"
  //   ],
  //   "metadata": {},
  //   "usage": null
  // }
}
```







### 22-创建线程并运行-推荐使用

```javascript
async createThreadAndRun () {
  const run = await this.openai.beta.threads.createAndRun({
    assistant_id: '...',
    thread: {
      messages: [
        {
          // 仅user支持
          role: 'user', 
          // 消息的内容
          content: '帮我整理下附加文件的内容，有条理一些。' 
        },
      ]
    },
    // 覆盖助手的默认系统消息
    instructions: '你是客户支持聊天机器人，需要使用中文回答客户的询问。'
  })

  console.log(run)
  // 示例
  // {
  //   "id": "run_hufgEjL7sAdnDhg66xiW7ijK",
  //   "object": "thread.run",
  //   "created_at": 1708941363,
  //   "assistant_id": "asst_2g2E8WwrZ35IndwNHU0fRoTM",
  //   "thread_id": "thread_JGzVIzv9e2TCkm52IG0J7lBZ",
  //   "status": "queued",
  //   "started_at": null,
  //   "expires_at": 1708941963,
  //   "cancelled_at": null,
  //   "failed_at": null,
  //   "completed_at": null,
  //   "required_action": null,
  //   "last_error": null,
  //   "model": "gpt-4-0125-preview",
  //   "instructions": "你是客户支持聊天机器人，需要使用中文回答客户的询问。",
  //   "tools": [
  //       {
  //           "type": "code_interpreter"
  //       }
  //   ],
  //   "file_ids": [
  //       "file-jFes5XzTIXCtliZ2uz9GrKmI"
  //   ],
  //   "metadata": {},
  //   "usage": null
  // }
}
```







### 23-Run的一些状态

![](https://github.com/helloXuYifan/test/blob/main/assets/images/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87-1.png?raw=true)

| 状态            | 定义                                                         |
| --------------- | ------------------------------------------------------------ |
| queued          | 当run首次创建或者调用了retrive获取状态后，就会变成queued等待运行。正常情况下，很快就会变成in_progress状态 |
| in_progress     | 说明run正在执行中，这时候可以调用run step来查看具体的执行过程 |
| completed       | 执行完成，可以获取Assistant返回的消息了，也可以继续向Assistant提问了 |
| requires_action | 如果Assistant需要执行函数调用，就会转到这个状态，然后你必须按给定的参数调用指定的方法，之后run才可以继续运行 |
| expired         | 当没有在expires_at之前提交函数调用输出，run将会过期。另外，如果在expires_at之前没获取输出，run也会变成expired状态 |
| cancelling      | 当你调用取消运行的API后，run就会变成cancelling，取消成功后就会变成callcelled状态 |
| cancelled       | run已成功取消                                                |
| failed          | 运行失败，你可以通过查看run中的last_error对象来查看失败的原因 |







### 24-检索运行

```javascript
async retrieveRun () {
  
  const run = await this.openai.beta.threads.runs.retrieve(
    this.threadId,
    this.runId
  )

  console.log(run)
  // 示例
  // {
  //   "id": "run_NMrjjyLDhKqzNGhVauNk9PxA",
  //   "object": "thread.run",
  //   "created_at": 1709001354,
  //   "assistant_id": "asst_2g2E8WwrZ35IndwNHU0fRoTM",
  //   "thread_id": "thread_vUq3ygNgpMo3FMQVxJG6Fwk5",
  //   "status": "completed",
  //   "started_at": 1709001354,
  //   "expires_at": null,
  //   "cancelled_at": null,
  //   "failed_at": null,
  //   "completed_at": 1709001383,
  //   "required_action": null,
  //   "last_error": null,
  //   "model": "gpt-4-0125-preview",
  //   "instructions": "你是客户支持聊天机器人，需要使用中文回答客户的询问。",
  //   "tools": [
  //     {
  //       "type": "code_interpreter"
  //     }
  //   ],
  //   "file_ids": [
  //     "file-jFes5XzTIXCtliZ2uz9GrKmI"
  //   ],
  //   "metadata": {},
  //   "usage": {
  //     "prompt_tokens": 1985,
  //     "completion_tokens": 540,
  //     "total_tokens": 2525
  //   }
  // }
}
```







### 25-列出消息

```javascript
// 运行状态completed时: 调用本API
// 返回给定线程的消息列表
reqMessagesList () {
  const messages = this.openai.beta.threads.messages.list(thread_id)
  console.log(messages)
  // 示例
  // {
  //   "options": {
  //     "method": "get",
  //     "path": "/threads/thread_PSopshRJ5NZXuGZBKpmx9NAH/messages",
  //     "query": {},
  //     "headers": {
  //       "OpenAI-Beta": "assistants=v1"
  //     }
  //   },
  //   "response": {},
  //   "body": {},
  //   "data": [
  //     {
  //       "id": "msg_86kWhnsp0uSBFzmqEgeAXKPO",
  //       "object": "thread.message",
  //       "created_at": 1709017498,
  //       "assistant_id": "asst_2g2E8WwrZ35IndwNHU0fRoTM",
  //       "thread_id": "thread_PSopshRJ5NZXuGZBKpmx9NAH",
  //       "run_id": "run_7LbxvnN5XTqzhMd8MG3rqeee",
  //       "role": "assistant",
  //       "content": [
  //         {
  //           "type": "text",
  //           "text": {
  //             "value": "我遇到了一些问题，无法直接读取或推断文件的类型。这可能是因为我对文件有一些操作限制。不过，您能否提供这个文件的格式或者内容类型的一些提示吗？比如它是一个文本文件、PDF文件、图片或者其他类型的文件？这样我或许能提供更具体的帮助。",
  //             "annotations": []
  //           }
  //         }
  //       ],
  //       "file_ids": [],
  //       "metadata": {}
  //     },
  //     {
  //       "id": "msg_k4tLifsC37EZOjYHPs0qZl2B",
  //       "object": "thread.message",
  //       "created_at": 1709017482,
  //       "assistant_id": "asst_2g2E8WwrZ35IndwNHU0fRoTM",
  //       "thread_id": "thread_PSopshRJ5NZXuGZBKpmx9NAH",
  //       "run_id": "run_7LbxvnN5XTqzhMd8MG3rqeee",
  //       "role": "assistant",
  //       "content": [
  //         {
  //           "type": "text",
  //           "text": {
  //             "value": "很抱歉，由于我当前的环境限制，无法直接使用某些方法来确定文件的类型。不过，我可以尝试以二进制形式读取文件，然后根据文件头或其他特征进行一些基本推测。让我先尝试这样做，看看能否提供一些有关文件类型的提示。",
  //             "annotations": []
  //           }
  //         }
  //       ],
  //       "file_ids": [],
  //       "metadata": {}
  //     }
  //   ]
  // }	
}
```







### 26-创建消息

```javascript
// 基于给定线程继续对话时使用
// 提交后: 需要创建运行来获取响应
const threadMessages = await openai.beta.threads.messages.create(
  thread_id,
  { role: 'user', content: '消息内容' }
)
console.log(threadMessages)
```







### 27-流程图

![](https://github.com/helloXuYifan/test/blob/main/assets/images/flow.png?raw=true)













### 28-OpenAI-文件大小&支持种类

```bash
 I. 单个文件的大小最大为 512MB
 II. 一个组织上传的所有文件大小最大为 100 GB
 III. 支持的文件种类:  https://platform.openai.com/docs/assistants/tools/supported-files
```

![](https://github.com/helloXuYifan/test/blob/main/assets/images/Supported%20files.png?raw=true)

![](https://github.com/helloXuYifan/test/blob/main/assets/images/Upload%20file.png?raw=true)





### 29-OpenAI-支持的最大Token

https://platform.openai.com/docs/models/overview
![](https://github.com/helloXuYifan/test/blob/main/assets/images/GPT-4-Token.png?raw=true)
![](https://github.com/helloXuYifan/test/blob/main/assets/images/GPT-3-5-Token.png?raw=true)



### 30-OpenAI-Token计算器

https://platform.openai.com/tokenizer

![](https://github.com/helloXuYifan/test/blob/main/assets/images/Token-calc.png?raw=true)









### 31-OpenAI错误码

https://platform.openai.com/docs/guides/error-codes/api-errors

![](https://github.com/helloXuYifan/test/blob/main/assets/images/GPT-Error.png?raw=true)









### 32-附录

1. 快速开始 <https://platform.openai.com/docs/quickstart?context=node> 
2. API文档官方 https://platform.openai.com/docs/api-reference
3. API文档汉化1 https://openai.xiniushu.com/
4. API文档汉化2 https://openai.apifox.cn/



























