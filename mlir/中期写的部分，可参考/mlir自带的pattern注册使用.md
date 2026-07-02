#### 1getCanonicalizationPatterns

首先就是getCanonicalizationPatterns这个方法，在每个Op对应的Pattern里

lib/Dialect/Top/Canonicalize/Add.cpp

```c++
void AddOp::getCanonicalizationPatterns(RewritePatternSet &results,
                                        MLIRContext *context) {
  results.insert<SwapInput, AddToAddConst, AddToScale, AddToRope, AddMerge,
                 AlignInputsDim>(context);
}

```



#### 2、applyPatternsAndFoldGreedily

pattern统一都用这个函数来运行，之前我们在Pass里面看到过

不用我们显式来调用



#### 3、createCanonicalizerPass 

创建一个set实例，然后调用每个Op的getCanonicalizationPatterns方法，然后统一再用apply去调用这些pattern



