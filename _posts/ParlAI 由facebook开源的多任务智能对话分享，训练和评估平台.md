---
layout:     post
title:      ParlAI 的使用测试
subtitle:   测试
date:       2019-12-04
author:     IceGomic
header-img: img/post-bg-swift2.jpg
catalog: true
tags:
    - ParlAI
    - 人工智能
    - 对话系统
---


> 本文首次发布于 [IceGomic Blog](http://icegomic.github.io), 作者 [@IceGomic](http://github.com/icegomic) ,转载请保留原文链接.


[![ParlAI](https://upload-images.jianshu.io/upload_images/100763-7ac7df32dfa23fec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

特色介绍
- 1 集成了大量的公开数据集---从公开领域闲聊到专业的视觉问题问答一应俱全
- 2 海量参考模型
- 3 无缝衔接亚马逊Mechanical Turk系统，完成数据收集，训练和人工评估

新建一个目录
```
 mkdir  parlAI
```
下载和安装
```
git clone https://github.com/facebookresearch/ParlAI.git
cd ParlAI; python setup.py develop
```
可以看到输出
```
Cloning into 'ParlAI'...
remote: Enumerating objects: 47, done.
remote: Counting objects: 100% (47/47), done.
remote: Compressing objects: 100% (44/44), done.
remote: Total 24850 (delta 18), reused 10 (delta 3), pack-reused 24803
Receiving objects: 100% (24850/24850), 28.80 MiB | 1.36 MiB/s, done.
Resolving deltas: 100% (17462/17462), done.
```
安装速度还是很快的
之后进入目录
执行测试代码

这个测试是在1k训练样本的BabI任务上随机输出 10条任务1的样例结果
```
python examples/display_data.py -t babi:task1k:1
```
这个中间可能有很多依赖需要安装，请自行处理吧
正常运行的结果，我们来看一下
```
[ optional arguments: ] 
[  display_ignore_fields: agent_reply ]
[  max_display_len: 1000 ]
[  num_examples: 10 ]
[ Main ParlAI Arguments: ] 
[  batchsize: 1 ]
[  datapath: /home/xxx/parlAI/ParlAI/data ]
[  datatype: train:stream ]
[  download_path: /home/xxx/parlAI/ParlAI/downloads ]
[  hide_labels: False ]
[  image_mode: raw ]
[  init_opt: None ]
[  multitask_weights: [1] ]
[  numthreads: 1 ]
[  show_advanced_args: False ]
[  task: babi:task1k:1 ]
[ ParlAI Model Arguments: ] 
[  dict_class: None ]
[  init_model: None ]
[  model: None ]
[  model_file: None ]
[ PytorchData Arguments: ] 
[  batch_length_range: 5 ]
[  batch_sort_cache_type: pop ]
[  batch_sort_field: text ]
[  numworkers: 4 ]
[  pytorch_context_length: -1 ]
[  pytorch_datapath: None ]
[  pytorch_include_labels: True ]
[  pytorch_preprocess: False ]
[  pytorch_teacher_batch_sort: False ]
[  pytorch_teacher_dataset: None ]
[  pytorch_teacher_task: None ]
[  shuffle: False ]
[ ParlAI Image Preprocessing Arguments: ] 
[  image_cropsize: 224 ]
[  image_size: 256 ]
[ Current ParlAI commit: e8d0a75d291c7bb4b4e5565d60a899f794c10963 ]
[creating task(s): babi:task1k:1]
[building data: /home/xxx/parlAI/ParlAI/data/bAbI]
[ downloading: http://parl.ai/downloads/babi/babi.tar.gz to /home/xxx/parlAI/ParlAI/data/bAbI/babi.tar.gz ]
Downloading babi.tar.gz: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████| 19.2M/19.2M [00:05<00:00, 3.29MB/s]
unpacking babi.tar.gz
[loading fbdialog data:/home/xxx/parlAI/ParlAI/data/bAbI/tasks_1-20_v1-2/en-valid-nosf/qa1_train.txt]
[loading fbdialog data:/home/xxx/parlAI/ParlAI/data/bAbI/tasks_1-20_v1-2/en-valid-nosf/qa1_train.txt]
[babi:task1k:1]: Mary moved to the bathroom.
John went to the hallway.
Where is Mary?
[labels: bathroom]
[label_candidates: office|hallway|kitchen|bathroom|bedroom|...and 1 more]
~~
[babi:task1k:1]: Daniel went back to the hallway.
Sandra moved to the garden.
Where is Daniel?
[labels: hallway]
[label_candidates: office|hallway|kitchen|bathroom|bedroom|...and 1 more]
~~
[babi:task1k:1]: John moved to the office.
Sandra journeyed to the bathroom.
Where is Daniel?
[labels: hallway]
[label_candidates: office|hallway|kitchen|bathroom|bedroom|...and 1 more]
~~
[babi:task1k:1]: Mary moved to the hallway.
Daniel travelled to the office.
Where is Daniel?
[labels: office]
[label_candidates: office|hallway|kitchen|bathroom|bedroom|...and 1 more]
~~
[babi:task1k:1]: John went back to the garden.
John moved to the bedroom.
Where is Sandra?
[labels: bathroom]
[label_candidates: office|hallway|kitchen|bathroom|bedroom|...and 1 more]
- - - - - - - - - - - - - - - - - - - - -
~~
[babi:task1k:1]: Sandra travelled to the office.
Sandra went to the bathroom.
Where is Sandra?
[labels: bathroom]
[label_candidates: office|hallway|kitchen|bathroom|bedroom|...and 1 more]
~~
[babi:task1k:1]: Mary went to the bedroom.
Daniel moved to the hallway.
Where is Sandra?
[labels: bathroom]
[label_candidates: office|hallway|kitchen|bathroom|bedroom|...and 1 more]
~~
[babi:task1k:1]: John went to the garden.
John travelled to the office.
Where is Sandra?
[labels: bathroom]
[label_candidates: office|hallway|kitchen|bathroom|bedroom|...and 1 more]
~~
[babi:task1k:1]: Daniel journeyed to the bedroom.
Daniel travelled to the hallway.
Where is John?
[labels: office]
[label_candidates: office|hallway|kitchen|bathroom|bedroom|...and 1 more]
~~
[babi:task1k:1]: John went to the bedroom.
John travelled to the office.
Where is Daniel?
[labels: hallway]
[label_candidates: office|hallway|kitchen|bathroom|bedroom|...and 1 more]
- - - - - - - - - - - - - - - - - - - - -
~~
[ loaded 180 episodes with a total of 900 examples ]
```