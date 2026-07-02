# Craig记录

> 围绕 AI 编译器、MLIR / TPU-MLIR、后端代码、硬件指令集、模型量化与工程环境配置展开的学习笔记仓库。

## 仓库概览

- AI 编译器、MLIR、TPU-MLIR、后端代码、硬件指令集、模型量化、工程环境

## 分目录索引

<details>
<summary><strong>AI编译器的初始思路/</strong> · 0 个子目录 · 9 个文件</summary>

从传统编译器类比、算子定义、kernel、量化与学习路线切入，建立 AI 编译器的整体直觉。

- [01 kenel.md](<AI%E7%BC%96%E8%AF%91%E5%99%A8%E7%9A%84%E5%88%9D%E5%A7%8B%E6%80%9D%E8%B7%AF/01%20kenel.md>)
- [02 定义算子包括几个方面.md](<AI%E7%BC%96%E8%AF%91%E5%99%A8%E7%9A%84%E5%88%9D%E5%A7%8B%E6%80%9D%E8%B7%AF/02%20%E5%AE%9A%E4%B9%89%E7%AE%97%E5%AD%90%E5%8C%85%E6%8B%AC%E5%87%A0%E4%B8%AA%E6%96%B9%E9%9D%A2.md>)
- [03 pytroch运行.md](<AI%E7%BC%96%E8%AF%91%E5%99%A8%E7%9A%84%E5%88%9D%E5%A7%8B%E6%80%9D%E8%B7%AF/03%20pytroch%E8%BF%90%E8%A1%8C.md>)
- [04 量化区别.md](<AI%E7%BC%96%E8%AF%91%E5%99%A8%E7%9A%84%E5%88%9D%E5%A7%8B%E6%80%9D%E8%B7%AF/04%20%E9%87%8F%E5%8C%96%E5%8C%BA%E5%88%AB.md>)
- [05 kernel重写.md](<AI%E7%BC%96%E8%AF%91%E5%99%A8%E7%9A%84%E5%88%9D%E5%A7%8B%E6%80%9D%E8%B7%AF/05%20kernel%E9%87%8D%E5%86%99.md>)
- [06 fbgemm.md](<AI%E7%BC%96%E8%AF%91%E5%99%A8%E7%9A%84%E5%88%9D%E5%A7%8B%E6%80%9D%E8%B7%AF/06%20fbgemm.md>)
- [07 量化.md](<AI%E7%BC%96%E8%AF%91%E5%99%A8%E7%9A%84%E5%88%9D%E5%A7%8B%E6%80%9D%E8%B7%AF/07%20%E9%87%8F%E5%8C%96.md>)
- [AI编译器类比传统编译器.md](<AI%E7%BC%96%E8%AF%91%E5%99%A8%E7%9A%84%E5%88%9D%E5%A7%8B%E6%80%9D%E8%B7%AF/AI%E7%BC%96%E8%AF%91%E5%99%A8%E7%B1%BB%E6%AF%94%E4%BC%A0%E7%BB%9F%E7%BC%96%E8%AF%91%E5%99%A8.md>)
- [学习路线.md](<AI%E7%BC%96%E8%AF%91%E5%99%A8%E7%9A%84%E5%88%9D%E5%A7%8B%E6%80%9D%E8%B7%AF/%E5%AD%A6%E4%B9%A0%E8%B7%AF%E7%BA%BF.md>)

</details>

<details>
<summary><strong>cmake/</strong> · 0 个子目录 · 13 个文件</summary>

记录 CMake 构建、链接、安装、缓存、find_package / option 等工程构建问题。

- [01 解决编译占用cpu爆掉的问题.md](<cmake/01%20%E8%A7%A3%E5%86%B3%E7%BC%96%E8%AF%91%E5%8D%A0%E7%94%A8cpu%E7%88%86%E6%8E%89%E7%9A%84%E9%97%AE%E9%A2%98.md>)
- [02 链接中函数未定义错误.md](<cmake/02%20%E9%93%BE%E6%8E%A5%E4%B8%AD%E5%87%BD%E6%95%B0%E6%9C%AA%E5%AE%9A%E4%B9%89%E9%94%99%E8%AF%AF.md>)
- [03 构建缓存问题.md](<cmake/03%20%E6%9E%84%E5%BB%BA%E7%BC%93%E5%AD%98%E9%97%AE%E9%A2%98.md>)
- [04 立即编译的工具.md](<cmake/04%20%E7%AB%8B%E5%8D%B3%E7%BC%96%E8%AF%91%E7%9A%84%E5%B7%A5%E5%85%B7.md>)
- [05 install.md](<cmake/05%20install.md>)
- [07 find_program.md](<cmake/07%20find_program.md>)
- [08 option.md](<cmake/08%20option.md>)
- [09 find_package.md](<cmake/09%20find_package.md>)
- [10 覆盖选项.md](<cmake/10%20%E8%A6%86%E7%9B%96%E9%80%89%E9%A1%B9.md>)
- [11 install动态.md](<cmake/11%20install%E5%8A%A8%E6%80%81.md>)
- [12 test.md](<cmake/12%20test.md>)
- [13 strequal.md](<cmake/13%20strequal.md>)
- [Cmake相关命令.md](<cmake/Cmake%E7%9B%B8%E5%85%B3%E5%91%BD%E4%BB%A4.md>)

</details>

<details>
<summary><strong>git/</strong> · 0 个子目录 · 6 个文件</summary>

记录 Git 基础、分支、push、merge、fork 协作与冲突处理。

- [01 git 基本使用.md](<git/01%20git%20%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8.md>)
- [02 git 分支操作.md](<git/02%20git%20%E5%88%86%E6%94%AF%E6%93%8D%E4%BD%9C.md>)
- [03 push命令解析.md](<git/03%20push%E5%91%BD%E4%BB%A4%E8%A7%A3%E6%9E%90.md>)
- [04 git的merge.md](<git/04%20git%E7%9A%84merge.md>)
- [05 团队协作使用fork的内容.md](<git/05%20%E5%9B%A2%E9%98%9F%E5%8D%8F%E4%BD%9C%E4%BD%BF%E7%94%A8fork%E7%9A%84%E5%86%85%E5%AE%B9.md>)
- [06 解决冲突问题.md](<git/06%20%E8%A7%A3%E5%86%B3%E5%86%B2%E7%AA%81%E9%97%AE%E9%A2%98.md>)

</details>

<details>
<summary><strong>mlir/</strong> · 5 个子目录 · 78 个文件</summary>

系统整理 MLIR / TPU-MLIR 相关概念、TableGen、Pass、Pattern、Attr、Type、Shape Infer 与源码机制。

