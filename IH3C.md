H3C
===

### 基本命令

## dis

```
dis cu
dis this
```



#### nat

```
sys
int g1/0/4
dis this
nat server protocol tcp global 122.225.120.150 18007 inside 192.168.86.47 8006 rule 内部服务器规则_1712
nat server protocol tcp global 122.225.120.150 18008 inside 192.168.86.44 60202 rule 内部服务器规则_1713
qu
save
```

删除

```
undo nat server protocol tcp global 122.225.120.150 18008 
```

