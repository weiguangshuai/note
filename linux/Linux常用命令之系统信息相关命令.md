# Linux常用命令之系统信息查看

## 查看系统信息

### 查看物理cpu数、核心数、逻辑cpu个数

- 总核心数 = 物理cpu数 * 每颗物理cpu的核心数
- 总逻辑cpu数 = 物理cpu数 * 每颗物理cpu的核心数 * 超线程数

#### 查看物理cpu个数

```shell
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l
```

#### 查看每个物理cpu的核心数

```shell
cat /proc/cpuinfo| grep "cpu cores"| uniq
```

#### 查看逻辑cpu的个数

```shell
cat /proc/cpuinfo| grep "processor"| wc -l
```

#### 查看cpu信息（型号）

```shell
cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c
```



### 查看内存和硬盘信息

#### 查看内存信息

```shell
free
#加上-h以G为单位显示
free -h
```

#### 查看硬盘信息

```shell
df
#加上-h以G为单位显示
df -h
```



### 查看Linux系统型号

####查看操作系统型号

```shell
cat /etc/redhat-release
```





