### Bank就是UR的分区



### 1、 UR 指User Register用户寄存器

就是内存，AIACC内部存储数据的一块RAM



### 2、Bank

将UR的内容进行了分块，每一块都是一个Bank

**为了避免数据读写冲突，可以同时从Bank0读数据、向Bank2写数据这样子**



**文档中UR默认分为了8个Bank，（由设计参数 PARAM-UR-BANK-BIT-NUM=3 推导，2³=8）**

**每个Bank多大，256个bit**



