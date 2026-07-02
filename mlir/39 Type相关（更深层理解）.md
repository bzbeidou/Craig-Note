#### 

#### 1类型系统

```
Type
├── TensorType
│   ├── RankedTensorType
│   └── UnrankedTensorType
├── IntegerType
├── FloatType
├── IndexType
└── QuantizedType (比如 CalibratedQuantizedType)

```

- `RankedTensorType`，这个就是指的确定张量形状的tensor,比如上面`tensor<1x3x224x224xf32>`，`UnRanked`这种就是不确定类型的



#### 2TensorType和ElementType的使用举例

```cpp
auto f32 = builder.getF32Type();  // ElementType
auto tensorType = RankedTensorType::get({1, 3, 224, 224}, f32);
```

```mlir
tensor<1x3x224x224xf32>
```

```c++
auto i8 = builder.getIntegerType(8);
auto tensorType = RankedTensorType::get({1, 3, 224, 224}, i8);
```

```mlir
tensor<1x3x224x224xi8>
```

#### 3带校准信息的量化张量 (CalibratedQuantizedType)

```c++
auto calibratedType = quant::CalibratedQuantizedType::get(
    builder.getF32Type(),  // 原始存储类型
    -1.0, 1.0              // min/max
);
auto tensorType = RankedTensorType::get({1, 3, 224, 224}, calibratedType);
```

```mlir
tensor<1x3x224x224x!quant.calibrated<f32, -1.0:1.0>>
```
