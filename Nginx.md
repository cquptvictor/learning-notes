# Nginx

### location匹配规则

1. =精准匹配优先级最高
2. 普通字符串匹配优先级第二，按照最长前缀匹配
3. 普通字符串匹配到后，继续查找正则匹配，若找到，使用正则匹配的规则；若没找到，使用普通字符串匹配规则
4. 前缀匹配
5. 正则匹配

## alias和root

- alias是alias路径替换location中的路径
- root是root路径+location中的路径
- alias路径末尾一定要加上/

## 反向代理

## 负载均衡

## 动静分离

## 高可用