---
title: 常用Linux命令
---

# 常用Linux命令

---

## 进入到锦鲤部署主目录

```bash
cd /var/jinli
```

## 查看锦鲤目录下的文件

```bash
ls /var/jinli
```

## 统计锦鲤部署的空间占用

```bash
du -sh /var/jinli/
```

## 查看Linux进程

```bash
ps aux
```

如果需要查看某些进程，比如MySQL可以使用grep做过滤

```bash
ps aux | grep mysql
```