- **llvm提供的一些内容/** — 2 个文件
  - [01 llvm StringRef 的 lower方法.md](<mlir/llvm%E6%8F%90%E4%BE%9B%E7%9A%84%E4%B8%80%E4%BA%9B%E5%86%85%E5%AE%B9/01%20llvm%20StringRef%20%E7%9A%84%20lower%E6%96%B9%E6%B3%95.md>)
  - [02 llvm StringSwitch方法.md](<mlir/llvm%E6%8F%90%E4%BE%9B%E7%9A%84%E4%B8%80%E4%BA%9B%E5%86%85%E5%AE%B9/02%20llvm%20StringSwitch%E6%96%B9%E6%B3%95.md>)
- **mlir文件的相关语法/** — 2 个文件
  - [01 MLIR的核心：Operation.md](<mlir/mlir%E6%96%87%E4%BB%B6%E7%9A%84%E7%9B%B8%E5%85%B3%E8%AF%AD%E6%B3%95/01%20MLIR%E7%9A%84%E6%A0%B8%E5%BF%83%EF%BC%9AOperation.md>)
  - [02 module.md](<mlir/mlir%E6%96%87%E4%BB%B6%E7%9A%84%E7%9B%B8%E5%85%B3%E8%AF%AD%E6%B3%95/02%20module.md>)
- **mlir相关cmake命令/** — 2 个文件
  - [mlir_tablegen.md](<mlir/mlir%E7%9B%B8%E5%85%B3cmake%E5%91%BD%E4%BB%A4/mlir_tablegen.md>)
  - [包装命令.md](<mlir/mlir%E7%9B%B8%E5%85%B3cmake%E5%91%BD%E4%BB%A4/%E5%8C%85%E8%A3%85%E5%91%BD%E4%BB%A4.md>)
- **shape-inferpass代码/** — 3 个文件
  - [30 TupleFusePattern代码内容.md](<mlir/shape-inferpass%E4%BB%A3%E7%A0%81/30%20TupleFusePattern%E4%BB%A3%E7%A0%81%E5%86%85%E5%AE%B9.md>)
  - [32 UnTupleFusePattern代码内容.md](<mlir/shape-inferpass%E4%BB%A3%E7%A0%81/32%20UnTupleFusePattern%E4%BB%A3%E7%A0%81%E5%86%85%E5%AE%B9.md>)
  - [33 CopyMultiUseWeight代码内容.md](<mlir/shape-inferpass%E4%BB%A3%E7%A0%81/33%20CopyMultiUseWeight%E4%BB%A3%E7%A0%81%E5%86%85%E5%AE%B9.md>)
- **中期写的部分，可参考/** — 9 个文件
  - [addOperation.md](<mlir/%E4%B8%AD%E6%9C%9F%E5%86%99%E7%9A%84%E9%83%A8%E5%88%86%EF%BC%8C%E5%8F%AF%E5%8F%82%E8%80%83/addOperation.md>)
  - [attribute.md](<mlir/%E4%B8%AD%E6%9C%9F%E5%86%99%E7%9A%84%E9%83%A8%E5%88%86%EF%BC%8C%E5%8F%AF%E5%8F%82%E8%80%83/attribute.md>)
  - [InputOp.md](<mlir/%E4%B8%AD%E6%9C%9F%E5%86%99%E7%9A%84%E9%83%A8%E5%88%86%EF%BC%8C%E5%8F%AF%E5%8F%82%E8%80%83/InputOp.md>)
  - [mlir自带的pattern注册使用.md](<mlir/%E4%B8%AD%E6%9C%9F%E5%86%99%E7%9A%84%E9%83%A8%E5%88%86%EF%BC%8C%E5%8F%AF%E5%8F%82%E8%80%83/mlir%E8%87%AA%E5%B8%A6%E7%9A%84pattern%E6%B3%A8%E5%86%8C%E4%BD%BF%E7%94%A8.md>)
  - [op的定义相关，生成的属性存在.md](<mlir/%E4%B8%AD%E6%9C%9F%E5%86%99%E7%9A%84%E9%83%A8%E5%88%86%EF%BC%8C%E5%8F%AF%E5%8F%82%E8%80%83/op%E7%9A%84%E5%AE%9A%E4%B9%89%E7%9B%B8%E5%85%B3%EF%BC%8C%E7%94%9F%E6%88%90%E7%9A%84%E5%B1%9E%E6%80%A7%E5%AD%98%E5%9C%A8.md>)
  - [Pass.md](<mlir/%E4%B8%AD%E6%9C%9F%E5%86%99%E7%9A%84%E9%83%A8%E5%88%86%EF%BC%8C%E5%8F%AF%E5%8F%82%E8%80%83/Pass.md>)
  - [PassManager和Pipline.md](<mlir/%E4%B8%AD%E6%9C%9F%E5%86%99%E7%9A%84%E9%83%A8%E5%88%86%EF%BC%8C%E5%8F%AF%E5%8F%82%E8%80%83/PassManager%E5%92%8CPipline.md>)
  - [tablegen语法.md](<mlir/%E4%B8%AD%E6%9C%9F%E5%86%99%E7%9A%84%E9%83%A8%E5%88%86%EF%BC%8C%E5%8F%AF%E5%8F%82%E8%80%83/tablegen%E8%AF%AD%E6%B3%95.md>)
  - [Type.md](<mlir/%E4%B8%AD%E6%9C%9F%E5%86%99%E7%9A%84%E9%83%A8%E5%88%86%EF%BC%8C%E5%8F%AF%E5%8F%82%E8%80%83/Type.md>)
- [01 生成属性的名称.md](<mlir/01%20%E7%94%9F%E6%88%90%E5%B1%9E%E6%80%A7%E7%9A%84%E5%90%8D%E7%A7%B0.md>)
- [02 arguments和results.md](<mlir/02%20arguments%E5%92%8Cresults.md>)
- [03 arguments参数联想问题.md](<mlir/03%20arguments%E5%8F%82%E6%95%B0%E8%81%94%E6%83%B3%E9%97%AE%E9%A2%98.md>)
- [04 trait补充.md](<mlir/04%20trait%E8%A1%A5%E5%85%85.md>)
- [05 trait写.md](<mlir/05%20trait%E5%86%99.md>)
- [06 定义AddConst.md](<mlir/06%20%E5%AE%9A%E4%B9%89AddConst.md>)
- [07 关于TPU-MLIR的attr.md](<mlir/07%20%E5%85%B3%E4%BA%8ETPU-MLIR%E7%9A%84attr.md>)
- [08 Pass中的option.md](<mlir/08%20Pass%E4%B8%AD%E7%9A%84option.md>)
- [09 tpu-mlir中的Pass设计.md](<mlir/09%20tpu-mlir%E4%B8%AD%E7%9A%84Pass%E8%AE%BE%E8%AE%A1.md>)
- [10 tpu-mlir里module设计.md](<mlir/10%20tpu-mlir%E9%87%8Cmodule%E8%AE%BE%E8%AE%A1.md>)
- [11 Attr.md](<mlir/11%20Attr.md>)
- [12 td声明的接口.md](<mlir/12%20td%E5%A3%B0%E6%98%8E%E7%9A%84%E6%8E%A5%E5%8F%A3.md>)
- [13 Operation里的getAttrOfType.md](<mlir/13%20Operation%E9%87%8C%E7%9A%84getAttrOfType.md>)
- [14 枚举属性类的定义补充.md](<mlir/14%20%E6%9E%9A%E4%B8%BE%E5%B1%9E%E6%80%A7%E7%B1%BB%E7%9A%84%E5%AE%9A%E4%B9%89%E8%A1%A5%E5%85%85.md>)
- [15 定义枚举附带方法.md](<mlir/15%20%E5%AE%9A%E4%B9%89%E6%9E%9A%E4%B8%BE%E9%99%84%E5%B8%A6%E6%96%B9%E6%B3%95.md>)
- [16 Arch.h.md](<mlir/16%20Arch.h.md>)
- [17 Arch.h.md](<mlir/17%20Arch.h.md>)
- [18 Arch.md](<mlir/18%20Arch.md>)
- [19 BM1684和Arch.md](<mlir/19%20BM1684%E5%92%8CArch.md>)
- [20 getCoreNum获取核心.md](<mlir/20%20getCoreNum%E8%8E%B7%E5%8F%96%E6%A0%B8%E5%BF%83.md>)
- [21 MLIR命令行参数.md](<mlir/21%20MLIR%E5%91%BD%E4%BB%A4%E8%A1%8C%E5%8F%82%E6%95%B0.md>)
- [22 依然是MLIR命令行选项的例子.md](<mlir/22%20%E4%BE%9D%E7%84%B6%E6%98%AFMLIR%E5%91%BD%E4%BB%A4%E8%A1%8C%E9%80%89%E9%A1%B9%E7%9A%84%E4%BE%8B%E5%AD%90.md>)
- [22 基于LLVM的输出日志.md](<mlir/22%20%E5%9F%BA%E4%BA%8ELLVM%E7%9A%84%E8%BE%93%E5%87%BA%E6%97%A5%E5%BF%97.md>)
- [24 接口默认实现导致接口方法消失.md](<mlir/24%20%E6%8E%A5%E5%8F%A3%E9%BB%98%E8%AE%A4%E5%AE%9E%E7%8E%B0%E5%AF%BC%E8%87%B4%E6%8E%A5%E5%8F%A3%E6%96%B9%E6%B3%95%E6%B6%88%E5%A4%B1.md>)
- [25 复习开发流程.md](<mlir/25%20%E5%A4%8D%E4%B9%A0%E5%BC%80%E5%8F%91%E6%B5%81%E7%A8%8B.md>)
- [26 解析阶段.md](<mlir/26%20%E8%A7%A3%E6%9E%90%E9%98%B6%E6%AE%B5.md>)
- [27 InitPass和init关系.md](<mlir/27%20InitPass%E5%92%8Cinit%E5%85%B3%E7%B3%BB.md>)
- [28 inference().md](<mlir/28%20inference().md>)
- [29 top层下的Pass学习顺序.md](<mlir/29%20top%E5%B1%82%E4%B8%8B%E7%9A%84Pass%E5%AD%A6%E4%B9%A0%E9%A1%BA%E5%BA%8F.md>)
- [30 读取.mlir文件后生成的图结构.md](<mlir/30%20%E8%AF%BB%E5%8F%96.mlir%E6%96%87%E4%BB%B6%E5%90%8E%E7%94%9F%E6%88%90%E7%9A%84%E5%9B%BE%E7%BB%93%E6%9E%84.md>)
- [31 模型输入与模型内部的张量形状内容.md](<mlir/31%20%E6%A8%A1%E5%9E%8B%E8%BE%93%E5%85%A5%E4%B8%8E%E6%A8%A1%E5%9E%8B%E5%86%85%E9%83%A8%E7%9A%84%E5%BC%A0%E9%87%8F%E5%BD%A2%E7%8A%B6%E5%86%85%E5%AE%B9.md>)
- [32 MLIRContext和Builder.md](<mlir/32%20MLIRContext%E5%92%8CBuilder.md>)
- [33 block.md](<mlir/33%20block.md>)
- [34 dyn_cast.md](<mlir/34%20dyn_cast.md>)
- [35 Operaion.md](<mlir/35%20Operaion.md>)
- [36 Operation walk().md](<mlir/36%20Operation%20walk().md>)
- [37 设置新的tensor类型 get方法.md](<mlir/37%20%E8%AE%BE%E7%BD%AE%E6%96%B0%E7%9A%84tensor%E7%B1%BB%E5%9E%8B%20get%E6%96%B9%E6%B3%95.md>)
- [38 mlir的loc.md](<mlir/38%20mlir%E7%9A%84loc.md>)
- [39 Type相关（更深层理解）.md](<mlir/39%20Type%E7%9B%B8%E5%85%B3%EF%BC%88%E6%9B%B4%E6%B7%B1%E5%B1%82%E7%90%86%E8%A7%A3%EF%BC%89.md>)
- [40 CalibratedQuantizedType.md](<mlir/40%20CalibratedQuantizedType.md>)
- [43 多于4维的情况.md](<mlir/43%20%E5%A4%9A%E4%BA%8E4%E7%BB%B4%E7%9A%84%E6%83%85%E5%86%B5.md>)
- [44 llvm ArrayRef相关内容.md](<mlir/44%20llvm%20ArrayRef%E7%9B%B8%E5%85%B3%E5%86%85%E5%AE%B9.md>)
- [45 std get()方法 pair和tuple.md](<mlir/45%20std%20get()%E6%96%B9%E6%B3%95%20pair%E5%92%8Ctuple.md>)
- [46 std shared_ptr.md](<mlir/46%20std%20shared_ptr.md>)
- [47 ArrayAttr类相关.md](<mlir/47%20ArrayAttr%E7%B1%BB%E7%9B%B8%E5%85%B3.md>)
- [48 llvm enumerate是什么.md](<mlir/48%20llvm%20enumerate%E6%98%AF%E4%BB%80%E4%B9%88.md>)
- [49 llvm_unreachable.md](<mlir/49%20llvm_unreachable.md>)
- [50 预处理Pass流程.md](<mlir/50%20%E9%A2%84%E5%A4%84%E7%90%86Pass%E6%B5%81%E7%A8%8B.md>)
- [51 编译时链接错误，涉及mlir的库.md](<mlir/51%20%E7%BC%96%E8%AF%91%E6%97%B6%E9%93%BE%E6%8E%A5%E9%94%99%E8%AF%AF%EF%BC%8C%E6%B6%89%E5%8F%8Amlir%E7%9A%84%E5%BA%93.md>)
- [52 运行带选项的Pass.md](<mlir/52%20%E8%BF%90%E8%A1%8C%E5%B8%A6%E9%80%89%E9%A1%B9%E7%9A%84Pass.md>)
- [53 mlir未修改函数签名带来的问题.md](<mlir/53%20mlir%E6%9C%AA%E4%BF%AE%E6%94%B9%E5%87%BD%E6%95%B0%E7%AD%BE%E5%90%8D%E5%B8%A6%E6%9D%A5%E7%9A%84%E9%97%AE%E9%A2%98.md>)
- [58 EnumAttr的定义误导性.md](<mlir/58%20EnumAttr%E7%9A%84%E5%AE%9A%E4%B9%89%E8%AF%AF%E5%AF%BC%E6%80%A7.md>)
- [59 属性自己的get方法和Builder的get方法.md](<mlir/59%20%E5%B1%9E%E6%80%A7%E8%87%AA%E5%B7%B1%E7%9A%84get%E6%96%B9%E6%B3%95%E5%92%8CBuilder%E7%9A%84get%E6%96%B9%E6%B3%95.md>)
- [60 pattern的应用层面.md](<mlir/60%20pattern%E7%9A%84%E5%BA%94%E7%94%A8%E5%B1%82%E9%9D%A2.md>)
- [61 patternEx的使用.md](<mlir/61%20patternEx%E7%9A%84%E4%BD%BF%E7%94%A8.md>)
- [62 op.getInput.md](<mlir/62%20op.getInput.md>)
- [63 PatternWriter提供的功能，类比OpBuilder.md](<mlir/63%20PatternWriter%E6%8F%90%E4%BE%9B%E7%9A%84%E5%8A%9F%E8%83%BD%EF%BC%8C%E7%B1%BB%E6%AF%94OpBuilder.md>)
- [64 在Op中属性常见的coeff是做什么呢？.md](<mlir/64%20%E5%9C%A8Op%E4%B8%AD%E5%B1%9E%E6%80%A7%E5%B8%B8%E8%A7%81%E7%9A%84coeff%E6%98%AF%E5%81%9A%E4%BB%80%E4%B9%88%E5%91%A2%EF%BC%9F.md>)
- [65 PatternSet.md](<mlir/65%20PatternSet.md>)
- [66 Value类中的迭代器使用.md](<mlir/66%20Value%E7%B1%BB%E4%B8%AD%E7%9A%84%E8%BF%AD%E4%BB%A3%E5%99%A8%E4%BD%BF%E7%94%A8.md>)

</details>

<details>
<summary><strong>后端代码/</strong> · 5 个子目录 · 57 个文件</summary>

记录 TPU 后端、cmodel、矩阵乘法、BD 流程、数据迁移、SFU、libsophon 与整体项目解析。

- **libsophon代码/** — 1 个文件
  - [01 库.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/libsophon%E4%BB%A3%E7%A0%81/01%20%E5%BA%93.md>)
- **sfu/** — 4 个文件
  - [02 浮点数存储.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/sfu/02%20%E6%B5%AE%E7%82%B9%E6%95%B0%E5%AD%98%E5%82%A8.md>)
  - [03 收敛指的是什么？.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/sfu/03%20%E6%94%B6%E6%95%9B%E6%8C%87%E7%9A%84%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%9F.md>)
  - [04 sfu 典型实现方式.docx](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/sfu/04%20sfu%20%E5%85%B8%E5%9E%8B%E5%AE%9E%E7%8E%B0%E6%96%B9%E5%BC%8F.docx>)
  - [04 sfu底层算子.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/sfu/04%20sfu%E5%BA%95%E5%B1%82%E7%AE%97%E5%AD%90.md>)
- **sfu表格/** — 3 个文件
  - [SFU_ANALYSIS_COMPLETION_REPORT.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/sfu%E8%A1%A8%E6%A0%BC/SFU_ANALYSIS_COMPLETION_REPORT.md>)
  - [SFU_OPERATORS_ANALYSIS.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/sfu%E8%A1%A8%E6%A0%BC/SFU_OPERATORS_ANALYSIS.md>)
  - [SFU_OPERATORS_QUICK_REFERENCE.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/sfu%E8%A1%A8%E6%A0%BC/SFU_OPERATORS_QUICK_REFERENCE.md>)
- **整体项目解析/** — 2 个子目录，22 个文件
  - **tpuDNN分析/** — 1 个子目录，8 个文件
    - **conv算子的映射关系/** — 1 个文件
      - [conv算子内存映射关系.pptx](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/%E6%95%B4%E4%BD%93%E9%A1%B9%E7%9B%AE%E8%A7%A3%E6%9E%90/tpuDNN%E5%88%86%E6%9E%90/conv%E7%AE%97%E5%AD%90%E7%9A%84%E6%98%A0%E5%B0%84%E5%85%B3%E7%B3%BB/conv%E7%AE%97%E5%AD%90%E5%86%85%E5%AD%98%E6%98%A0%E5%B0%84%E5%85%B3%E7%B3%BB.pptx>)
    - [01 readme解析.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/%E6%95%B4%E4%BD%93%E9%A1%B9%E7%9B%AE%E8%A7%A3%E6%9E%90/tpuDNN%E5%88%86%E6%9E%90/01%20readme%E8%A7%A3%E6%9E%90.md>)
    - [02 testTpudnnConvFloat.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/%E6%95%B4%E4%BD%93%E9%A1%B9%E7%9B%AE%E8%A7%A3%E6%9E%90/tpuDNN%E5%88%86%E6%9E%90/02%20testTpudnnConvFloat.md>)
    - [03 testTpudnnConvFloat1.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/%E6%95%B4%E4%BD%93%E9%A1%B9%E7%9B%AE%E8%A7%A3%E6%9E%90/tpuDNN%E5%88%86%E6%9E%90/03%20testTpudnnConvFloat1.md>)
    - [04 卷积.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/%E6%95%B4%E4%BD%93%E9%A1%B9%E7%9B%AE%E8%A7%A3%E6%9E%90/tpuDNN%E5%88%86%E6%9E%90/04%20%E5%8D%B7%E7%A7%AF.md>)
    - [05 rt相关.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/%E6%95%B4%E4%BD%93%E9%A1%B9%E7%9B%AE%E8%A7%A3%E6%9E%90/tpuDNN%E5%88%86%E6%9E%90/05%20rt%E7%9B%B8%E5%85%B3.md>)
    - [06 include库文件的内容.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/%E6%95%B4%E4%BD%93%E9%A1%B9%E7%9B%AE%E8%A7%A3%E6%9E%90/tpuDNN%E5%88%86%E6%9E%90/06%20include%E5%BA%93%E6%96%87%E4%BB%B6%E7%9A%84%E5%86%85%E5%AE%B9.md>)
    - [07 bmlib 的流程.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/%E6%95%B4%E4%BD%93%E9%A1%B9%E7%9B%AE%E8%A7%A3%E6%9E%90/tpuDNN%E5%88%86%E6%9E%90/07%20bmlib%20%E7%9A%84%E6%B5%81%E7%A8%8B.md>)
  - **编译过程解析/** — 6 个文件
    - [01 rebuild_firmware_cmodel.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/%E6%95%B4%E4%BD%93%E9%A1%B9%E7%9B%AE%E8%A7%A3%E6%9E%90/%E7%BC%96%E8%AF%91%E8%BF%87%E7%A8%8B%E8%A7%A3%E6%9E%90/01%20rebuild_firmware_cmodel.md>)
    - [02 rebuild_all和build_all.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/%E6%95%B4%E4%BD%93%E9%A1%B9%E7%9B%AE%E8%A7%A3%E6%9E%90/%E7%BC%96%E8%AF%91%E8%BF%87%E7%A8%8B%E8%A7%A3%E6%9E%90/02%20rebuild_all%E5%92%8Cbuild_all.md>)
    - [03 build_test里调用的build_all分析.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/%E6%95%B4%E4%BD%93%E9%A1%B9%E7%9B%AE%E8%A7%A3%E6%9E%90/%E7%BC%96%E8%AF%91%E8%BF%87%E7%A8%8B%E8%A7%A3%E6%9E%90/03%20build_test%E9%87%8C%E8%B0%83%E7%94%A8%E7%9A%84build_all%E5%88%86%E6%9E%90.md>)
    - [04 直接调用rebuild_test的输出.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/%E6%95%B4%E4%BD%93%E9%A1%B9%E7%9B%AE%E8%A7%A3%E6%9E%90/%E7%BC%96%E8%AF%91%E8%BF%87%E7%A8%8B%E8%A7%A3%E6%9E%90/04%20%E7%9B%B4%E6%8E%A5%E8%B0%83%E7%94%A8rebuild_test%E7%9A%84%E8%BE%93%E5%87%BA.md>)
    - [05 build_helper.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/%E6%95%B4%E4%BD%93%E9%A1%B9%E7%9B%AE%E8%A7%A3%E6%9E%90/%E7%BC%96%E8%AF%91%E8%BF%87%E7%A8%8B%E8%A7%A3%E6%9E%90/05%20build_helper.md>)
    - [06 调用rebuild_test选项.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/%E6%95%B4%E4%BD%93%E9%A1%B9%E7%9B%AE%E8%A7%A3%E6%9E%90/%E7%BC%96%E8%AF%91%E8%BF%87%E7%A8%8B%E8%A7%A3%E6%9E%90/06%20%E8%B0%83%E7%94%A8rebuild_test%E9%80%89%E9%A1%B9.md>)
  - [02 其中间接调用.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/%E6%95%B4%E4%BD%93%E9%A1%B9%E7%9B%AE%E8%A7%A3%E6%9E%90/02%20%E5%85%B6%E4%B8%AD%E9%97%B4%E6%8E%A5%E8%B0%83%E7%94%A8.md>)
  - [03 外层cmakelist全解析.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/%E6%95%B4%E4%BD%93%E9%A1%B9%E7%9B%AE%E8%A7%A3%E6%9E%90/03%20%E5%A4%96%E5%B1%82cmakelist%E5%85%A8%E8%A7%A3%E6%9E%90.md>)
  - [04 test.sh分析.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/%E6%95%B4%E4%BD%93%E9%A1%B9%E7%9B%AE%E8%A7%A3%E6%9E%90/04%20test.sh%E5%88%86%E6%9E%90.md>)
  - [05 builder_helper.sh分析.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/%E6%95%B4%E4%BD%93%E9%A1%B9%E7%9B%AE%E8%A7%A3%E6%9E%90/05%20builder_helper.sh%E5%88%86%E6%9E%90.md>)
  - [06 build_backend.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/%E6%95%B4%E4%BD%93%E9%A1%B9%E7%9B%AE%E8%A7%A3%E6%9E%90/06%20build_backend.md>)
  - [07 bmlib分析.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/%E6%95%B4%E4%BD%93%E9%A1%B9%E7%9B%AE%E8%A7%A3%E6%9E%90/07%20bmlib%E5%88%86%E6%9E%90.md>)
  - [08 TPUDNNHost.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/%E6%95%B4%E4%BD%93%E9%A1%B9%E7%9B%AE%E8%A7%A3%E6%9E%90/08%20TPUDNNHost.md>)
  - [错误记录.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/%E6%95%B4%E4%BD%93%E9%A1%B9%E7%9B%AE%E8%A7%A3%E6%9E%90/%E9%94%99%E8%AF%AF%E8%AE%B0%E5%BD%95.md>)
- **测试的两种方式/** — 1 个文件
  - [readme提到的测试方式.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/%E6%B5%8B%E8%AF%95%E7%9A%84%E4%B8%A4%E7%A7%8D%E6%96%B9%E5%BC%8F/readme%E6%8F%90%E5%88%B0%E7%9A%84%E6%B5%8B%E8%AF%95%E6%96%B9%E5%BC%8F.md>)
- [01 cmodel_init.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/01%20cmodel_init.md>)
- [03 CONTINUOUS_MEM .md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/03%20CONTINUOUS_MEM%20.md>)
- [04 thread part 变量.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/04%20thread%20part%20%E5%8F%98%E9%87%8F.md>)
- [05 时间戳.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/05%20%E6%97%B6%E9%97%B4%E6%88%B3.md>)
- [06 为矩阵分配地址.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/06%20%E4%B8%BA%E7%9F%A9%E9%98%B5%E5%88%86%E9%85%8D%E5%9C%B0%E5%9D%80.md>)
- [07 对齐到a倍数的公式.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/07%20%E5%AF%B9%E9%BD%90%E5%88%B0a%E5%80%8D%E6%95%B0%E7%9A%84%E5%85%AC%E5%BC%8F.md>)
- [08 存放数据按照EU宽度来放.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/08%20%E5%AD%98%E6%94%BE%E6%95%B0%E6%8D%AE%E6%8C%89%E7%85%A7EU%E5%AE%BD%E5%BA%A6%E6%9D%A5%E6%94%BE.md>)
- [09 c_stride.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/09%20c_stride.md>)
- [11 生成随机数据的方式.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/11%20%E7%94%9F%E6%88%90%E9%9A%8F%E6%9C%BA%E6%95%B0%E6%8D%AE%E7%9A%84%E6%96%B9%E5%BC%8F.md>)
- [12 FP32强转u32.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/12%20FP32%E5%BC%BA%E8%BD%ACu32.md>)
- [13 矩阵乘法逻辑.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/13%20%E7%9F%A9%E9%98%B5%E4%B9%98%E6%B3%95%E9%80%BB%E8%BE%91.md>)
- [14 通用矩阵乘法.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/14%20%E9%80%9A%E7%94%A8%E7%9F%A9%E9%98%B5%E4%B9%98%E6%B3%95.md>)
- [15 矩阵信息转成在tpu里的格式.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/15%20%E7%9F%A9%E9%98%B5%E4%BF%A1%E6%81%AF%E8%BD%AC%E6%88%90%E5%9C%A8tpu%E9%87%8C%E7%9A%84%E6%A0%BC%E5%BC%8F.md>)
- [16 n_和c_详解.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/16%20n_%E5%92%8Cc_%E8%AF%A6%E8%A7%A3.md>)
- [17 数据迁移部分详解.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/17%20%E6%95%B0%E6%8D%AE%E8%BF%81%E7%A7%BB%E9%83%A8%E5%88%86%E8%AF%A6%E8%A7%A3.md>)
- [18 CMD_ID_NODE.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/18%20CMD_ID_NODE.md>)
- [19 初始化指令node名称的含义.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/19%20%E5%88%9D%E5%A7%8B%E5%8C%96%E6%8C%87%E4%BB%A4node%E5%90%8D%E7%A7%B0%E7%9A%84%E5%90%AB%E4%B9%89.md>)
- [20 BD逻辑.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/20%20BD%E9%80%BB%E8%BE%91.md>)
- [21 BD流程解析.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/21%20BD%E6%B5%81%E7%A8%8B%E8%A7%A3%E6%9E%90.md>)
- [22 宏拼接符号.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/22%20%E5%AE%8F%E6%8B%BC%E6%8E%A5%E7%AC%A6%E5%8F%B7.md>)
- [23 gen_cmd_t生成指令的结构体.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/23%20gen_cmd_t%E7%94%9F%E6%88%90%E6%8C%87%E4%BB%A4%E7%9A%84%E7%BB%93%E6%9E%84%E4%BD%93.md>)
- [24 为什么判断FIFO是否满的代码没实现.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/24%20%E4%B8%BA%E4%BB%80%E4%B9%88%E5%88%A4%E6%96%ADFIFO%E6%98%AF%E5%90%A6%E6%BB%A1%E7%9A%84%E4%BB%A3%E7%A0%81%E6%B2%A1%E5%AE%9E%E7%8E%B0.md>)
- [25 tvgen模式是什么.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/25%20tvgen%E6%A8%A1%E5%BC%8F%E6%98%AF%E4%BB%80%E4%B9%88.md>)
- [26 矩阵乘法搭配指令.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/26%20%E7%9F%A9%E9%98%B5%E4%B9%98%E6%B3%95%E6%90%AD%E9%85%8D%E6%8C%87%E4%BB%A4.md>)
- [27 等待命令执行完在当前cmodel情况下并没有效果.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/27%20%E7%AD%89%E5%BE%85%E5%91%BD%E4%BB%A4%E6%89%A7%E8%A1%8C%E5%AE%8C%E5%9C%A8%E5%BD%93%E5%89%8Dcmodel%E6%83%85%E5%86%B5%E4%B8%8B%E5%B9%B6%E6%B2%A1%E6%9C%89%E6%95%88%E6%9E%9C.md>)
- [28 寄存器查找修改步骤.md](<%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81/28%20%E5%AF%84%E5%AD%98%E5%99%A8%E6%9F%A5%E6%89%BE%E4%BF%AE%E6%94%B9%E6%AD%A5%E9%AA%A4.md>)

</details>

<details>
<summary><strong>实际配置安装环境相关/</strong> · 1 个子目录 · 7 个文件</summary>

记录 TPU-MLIR、NVIDIA、Docker、dpkg、pip、算力卡编译等环境配置与故障处理。

- **docker相关/** — 2 个文件
  - [01 docker命令模式.md](<%E5%AE%9E%E9%99%85%E9%85%8D%E7%BD%AE%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83%E7%9B%B8%E5%85%B3/docker%E7%9B%B8%E5%85%B3/01%20docker%E5%91%BD%E4%BB%A4%E6%A8%A1%E5%BC%8F.md>)
  - [02 docker常用命令.md](<%E5%AE%9E%E9%99%85%E9%85%8D%E7%BD%AE%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83%E7%9B%B8%E5%85%B3/docker%E7%9B%B8%E5%85%B3/02%20docker%E5%B8%B8%E7%94%A8%E5%91%BD%E4%BB%A4.md>)
- [01 tpu-mlir和lib.md](<%E5%AE%9E%E9%99%85%E9%85%8D%E7%BD%AE%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83%E7%9B%B8%E5%85%B3/01%20tpu-mlir%E5%92%8Clib.md>)
- [02 NVIDIA.md](<%E5%AE%9E%E9%99%85%E9%85%8D%E7%BD%AE%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83%E7%9B%B8%E5%85%B3/02%20NVIDIA.md>)
- [03 dpkg+.deb.md](<%E5%AE%9E%E9%99%85%E9%85%8D%E7%BD%AE%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83%E7%9B%B8%E5%85%B3/03%20dpkg%2B.deb.md>)
- [04 pip.md](<%E5%AE%9E%E9%99%85%E9%85%8D%E7%BD%AE%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83%E7%9B%B8%E5%85%B3/04%20pip.md>)
- [编译到算力卡错误内容.md](<%E5%AE%9E%E9%99%85%E9%85%8D%E7%BD%AE%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83%E7%9B%B8%E5%85%B3/%E7%BC%96%E8%AF%91%E5%88%B0%E7%AE%97%E5%8A%9B%E5%8D%A1%E9%94%99%E8%AF%AF%E5%86%85%E5%AE%B9.md>)

</details>

<details>
<summary><strong>开学前/</strong> · 0 个子目录 · 55 个文件</summary>

早期集中学习笔记，覆盖 AI 编译全景、ONNX、MLIR、Pass、TPU-MLIR、pybind、CMake、Docker 等。

- [AbsAdd接口.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/AbsAdd%E6%8E%A5%E5%8F%A3.md>)
- [AddOp生成.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/AddOp%E7%94%9F%E6%88%90.md>)
- [Add执行过程.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/Add%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B.md>)
- [AI编译全景.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/AI%E7%BC%96%E8%AF%91%E5%85%A8%E6%99%AF.md>)
- [C++与模型.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/C%2B%2B%E4%B8%8E%E6%A8%A1%E5%9E%8B.md>)
- [C++工程结构.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/C%2B%2B%E5%B7%A5%E7%A8%8B%E7%BB%93%E6%9E%84.md>)
- [CMake变量.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/CMake%E5%8F%98%E9%87%8F.md>)
- [CMake生成链.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/CMake%E7%94%9F%E6%88%90%E9%93%BE.md>)
- [DNNL封装.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/DNNL%E5%B0%81%E8%A3%85.md>)
- [Docker命令.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/Docker%E5%91%BD%E4%BB%A4.md>)
- [FFN到运行.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/FFN%E5%88%B0%E8%BF%90%E8%A1%8C.md>)
- [FLOPs接口.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/FLOPs%E6%8E%A5%E5%8F%A3.md>)
- [LLVM源码构建.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/LLVM%E6%BA%90%E7%A0%81%E6%9E%84%E5%BB%BA.md>)
- [MatMul融合.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/MatMul%E8%9E%8D%E5%90%88.md>)
- [MemRef练习.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/MemRef%E7%BB%83%E4%B9%A0.md>)
- [MLIR基本语法.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/MLIR%E5%9F%BA%E6%9C%AC%E8%AF%AD%E6%B3%95.md>)
- [ONNX导出坑.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/ONNX%E5%AF%BC%E5%87%BA%E5%9D%91.md>)
- [ONNX算子谱.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/ONNX%E7%AE%97%E5%AD%90%E8%B0%B1.md>)
- [ONNX节点转.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/ONNX%E8%8A%82%E7%82%B9%E8%BD%AC.md>)
- [Op和TD.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/Op%E5%92%8CTD.md>)
- [Op嵌套树.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/Op%E5%B5%8C%E5%A5%97%E6%A0%91.md>)
- [Op接口表.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/Op%E6%8E%A5%E5%8F%A3%E8%A1%A8.md>)
- [Op测试.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/Op%E6%B5%8B%E8%AF%95.md>)
- [pybind构建.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/pybind%E6%9E%84%E5%BB%BA.md>)
- [Python类.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/Python%E7%B1%BB.md>)
- [Python路径.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/Python%E8%B7%AF%E5%BE%84.md>)
- [Py类与RMS.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/Py%E7%B1%BB%E4%B8%8ERMS.md>)
- [RMS输入图.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/RMS%E8%BE%93%E5%85%A5%E5%9B%BE.md>)
- [runner解析.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/runner%E8%A7%A3%E6%9E%90.md>)
- [SCF控制流.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/SCF%E6%8E%A7%E5%88%B6%E6%B5%81.md>)
- [TD生成目标.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/TD%E7%94%9F%E6%88%90%E7%9B%AE%E6%A0%87.md>)
- [TopToTpu.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/TopToTpu.md>)
- [Top接口复盘.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/Top%E6%8E%A5%E5%8F%A3%E5%A4%8D%E7%9B%98.md>)
- [TPUAdd接口.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/TPUAdd%E6%8E%A5%E5%8F%A3.md>)
- [tpuc入口.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/tpuc%E5%85%A5%E5%8F%A3.md>)
- [TPU源码导读.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/TPU%E6%BA%90%E7%A0%81%E5%AF%BC%E8%AF%BB.md>)
- [TPU量化初识.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/TPU%E9%87%8F%E5%8C%96%E5%88%9D%E8%AF%86.md>)
- [优化Pass.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/%E4%BC%98%E5%8C%96Pass.md>)
- [内置TD导读.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/%E5%86%85%E7%BD%AETD%E5%AF%BC%E8%AF%BB.md>)
- [单层算子.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/%E5%8D%95%E5%B1%82%E7%AE%97%E5%AD%90.md>)
- [子图形状推理.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/%E5%AD%90%E5%9B%BE%E5%BD%A2%E7%8A%B6%E6%8E%A8%E7%90%86.md>)
- [张量类型统计.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/%E5%BC%A0%E9%87%8F%E7%B1%BB%E5%9E%8B%E7%BB%9F%E8%AE%A1.md>)
- [手写MLIR.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/%E6%89%8B%E5%86%99MLIR.md>)
- [推理参数复习.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/%E6%8E%A8%E7%90%86%E5%8F%82%E6%95%B0%E5%A4%8D%E4%B9%A0.md>)
- [方言加载链.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/%E6%96%B9%E8%A8%80%E5%8A%A0%E8%BD%BD%E9%93%BE.md>)
- [方言接口Pass.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/%E6%96%B9%E8%A8%80%E6%8E%A5%E5%8F%A3Pass.md>)
- [方言是什么.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/%E6%96%B9%E8%A8%80%E6%98%AF%E4%BB%80%E4%B9%88.md>)
- [构建器比较.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/%E6%9E%84%E5%BB%BA%E5%99%A8%E6%AF%94%E8%BE%83.md>)
- [模型转脚本.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/%E6%A8%A1%E5%9E%8B%E8%BD%AC%E8%84%9A%E6%9C%AC.md>)
- [模式是什么.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/%E6%A8%A1%E5%BC%8F%E6%98%AF%E4%BB%80%E4%B9%88.md>)
- [模式重写.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/%E6%A8%A1%E5%BC%8F%E9%87%8D%E5%86%99.md>)
- [注册与工厂.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/%E6%B3%A8%E5%86%8C%E4%B8%8E%E5%B7%A5%E5%8E%82.md>)
- [算子爆炸.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/%E7%AE%97%E5%AD%90%E7%88%86%E7%82%B8.md>)
- [类型和属性.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/%E7%B1%BB%E5%9E%8B%E5%92%8C%E5%B1%9E%E6%80%A7.md>)
- [自定义Pass.md](<%E5%BC%80%E5%AD%A6%E5%89%8D/%E8%87%AA%E5%AE%9A%E4%B9%89Pass.md>)

</details>

<details>
<summary><strong>拆解TPU-MLIR/</strong> · 0 个子目录 · 4 个文件</summary>

围绕 TPU-MLIR 源码流程进行阶段性拆解，如 shape-infer、init Pass、bmodel 生成等。

- [A 生成Bmodel.md](<%E6%8B%86%E8%A7%A3TPU-MLIR/A%20%E7%94%9F%E6%88%90Bmodel.md>)
- [OpRewriterPatternEx封装.md](<%E6%8B%86%E8%A7%A3TPU-MLIR/OpRewriterPatternEx%E5%B0%81%E8%A3%85.md>)
- [关于init Pass是做什么的呢.md](<%E6%8B%86%E8%A7%A3TPU-MLIR/%E5%85%B3%E4%BA%8Einit%20Pass%E6%98%AF%E5%81%9A%E4%BB%80%E4%B9%88%E7%9A%84%E5%91%A2.md>)
- [阶段性总结 shape-infer功能基本完成.md](<%E6%8B%86%E8%A7%A3TPU-MLIR/%E9%98%B6%E6%AE%B5%E6%80%A7%E6%80%BB%E7%BB%93%20shape-infer%E5%8A%9F%E8%83%BD%E5%9F%BA%E6%9C%AC%E5%AE%8C%E6%88%90.md>)

</details>

<details>
<summary><strong>服务器登录报错问题/</strong> · 0 个子目录 · 1 个文件</summary>

记录服务器登录与排错流程。

- [流程记录.md](<%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%99%BB%E5%BD%95%E6%8A%A5%E9%94%99%E9%97%AE%E9%A2%98/%E6%B5%81%E7%A8%8B%E8%AE%B0%E5%BD%95.md>)

</details>

<details>
<summary><strong>硬件相关/</strong> · 3 个子目录 · 30 个文件</summary>

记录 AIACC 指令集、微指令、硬件模块、浮点流程、bank、UR/RtoVR、模块修改顺序等硬件相关内容。

- **UR, RtoVR等模块解析/** — 2 个文件
  - [01 UR流程解析.md](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/UR%2C%20RtoVR%E7%AD%89%E6%A8%A1%E5%9D%97%E8%A7%A3%E6%9E%90/01%20UR%E6%B5%81%E7%A8%8B%E8%A7%A3%E6%9E%90.md>)
  - [02 RtoVR流程解析.md](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/UR%2C%20RtoVR%E7%AD%89%E6%A8%A1%E5%9D%97%E8%A7%A3%E6%9E%90/02%20RtoVR%E6%B5%81%E7%A8%8B%E8%A7%A3%E6%9E%90.md>)
- **修改模块的顺序内容/** — 9 个文件
  - [FP16MUL.md](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/%E4%BF%AE%E6%94%B9%E6%A8%A1%E5%9D%97%E7%9A%84%E9%A1%BA%E5%BA%8F%E5%86%85%E5%AE%B9/FP16MUL.md>)
  - [FP32ADDTREE8TO1相关的微指令.md](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/%E4%BF%AE%E6%94%B9%E6%A8%A1%E5%9D%97%E7%9A%84%E9%A1%BA%E5%BA%8F%E5%86%85%E5%AE%B9/FP32ADDTREE8TO1%E7%9B%B8%E5%85%B3%E7%9A%84%E5%BE%AE%E6%8C%87%E4%BB%A4.md>)
  - [FPADD.md](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/%E4%BF%AE%E6%94%B9%E6%A8%A1%E5%9D%97%E7%9A%84%E9%A1%BA%E5%BA%8F%E5%86%85%E5%AE%B9/FPADD.md>)
  - [FPtoFP.md](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/%E4%BF%AE%E6%94%B9%E6%A8%A1%E5%9D%97%E7%9A%84%E9%A1%BA%E5%BA%8F%E5%86%85%E5%AE%B9/FPtoFP.md>)
  - [FPtoINT.md](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/%E4%BF%AE%E6%94%B9%E6%A8%A1%E5%9D%97%E7%9A%84%E9%A1%BA%E5%BA%8F%E5%86%85%E5%AE%B9/FPtoINT.md>)
  - [INTADD.md](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/%E4%BF%AE%E6%94%B9%E6%A8%A1%E5%9D%97%E7%9A%84%E9%A1%BA%E5%BA%8F%E5%86%85%E5%AE%B9/INTADD.md>)
  - [INTtoFP.md](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/%E4%BF%AE%E6%94%B9%E6%A8%A1%E5%9D%97%E7%9A%84%E9%A1%BA%E5%BA%8F%E5%86%85%E5%AE%B9/INTtoFP.md>)
  - [INTtoINT.md](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/%E4%BF%AE%E6%94%B9%E6%A8%A1%E5%9D%97%E7%9A%84%E9%A1%BA%E5%BA%8F%E5%86%85%E5%AE%B9/INTtoINT.md>)
  - [LOGIC.md](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/%E4%BF%AE%E6%94%B9%E6%A8%A1%E5%9D%97%E7%9A%84%E9%A1%BA%E5%BA%8F%E5%86%85%E5%AE%B9/LOGIC.md>)
- **硬件架构信息/** — 3 个文件
  - [AIACC 指令集架构附录A v0.1.0.docx](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/%E7%A1%AC%E4%BB%B6%E6%9E%B6%E6%9E%84%E4%BF%A1%E6%81%AF/AIACC%20%E6%8C%87%E4%BB%A4%E9%9B%86%E6%9E%B6%E6%9E%84%E9%99%84%E5%BD%95A%20v0.1.0.docx>)
  - [AIACC 架构介绍 v0.1.2.pptx](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/%E7%A1%AC%E4%BB%B6%E6%9E%B6%E6%9E%84%E4%BF%A1%E6%81%AF/AIACC%20%E6%9E%B6%E6%9E%84%E4%BB%8B%E7%BB%8D%20v0.1.2.pptx>)
  - [RtoVR连接关系 v0.1.1.xlsx](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/%E7%A1%AC%E4%BB%B6%E6%9E%B6%E6%9E%84%E4%BF%A1%E6%81%AF/RtoVR%E8%BF%9E%E6%8E%A5%E5%85%B3%E7%B3%BB%20v0.1.1.xlsx>)
- [####重要 更新rsqrt的实现记录.md](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/%23%23%23%23%E9%87%8D%E8%A6%81%20%E6%9B%B4%E6%96%B0rsqrt%E7%9A%84%E5%AE%9E%E7%8E%B0%E8%AE%B0%E5%BD%95.md>)
- [01 MID SMI SMC LSU.md](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/01%20MID%20SMI%20SMC%20LSU.md>)
- [02 宏指令和微指令.md](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/02%20%E5%AE%8F%E6%8C%87%E4%BB%A4%E5%92%8C%E5%BE%AE%E6%8C%87%E4%BB%A4.md>)
- [03 SMI-IQ是什么.md](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/03%20SMI-IQ%E6%98%AF%E4%BB%80%E4%B9%88.md>)
- [04 aten是什么.md](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/04%20aten%E6%98%AF%E4%BB%80%E4%B9%88.md>)
- [05 写rsqrt流程.md](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/05%20%E5%86%99rsqrt%E6%B5%81%E7%A8%8B.md>)
- [06 intadd_iq这种的功能是什么.md](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/06%20intadd_iq%E8%BF%99%E7%A7%8D%E7%9A%84%E5%8A%9F%E8%83%BD%E6%98%AF%E4%BB%80%E4%B9%88.md>)
- [07 SMIIQ微指令长度并不固定.md](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/07%20SMIIQ%E5%BE%AE%E6%8C%87%E4%BB%A4%E9%95%BF%E5%BA%A6%E5%B9%B6%E4%B8%8D%E5%9B%BA%E5%AE%9A.md>)
- [08 浮点数加法展示指令流程.md](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/08%20%E6%B5%AE%E7%82%B9%E6%95%B0%E5%8A%A0%E6%B3%95%E5%B1%95%E7%A4%BA%E6%8C%87%E4%BB%A4%E6%B5%81%E7%A8%8B.md>)
- [09 bank是什么？.md](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/09%20bank%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%9F.md>)
- [AIACC 指令集架构 v0.3.15.docx](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/AIACC%20%E6%8C%87%E4%BB%A4%E9%9B%86%E6%9E%B6%E6%9E%84%20v0.3.15.docx>)
- [AIACC 指令集架构 v0.3.4.docx](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/AIACC%20%E6%8C%87%E4%BB%A4%E9%9B%86%E6%9E%B6%E6%9E%84%20v0.3.4.docx>)
- [AIACC 指令集架构 v0.4.3.docx](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/AIACC%20%E6%8C%87%E4%BB%A4%E9%9B%86%E6%9E%B6%E6%9E%84%20v0.4.3.docx>)
- [AIACC_指令集架构_v0.4.15.docx](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/AIACC_%E6%8C%87%E4%BB%A4%E9%9B%86%E6%9E%B6%E6%9E%84_v0.4.15.docx>)
- [AIACC指令集架构.md](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/AIACC%E6%8C%87%E4%BB%A4%E9%9B%86%E6%9E%B6%E6%9E%84.md>)
- [指令集.md](<%E7%A1%AC%E4%BB%B6%E7%9B%B8%E5%85%B3/%E6%8C%87%E4%BB%A4%E9%9B%86.md>)

</details>

<details>
<summary><strong>神经网络/</strong> · 1 个子目录 · 10 个文件</summary>

整理 MLP、CNN、ResNet、opset、归一化、量化等神经网络基础知识。

- **模型量化/** — 3 个文件
  - [01 量化基本.md](<%E7%A5%9E%E7%BB%8F%E7%BD%91%E7%BB%9C/%E6%A8%A1%E5%9E%8B%E9%87%8F%E5%8C%96/01%20%E9%87%8F%E5%8C%96%E5%9F%BA%E6%9C%AC.md>)
  - [04 权重量化和激活值量化.md](<%E7%A5%9E%E7%BB%8F%E7%BD%91%E7%BB%9C/%E6%A8%A1%E5%9E%8B%E9%87%8F%E5%8C%96/04%20%E6%9D%83%E9%87%8D%E9%87%8F%E5%8C%96%E5%92%8C%E6%BF%80%E6%B4%BB%E5%80%BC%E9%87%8F%E5%8C%96.md>)
  - [05 关于QAT.md](<%E7%A5%9E%E7%BB%8F%E7%BD%91%E7%BB%9C/%E6%A8%A1%E5%9E%8B%E9%87%8F%E5%8C%96/05%20%E5%85%B3%E4%BA%8EQAT.md>)
- [01 MLP vs CNN.md](<%E7%A5%9E%E7%BB%8F%E7%BD%91%E7%BB%9C/01%20MLP%20vs%20CNN.md>)
- [02 什么叫全连接.md](<%E7%A5%9E%E7%BB%8F%E7%BD%91%E7%BB%9C/02%20%E4%BB%80%E4%B9%88%E5%8F%AB%E5%85%A8%E8%BF%9E%E6%8E%A5.md>)
- [03 层的概念.md](<%E7%A5%9E%E7%BB%8F%E7%BD%91%E7%BB%9C/03%20%E5%B1%82%E7%9A%84%E6%A6%82%E5%BF%B5.md>)
- [04 多层卷积的意义.md](<%E7%A5%9E%E7%BB%8F%E7%BD%91%E7%BB%9C/04%20%E5%A4%9A%E5%B1%82%E5%8D%B7%E7%A7%AF%E7%9A%84%E6%84%8F%E4%B9%89.md>)
- [05 ResNet的结构组织.md](<%E7%A5%9E%E7%BB%8F%E7%BD%91%E7%BB%9C/05%20ResNet%E7%9A%84%E7%BB%93%E6%9E%84%E7%BB%84%E7%BB%87.md>)
- [06 opset的概念.md](<%E7%A5%9E%E7%BB%8F%E7%BD%91%E7%BB%9C/06%20opset%E7%9A%84%E6%A6%82%E5%BF%B5.md>)
- [07 输入归一化操作.md](<%E7%A5%9E%E7%BB%8F%E7%BD%91%E7%BB%9C/07%20%E8%BE%93%E5%85%A5%E5%BD%92%E4%B8%80%E5%8C%96%E6%93%8D%E4%BD%9C.md>)

</details>

<details>
<summary><strong>论文（包括TVM详细内容）/</strong> · 2 个子目录 · 5 个文件</summary>

整理 TVM 相关论文、核心理念、与 TPU-MLIR 的对比，以及论文阅读资料。

- **APOLLO/** — 1 个文件
  - [论文.xlsx](<%E8%AE%BA%E6%96%87%EF%BC%88%E5%8C%85%E6%8B%ACTVM%E8%AF%A6%E7%BB%86%E5%86%85%E5%AE%B9%EF%BC%89/APOLLO/%E8%AE%BA%E6%96%87.xlsx>)
- **TVM/** — 4 个文件
  - [01 tvm.md](<%E8%AE%BA%E6%96%87%EF%BC%88%E5%8C%85%E6%8B%ACTVM%E8%AF%A6%E7%BB%86%E5%86%85%E5%AE%B9%EF%BC%89/TVM/01%20tvm.md>)
  - [02 tvm tpu-mlir区别.md](<%E8%AE%BA%E6%96%87%EF%BC%88%E5%8C%85%E6%8B%ACTVM%E8%AF%A6%E7%BB%86%E5%86%85%E5%AE%B9%EF%BC%89/TVM/02%20tvm%20tpu-mlir%E5%8C%BA%E5%88%AB.md>)
  - [03 tvm的核心理念.md](<%E8%AE%BA%E6%96%87%EF%BC%88%E5%8C%85%E6%8B%ACTVM%E8%AF%A6%E7%BB%86%E5%86%85%E5%AE%B9%EF%BC%89/TVM/03%20tvm%E7%9A%84%E6%A0%B8%E5%BF%83%E7%90%86%E5%BF%B5.md>)
  - [04 tvm的意义所在.md](<%E8%AE%BA%E6%96%87%EF%BC%88%E5%8C%85%E6%8B%ACTVM%E8%AF%A6%E7%BB%86%E5%86%85%E5%AE%B9%EF%BC%89/TVM/04%20tvm%E7%9A%84%E6%84%8F%E4%B9%89%E6%89%80%E5%9C%A8.md>)

</details>

## 总目录树。

```text
AI编译器学习记录/
├── AI编译器的初始思路/
│   ├── 01 kenel.md
│   ├── 02 定义算子包括几个方面.md
│   ├── 03 pytroch运行.md
│   ├── 04 量化区别.md
│   ├── 05 kernel重写.md
│   ├── 06 fbgemm.md
│   ├── 07 量化.md
│   ├── AI编译器类比传统编译器.md
│   └── 学习路线.md
├── cmake/
│   ├── 01 解决编译占用cpu爆掉的问题.md
│   ├── 02 链接中函数未定义错误.md
│   ├── 03 构建缓存问题.md
│   ├── 04 立即编译的工具.md
│   ├── 05 install.md
│   ├── 07 find_program.md
│   ├── 08 option.md
│   ├── 09 find_package.md
│   ├── 10 覆盖选项.md
│   ├── 11 install动态.md
│   ├── 12 test.md
│   ├── 13 strequal.md
│   └── Cmake相关命令.md
├── git/
│   ├── 01 git 基本使用.md
│   ├── 02 git 分支操作.md
│   ├── 03 push命令解析.md
│   ├── 04 git的merge.md
│   ├── 05 团队协作使用fork的内容.md
│   └── 06 解决冲突问题.md
├── mlir/
│   ├── llvm提供的一些内容/
│   │   ├── 01 llvm StringRef 的 lower方法.md
│   │   └── 02 llvm StringSwitch方法.md
│   ├── mlir文件的相关语法/
│   │   ├── 01 MLIR的核心：Operation.md
│   │   └── 02 module.md
│   ├── mlir相关cmake命令/
│   │   ├── mlir_tablegen.md
│   │   └── 包装命令.md
│   ├── shape-inferpass代码/
│   │   ├── 30 TupleFusePattern代码内容.md
│   │   ├── 32 UnTupleFusePattern代码内容.md
│   │   └── 33 CopyMultiUseWeight代码内容.md
│   ├── 中期写的部分，可参考/
│   │   ├── addOperation.md
│   │   ├── attribute.md
│   │   ├── InputOp.md
│   │   ├── mlir自带的pattern注册使用.md
│   │   ├── op的定义相关，生成的属性存在.md
│   │   ├── Pass.md
│   │   ├── PassManager和Pipline.md
│   │   ├── tablegen语法.md
│   │   └── Type.md
│   ├── 01 生成属性的名称.md
│   ├── 02 arguments和results.md
│   ├── 03 arguments参数联想问题.md
│   ├── 04 trait补充.md
│   ├── 05 trait写.md
│   ├── 06 定义AddConst.md
│   ├── 07 关于TPU-MLIR的attr.md
│   ├── 08 Pass中的option.md
│   ├── 09 tpu-mlir中的Pass设计.md
│   ├── 10 tpu-mlir里module设计.md
│   ├── 11 Attr.md
│   ├── 12 td声明的接口.md
│   ├── 13 Operation里的getAttrOfType.md
│   ├── 14 枚举属性类的定义补充.md
│   ├── 15 定义枚举附带方法.md
│   ├── 16 Arch.h.md
│   ├── 17 Arch.h.md
│   ├── 18 Arch.md
│   ├── 19 BM1684和Arch.md
│   ├── 20 getCoreNum获取核心.md
│   ├── 21 MLIR命令行参数.md
│   ├── 22 依然是MLIR命令行选项的例子.md
│   ├── 22 基于LLVM的输出日志.md
│   ├── 24 接口默认实现导致接口方法消失.md
│   ├── 25 复习开发流程.md
│   ├── 26 解析阶段.md
│   ├── 27 InitPass和init关系.md
│   ├── 28 inference().md
│   ├── 29 top层下的Pass学习顺序.md
│   ├── 30 读取.mlir文件后生成的图结构.md
│   ├── 31 模型输入与模型内部的张量形状内容.md
│   ├── 32 MLIRContext和Builder.md
│   ├── 33 block.md
│   ├── 34 dyn_cast.md
│   ├── 35 Operaion.md
│   ├── 36 Operation walk().md
│   ├── 37 设置新的tensor类型 get方法.md
│   ├── 38 mlir的loc.md
│   ├── 39 Type相关（更深层理解）.md
│   ├── 40 CalibratedQuantizedType.md
│   ├── 43 多于4维的情况.md
│   ├── 44 llvm ArrayRef相关内容.md
│   ├── 45 std get()方法 pair和tuple.md
│   ├── 46 std shared_ptr.md
│   ├── 47 ArrayAttr类相关.md
│   ├── 48 llvm enumerate是什么.md
│   ├── 49 llvm_unreachable.md
│   ├── 50 预处理Pass流程.md
│   ├── 51 编译时链接错误，涉及mlir的库.md
│   ├── 52 运行带选项的Pass.md
│   ├── 53 mlir未修改函数签名带来的问题.md
│   ├── 58 EnumAttr的定义误导性.md
│   ├── 59 属性自己的get方法和Builder的get方法.md
│   ├── 60 pattern的应用层面.md
│   ├── 61 patternEx的使用.md
│   ├── 62 op.getInput.md
│   ├── 63 PatternWriter提供的功能，类比OpBuilder.md
│   ├── 64 在Op中属性常见的coeff是做什么呢？.md
│   ├── 65 PatternSet.md
│   └── 66 Value类中的迭代器使用.md
├── 后端代码/
│   ├── libsophon代码/
│   │   └── 01 库.md
│   ├── sfu/
│   │   ├── 02 浮点数存储.md
│   │   ├── 03 收敛指的是什么？.md
│   │   ├── 04 sfu 典型实现方式.docx
│   │   └── 04 sfu底层算子.md
│   ├── sfu表格/
│   │   ├── SFU_ANALYSIS_COMPLETION_REPORT.md
│   │   ├── SFU_OPERATORS_ANALYSIS.md
│   │   └── SFU_OPERATORS_QUICK_REFERENCE.md
│   ├── 整体项目解析/
│   │   ├── tpuDNN分析/
│   │   │   ├── conv算子的映射关系/
│   │   │   │   └── conv算子内存映射关系.pptx
│   │   │   ├── 01 readme解析.md
│   │   │   ├── 02 testTpudnnConvFloat.md
│   │   │   ├── 03 testTpudnnConvFloat1.md
│   │   │   ├── 04 卷积.md
│   │   │   ├── 05 rt相关.md
│   │   │   ├── 06 include库文件的内容.md
│   │   │   └── 07 bmlib 的流程.md
│   │   ├── 编译过程解析/
│   │   │   ├── 01 rebuild_firmware_cmodel.md
│   │   │   ├── 02 rebuild_all和build_all.md
│   │   │   ├── 03 build_test里调用的build_all分析.md
│   │   │   ├── 04 直接调用rebuild_test的输出.md
│   │   │   ├── 05 build_helper.md
│   │   │   └── 06 调用rebuild_test选项.md
│   │   ├── 02 其中间接调用.md
│   │   ├── 03 外层cmakelist全解析.md
│   │   ├── 04 test.sh分析.md
│   │   ├── 05 builder_helper.sh分析.md
│   │   ├── 06 build_backend.md
│   │   ├── 07 bmlib分析.md
│   │   ├── 08 TPUDNNHost.md
│   │   └── 错误记录.md
│   ├── 测试的两种方式/
│   │   └── readme提到的测试方式.md
│   ├── 01 cmodel_init.md
│   ├── 03 CONTINUOUS_MEM .md
│   ├── 04 thread part 变量.md
│   ├── 05 时间戳.md
│   ├── 06 为矩阵分配地址.md
│   ├── 07 对齐到a倍数的公式.md
│   ├── 08 存放数据按照EU宽度来放.md
│   ├── 09 c_stride.md
│   ├── 11 生成随机数据的方式.md
│   ├── 12 FP32强转u32.md
│   ├── 13 矩阵乘法逻辑.md
│   ├── 14 通用矩阵乘法.md
│   ├── 15 矩阵信息转成在tpu里的格式.md
│   ├── 16 n_和c_详解.md
│   ├── 17 数据迁移部分详解.md
│   ├── 18 CMD_ID_NODE.md
│   ├── 19 初始化指令node名称的含义.md
│   ├── 20 BD逻辑.md
│   ├── 21 BD流程解析.md
│   ├── 22 宏拼接符号.md
│   ├── 23 gen_cmd_t生成指令的结构体.md
│   ├── 24 为什么判断FIFO是否满的代码没实现.md
│   ├── 25 tvgen模式是什么.md
│   ├── 26 矩阵乘法搭配指令.md
│   ├── 27 等待命令执行完在当前cmodel情况下并没有效果.md
│   └── 28 寄存器查找修改步骤.md
├── 实际配置安装环境相关/
│   ├── docker相关/
│   │   ├── 01 docker命令模式.md
│   │   └── 02 docker常用命令.md
│   ├── 01 tpu-mlir和lib.md
│   ├── 02 NVIDIA.md
│   ├── 03 dpkg+.deb.md
│   ├── 04 pip.md
│   └── 编译到算力卡错误内容.md
├── 开学前/
│   ├── AbsAdd接口.md
│   ├── AddOp生成.md
│   ├── Add执行过程.md
│   ├── AI编译全景.md
│   ├── C++与模型.md
│   ├── C++工程结构.md
│   ├── CMake变量.md
│   ├── CMake生成链.md
│   ├── DNNL封装.md
│   ├── Docker命令.md
│   ├── FFN到运行.md
│   ├── FLOPs接口.md
│   ├── LLVM源码构建.md
│   ├── MatMul融合.md
│   ├── MemRef练习.md
│   ├── MLIR基本语法.md
│   ├── ONNX导出坑.md
│   ├── ONNX算子谱.md
│   ├── ONNX节点转.md
│   ├── Op和TD.md
│   ├── Op嵌套树.md
│   ├── Op接口表.md
│   ├── Op测试.md
│   ├── pybind构建.md
│   ├── Python类.md
│   ├── Python路径.md
│   ├── Py类与RMS.md
│   ├── RMS输入图.md
│   ├── runner解析.md
│   ├── SCF控制流.md
│   ├── TD生成目标.md
│   ├── TopToTpu.md
│   ├── Top接口复盘.md
│   ├── TPUAdd接口.md
│   ├── tpuc入口.md
│   ├── TPU源码导读.md
│   ├── TPU量化初识.md
│   ├── 优化Pass.md
│   ├── 内置TD导读.md
│   ├── 单层算子.md
│   ├── 子图形状推理.md
│   ├── 张量类型统计.md
│   ├── 手写MLIR.md
│   ├── 推理参数复习.md
│   ├── 方言加载链.md
│   ├── 方言接口Pass.md
│   ├── 方言是什么.md
│   ├── 构建器比较.md
│   ├── 模型转脚本.md
│   ├── 模式是什么.md
│   ├── 模式重写.md
│   ├── 注册与工厂.md
│   ├── 算子爆炸.md
│   ├── 类型和属性.md
│   └── 自定义Pass.md
├── 拆解TPU-MLIR/
│   ├── A 生成Bmodel.md
│   ├── OpRewriterPatternEx封装.md
│   ├── 关于init Pass是做什么的呢.md
│   └── 阶段性总结 shape-infer功能基本完成.md
├── 服务器登录报错问题/
│   └── 流程记录.md
├── 硬件相关/
│   ├── UR, RtoVR等模块解析/
│   │   ├── 01 UR流程解析.md
│   │   └── 02 RtoVR流程解析.md
│   ├── 修改模块的顺序内容/
│   │   ├── FP16MUL.md
│   │   ├── FP32ADDTREE8TO1相关的微指令.md
│   │   ├── FPADD.md
│   │   ├── FPtoFP.md
│   │   ├── FPtoINT.md
│   │   ├── INTADD.md
│   │   ├── INTtoFP.md
│   │   ├── INTtoINT.md
│   │   └── LOGIC.md
│   ├── 硬件架构信息/
│   │   ├── AIACC 指令集架构附录A v0.1.0.docx
│   │   ├── AIACC 架构介绍 v0.1.2.pptx
│   │   └── RtoVR连接关系 v0.1.1.xlsx
│   ├── ####重要 更新rsqrt的实现记录.md
│   ├── 01 MID SMI SMC LSU.md
│   ├── 02 宏指令和微指令.md
│   ├── 03 SMI-IQ是什么.md
│   ├── 04 aten是什么.md
│   ├── 05 写rsqrt流程.md
│   ├── 06 intadd_iq这种的功能是什么.md
│   ├── 07 SMIIQ微指令长度并不固定.md
│   ├── 08 浮点数加法展示指令流程.md
│   ├── 09 bank是什么？.md
│   ├── AIACC 指令集架构 v0.3.15.docx
│   ├── AIACC 指令集架构 v0.3.4.docx
│   ├── AIACC 指令集架构 v0.4.3.docx
│   ├── AIACC_指令集架构_v0.4.15.docx
│   ├── AIACC指令集架构.md
│   └── 指令集.md
├── 神经网络/
│   ├── 模型量化/
│   │   ├── 01 量化基本.md
│   │   ├── 04 权重量化和激活值量化.md
│   │   └── 05 关于QAT.md
│   ├── 01 MLP vs CNN.md
│   ├── 02 什么叫全连接.md
│   ├── 03 层的概念.md
│   ├── 04 多层卷积的意义.md
│   ├── 05 ResNet的结构组织.md
│   ├── 06 opset的概念.md
│   └── 07 输入归一化操作.md
└── 论文（包括TVM详细内容）/
    ├── APOLLO/
    │   └── 论文.xlsx
    └── TVM/
        ├── 01 tvm.md
        ├── 02 tvm tpu-mlir区别.md
        ├── 03 tvm的核心理念.md
        └── 04 tvm的意义所在.md
```

