---
title: mac m1下安装kafka for python
tags:
  - kafka
date: 2022-05-05 22:44:25
categories: kafka
keywords: 
description: 在Mac m1电脑上如何正确安装kafka client for python
---



1. 安装`librdkafka`

   安装位置在`/opt/homebrew/Cellar/librdkafka/1.8.2/`，最后的数字因版本而异

   ```bash
   brew install librdkafka
   ```

   

2. 安装`confluent_kafka`

   **1.8.2** 根据实际版本调整位置

   ```bash
   C_INCLUDE_PATH=/opt/homebrew/Cellar/librdkafka/1.8.2/include LIBRARY_PATH=/opt/homebrew/Cellar/librdkafka/1.8.2/lib pip install confluent_kafka
   ```

   



参考：

[1] [mac m1 arm 安装 confluent-kafka 报错解决方案 - SegmentFault 思否](https://segmentfault.com/a/1190000040867082)

[2] [Install failed in Apple Silicon · Issue #1025 · confluentinc/confluent-kafka-python (github.com)](https://github.com/confluentinc/confluent-kafka-python/issues/1025)
