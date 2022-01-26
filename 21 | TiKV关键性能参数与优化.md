# 21 | TiKV关键性能参数与优化
## TiKV数据写入流程
![](https://s3.bmp.ovh/imgs/2022/01/2f0ce6d414770b3a.jpg)

## 写入瓶颈分析
![](https://s3.bmp.ovh/imgs/2022/01/0cf16e6b106723b5.jpg)

## TiKV写入参数优化
![](https://s3.bmp.ovh/imgs/2022/01/e72470f9df1741b6.png)

## TiKV数据读取流程——点查
![](https://s3.bmp.ovh/imgs/2022/01/cf463830a922042a.png)

## TiKV数据读取流程——非点查
![](https://s3.bmp.ovh/imgs/2022/01/ae4310a5aec6ea32.png)

## 读取瓶颈分析
![](https://s3.bmp.ovh/imgs/2022/01/b3a8311b4f9101db.png)

## 读取参数优化
![](https://s3.bmp.ovh/imgs/2022/01/dc9cf710111eb781.png)

## 常见问题处理——Write Stall
- 由于TiKV中采用2个RocksDB作为raft log的kv数据的存储，所以当memtalbe或者level 0 的数据量过多时，RocksDB就会对写入进行减速，从而达到自我保护的目的，这个动作就叫write stall。
![](https://s3.bmp.ovh/imgs/2022/01/536848a0a5d16841.png)

## 常见问题处理——TiKV slow query
![](https://s3.bmp.ovh/imgs/2022/01/14595e948b79d7ff.png)
