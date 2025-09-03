# Mod-SymlinkDir Maker

## 简介

这是一个设计为给游戏打mod的玩家的项目。本项目的基本思路是:NTFS下的符号链接能够将文件保存成多个地址但是只存在一份文件，而相当多的mod都是进行侵入式修改的，也就是需要一定程度的修改游戏本体，为了防止打mod打炸就会进行备份，本项目便为了解决此问题。

### 原理

本项目映射的方式是递归检索每一个原始文件夹的文件，并生成文件表。基于此文件表，在新的文件夹中恢复文件结构，但是使用真实文件夹。

### 配置文件

本项目具有多份配置文件，第一份位于本程序根目录的Config目录中，第二份位于%Appdata%\..\Local\Mod-SymlinkDirConfig\目录中，其他份位于符号链接后的文件夹中。

#### 符号链接记录的文件夹配置

```
.\symlinfo
    \globalconfig\~
    \multisymlink\~
    symlinkindex
    symlinkstats
```

在symlinkindex中，文件将被描述为:

```
//SymLink INDEX UTF-8
LAYER:0 # 此条说明本文件夹在本程序的迭代层，一般情况下为0
ROOTDIR:"$ROOTDIR" #此条描述为原始文件夹的位置，如果LAYER参数不为0则为上层嵌套的记录，将会进入\multisymlink\layer$LAYER-1\中寻找更上层的记录。
//SYMLINKINDEX
.\samplefile1:.\samplefile1 #以冒号为分隔符，前者为本地的符号链接，后者为原始文件位置。
以\samplefile2:2:.\sampleile2 #双冒号直接为层，将会检索到其他层中的记录
```

由于此程序设计为支持多重的源，则第二个源将会在描述文件后面增加一个1，并将第一个源的描述文件后面增加一个0，multisymlink中的文件夹亦是，即：

```
.\symlinfo
    \~
    \multisymlink\layer0\symlinkindex0
    \multisymlink\layer0\symlinkindex1
    \multisymlink\layer0\symlinkindex2
    \multisymlink\layer1\symlinkindex1
    \multisymlink\layer1\symlinkindex2
    \multisymlink\layer2\symlinkindex1
    symlinkindex0
    symlinkstats0
    symlinkindex1
    symlinkstats1
    symlinkindex2
    symlinkstats2
```

由于文件路径可能出现极长的状态，若在较长的路径(>200字符)下，本程序将会进一步抽象为:

```
.\sli # 即为.\symlinfo
    \gc\~ # 即为.\symlinfo\globalconfig\~
    \msl\0\0 # 即为.\symlinfo\multisymlink\layer0\symlinkindex0
    \msl\0\1
    \msl\1\0
    i0 # 即为.\symlinkindex0
    s0 # 即为.\symlinkstats0
    i1
    s1
```

此项允许在全局配置中配置为默认模式。

在symlinkstats中，文件描述为:

```
//SymLink STATS UTF-8
ERROREXIT:0 # 此条默认为1，程序正常退出才会为0。若本程序异常退出，下次启动将会强制重新检查文件的所有状态。且本条仅会出现在s0中。
LINKSTAT:0 # 此条默认为0，程序将会以此判断符号链接的优先程度。0为最优先级，在点击重新同步之后本程序将会以此为判断条件进行覆盖。
```
