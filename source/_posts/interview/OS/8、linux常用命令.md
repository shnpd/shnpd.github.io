---
title: 8、linux常用命令
toc: true
date: 2025-03-03 00:00:00
tags: 操作系统
categories: 
	- 知识点整理
	- 操作系统


---

点击阅读更多查看文章内容<!--more-->

## **1. 查看日志文件**

### **cat/tac**

- `cat /var/log/syslog` # 查看完整日志
- `tac /var/log/syslog` # 逆序查看日志（最新日志在前）

### **less/more**

- `less /var/log/syslog` # 分页查看日志，支持上下滚动
- `more /var/log/syslog` # 逐页查看（只能向下翻页）

### **head/tail**

- `head -n 50 /var/log/syslog` # 查看日志的前50行
- `tail -n 50 /var/log/syslog` # 查看日志的最后50行
- `tail -f /var/log/syslog` # 实时跟踪日志变化（常用于监控）

------

## **2. 搜索日志**

### **grep**

- `grep "error" /var/log/syslog` # 查找包含"error"的日志
- `grep -i "failed" /var/log/auth.log` # 忽略大小写搜索"failed"
- `grep -E "error|fail|warning" /var/log/syslog` # 搜索多个关键字
- `grep -A 5 "error" /var/log/syslog` # 显示"error"及其后5行日志
- `grep -B 5 "error" /var/log/syslog` # 显示"error"及其前5行日志
- `grep -C 5 "error" /var/log/syslog` # 显示"error"及其前后5行日志

### **awk**

- `awk '{print $1, $2, $5}' /var/log/syslog` # 只显示日志的前两列时间和进程
- `awk '/error/ {print $0}' /var/log/syslog` # 只显示包含"error"的日志

### **sed**

- `sed -n '/error/p' /var/log/syslog` # 只打印包含"error"的行
- `sed -n '50,100p' /var/log/syslog` # 只显示50到100行的日志

------

## **3. 统计分析日志**

### **统计某个关键字出现的次数**

- `grep -c "error" /var/log/syslog` # 统计"error"出现的次数
- `grep "error" /var/log/syslog | wc -l` # 另一种统计方法