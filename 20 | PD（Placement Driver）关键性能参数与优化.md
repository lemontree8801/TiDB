# 20 | PD（Placement Driver）关键性能参数与优化
## 调度limit参数
### 生产者
|参数|默认值|说明|
|:-:|:-:|:-:|
|region-schedule-limit|2048|同时进行Region调度的任务个数|
|leader-schedule-limit|4|同时进行leader调度的任务个数|
|replica-schedule-limit|64|同时进行replica调度的任务个数|
|merge-schedule-limit|8|同时进行的Region Merge调度任务，设置为0则关闭Region Merge|
|hot-region-schedule-limit|4|控制同时进行的hot Region任务。该配置项独立于Region调度。

### 消费者
- 消费限速- store limit
	- 定义：限制单个store的消费速度
	- 方式：pd-ctl -u ip:port store limit  id  value
	- 区别：Store Limit限制的主要是operator的消费速度，而其他的limit主要限制operator的产生速度。

## 存储空间阈值参数
- high-space-ratio和low-space-ratio

## 其他参数
|参数|默认值|说明|
|:-:|:-:|
|patrol-region-interval|100ms|控制扫region的间隔，默认100ms，通常不需要调整|
|tolerant-size-ratio|0|控制balance region缓冲区大小，4.0以后版本默认0，表示自动调整，通常不需要修改。设置过大可能会造成集群数据分布不太均衡。|
|region_weight和leader_weight|1|PD计算region和leader分数之后，会除以weight得到最终的region和leader分值，weight参数默认为1，通常不需要修改。|

## pd-ctl基本操作
- 查看并修改调度参数
```
config show --显示当前调度相关参数
config set <key> <vlaue> --修改相关参数
store limit <store_id> <value> --限制单个store的调度速度
```
- 手动添加Operator
```
operator show [admin|leader|region] --展示当前全局或者是某类的调度任务
operator add --人工添加一些调度任务实现期望目标，例如：
	- operator add add-peer <region_id> <store_id>
	- operator add remove-peer <region_id> <store_id>
	- operator add transfer-leader <region_id> <store_id>
```

## 常见问题的处理-扩容后balance region 调度速度慢
![](https://s3.bmp.ovh/imgs/2022/01/e6d43732af5ce863.png)

## 常见问题的处理—Store节点故障后补副本的速度慢
![](https://s3.bmp.ovh/imgs/2022/01/a7d0848725dd9fb6.png)

## 常见问题的处理-Region merge速度慢
![](https://s3.bmp.ovh/imgs/2022/01/bd8d9ff52b7d34d2.png)

