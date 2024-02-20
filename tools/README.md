# 数据获取

⚙️基于API的数据获取与处理

## 需要准备的

1. OpenAI格式的api
2. python环境（参考快速开始中的环境配置环节）

## 数据的组成

项目数据组成分为以下三部分，三个部分都需要 api ，任意选择其中两个即可做出不错的效果

- 基础问题重复询问：使用API，让Chat-GPT扮演角色，提供一定的prompt让其模仿语气问答
- 原文短对话提取（参照[葱老师](https://github.com/KMnO4-zx)的[extract-dialogue](https://github.com/KMnO4-zx/extract-dialogue)）但作者进行了一定的修改
- 原文长对话提取

## 数据的获取

### 1.基础问题重复询问

提供脚本 `q2a_api.py` 但需要自行填入 `api_key` 和 `api_base_url` 以及 `base_prompt` 

注意：base_prompt 会影响回复的质量

💬以下是沙悟净的 prompt

```shell
base_prompt='沙悟净，原名沙和尚，是中国古典名著《西游记》中的重要角色之一，曾是天宫的卷帘大将，因犯下天条被贬至凡间，化为河边的一条怪鱼，直到遇见唐僧并成为其第三个徒弟。沙和尚在唐僧西行取经的过程中，扮演了重要的角色。他性格沉稳、忠诚，不善言辞，但行动力强，是队伍中的主要劳动力。沙悟净擅长使用武器“月牙铲”，在与妖魔鬼怪的战斗中，他总能稳重地给予支持，保护师傅和师兄弟们的安全。沙悟净的性格与他的过去有着密切的关系。他的经历让他深知忠诚与责任的重要性，因此在很多困难面前，他总是表现出坚定不移的勇气和毅力。尽管沙悟净的话语不多，但他的行动充分展现了他的勇敢和忠诚。他对佛法有着虔诚的信仰，经常以实际行动来体现佛教的教义，如助人为乐、勤劳不辍。在与唐僧和其他徒弟的互动中，沙悟净常常是稳重的一员，他的冷静和理性为团队解决了不少困难。他虽然不像孙悟空那样具有超凡的武艺，也不像猪八戒那样幽默风趣，但他的坚韧不拔和默默付出使他成为队伍中不可或缺的一员。沙悟净的言行举止虽然简单朴实，但正是这种朴实无华的品质，体现了他作为一名僧侣的真实修为和深厚的人生智慧。请你扮演沙悟净回答我的问题，尽量保持回答的自然回答，当然你也可以适当穿插一些文言文，尽可能贴合原著，我的问题是：'

```


本质是借助已经训练好的 LLM 进行角色扮演。

运行脚本 `q2a_api.py` 

```shell
python tools/get_data/Q2A/q2a_api.py --questions_path {your_question} --save_path {save_path} --repeat 5
```

参数说明：

`--questions_path` : 基础问题，可以从 Chat-GPT 等模型中获取，项目提供了955个基础问题用于提问。

`--save_path` :保存路径，一般是 output/xxx.jsonl，脚本会整理好 xtuner 可训练的格式。

`--repeat` :重复次数，西游系列的四个模型重复询问了5次。

### 2.原文短对话提取

原 repo 链接：**[extract-dialogue](https://github.com/KMnO4-zx/extract-dialogue)**

1.从原文中获取对话（以沙悟净为例）
    
    首先需要在 `tools/get_data/extract-dialogue/OpenAI_LLM.py` 中配置 api
    
    然后运行脚本
    

```shell
python tools/get_data/extract-dialogue/main.py --path {novel_path} --roles 沙僧,沙和尚,卷帘大将,金身罗汉,沙师弟,老沙,沙悟净,悟净
```

参数说明：

`--path` :小说路径，一般是 *.txt

`--roles` :角色可能的称呼，注意用英文逗号隔开

完成后会在 `tools/get_data/extract-dialogue/output` 下生成两个文件 *.json 就是对话内容

2.将对话内容转换为 xtuner 可用格式

```shell
python tools/get_data/extract-dialogue/process_data.py --raw_data {output.json} --save_path {swk.jsonl} --role 沙悟净
```

参数说明：

`--raw_data` :提取的对话

`--save_path` :保存的路径

`--role` :角色名称

### 3.长对话提取（此模块脚本可能需要优化）
    
  此脚本与方法1中脚本类似 同样需要配置 api ，具体prompt修改如下
    
  ```shell
  base_prompt='你是一个对话整理大师，以下内容为《西游记》节选，请你整理出角色“唐三藏”，“孙悟空”，“猪八戒”，“沙悟净”四人的对话内容，当然，这四人在小说中可能以别的名字出现，如：唐三藏->金蝉子，孙悟空->猴王->行者等人物需要你根据理解自行判别，直接返回对话内容，返回格式为：唐三藏：{对话内容}，孙悟空：{对话内容}，猪八戒：{对话内容}，沙悟净：{对话内容}，某人说：{对话内容}；若内容中无对话，则直接回答“无对话内容”无需提及人物，若对话不完整或者你没法确定对话的人物关系，你可以放弃整理，直接回复“无对话内容”无需提及人物，若出现非四人内任务与四人对话，非四人内的以“某人说”记录，请保持对话的准确性，不要修改和翻译，请不要解释。以下为节选片段：'
  ```
    
  运行脚本
    
  ```shell
  python tools/get_data/long-dialogue/q2a_api.py --file_path {novel_path} --save_path {save_path}
  ```
  
  完成后会生成由 GPT 生成的对话整理
  
  接下来运行脚本提取长对话
  
  ```shell
  python tools/get_data/long-dialogue/get_data.py --data_path {conversation.txt} --save_path {output path} 
  ```
    
  该脚本一次可以生成多个角色的符合 xtuner 的训练数据
    

三个方法完成后需要整理到同一个 .jsonl 文件下，即可进行下一步使用 XTuner 微调
