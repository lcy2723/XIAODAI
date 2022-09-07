# XDAI

### 配置

**相关配置(端口等)在config/conf.ini中写出**

激活虚拟环境

```bash
conda activate xdai
```

开启GLM的服务，后台记录(XDAI目录下)：

```bash
CUDA_VISIBLE_DEVICES=4 nohup bash tools/deploy_plm.sh > glm_server_68.log &
```

配置mongoDB

配置Knowledge Exploration，运行

```bash
python tools/knowledge/explore.py -t init -fdata/seed_concept.json
```

在XDAI/data/创建zh_list，运行

```bash
nohup python tools/knowledge/explore.py -t update -fdata/seed_concept.json -i 1 > ke_server_69.log 
```

部署similarity

```bash
nohup bash tools/deploy_sentsim.sh > similarity_server.log &
```

### 部署小木接口

在XDAI/api目录下，运行

```bash
nohup uvicorn main:app --host '0.0.0.0' --port 9621 --reload > xiaomu.log &
```

#### 接口格式

- question是当前用户输入的对话语句

- chat_id是当前会话的id,用户新打开一次对话框会更新一个chat_id

- source是小木自带的 分类器 给用户输入语句分的类，我们的小呆只负责处理“来自图灵机器人"这一类，即闲聊

- dialog_cache是此用户的对话历史
  - dialog_cache的列表里的每个字典是一次对话记录
    - final_question是用户的问题/说的话
    - answers是小木的回复，大部分情况下answers里面只有一个字典，我还没见到多个answer的情况

- user_video_cache是此用户的看视频行为历史

- dialog_cache与user_video_cache的最大长度暂时定为16

* **course_id**，是当前用户所处课程的id, 32服务器收到后可以直接拿它从/data/tsq/MOOCCube2/entities/course.json 里面查course实体的信息

* faq_qa_pairs

  - 里面是一个列表，每个元素是一个字典，形如：{'question': '如何获得认证证书', 'answer': '成绩合格就可‘}

  - 这个列表的长度不会超过6

  - 我们是在小木的服务器上，用学堂的FAQ库和Mooccube的概念库去搜索qa_pair的，很多情况搜不到结果，故列表可能为空

请求体示例:

```json
{
    "user_id":"112",
    "question":"比如？",
    "chat_id":"d1dd4ade28ba96431896ad56bb6927bd4dd0",
    "course_id": "C_697797",
    "source":"来自图灵机器人",
    "dialog_cache":[
        {
            "final_question":"你能帮助我什么",
            "answers":[{"uid": 0, "tag": -1, "flag": "", "message": "我可以帮助你好多好多", "source": "来自图灵机器人", "score": 0.8575681447982788, "course_id": "883345", "chat_id": "d1dd4ade28ba96431896ad56bb6927bd4dd0", "qid": "62d8f3f75f0fe57d537a4a8c", "rid": "62d8f3f75f0fe57d537a4a8d"}],
        },
    ],
    "user_video_cache":[
        {
            "video_id": "17397687",
            "event_type":"play",
            "course_id": "697797",
            "current_point":"10",
            "from_position":"15",
            "to_position":"10",
            "duration":"577.1",
            "speed":"1",
            "interval":"5",
            "user_time": "2022-07-18 16:00:31",
            
        }
    ],
    "faq_qa_pairs":[
    {"question": "如何获得认证证书", "answer": "课程结课后，成绩达到合格线，可以获得认证证书。"}, 
    {"question": "如何可获得认证证书？", "answer": "请在学堂在线官网或微信小程序中，点击个人中心-我的证书，按提示进行证书申请。"}
    ]
}
```

返回数据

```json
{
    "res": "比如学习",
    "user_id": "112" 
}
```


