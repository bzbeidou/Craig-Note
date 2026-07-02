#### 1 实际上是复习

```
Type
├── IntegerType         // i1, i8, i16, i32, ...
├── FloatType           // f16, f32, f64, ...
│   ├── F16Type
│   ├── F32Type
│   └── F64Type
├── TensorType
│   ├── UnrankedTensorType
│   └── RankedTensorType
│        └── (elementType = ElementType)
├── ...
└── QuantType (quant dialect)
    ├── CalibratedQuantizedType
    ├── UniformQuantizedType
    └── UniformQuantizedPerAxisType
```

#### 2 具体例子

```
tensor<1x3x224x224xf32>
```

```
RankedTensorType (shape = [1,3,224,224])
 └── elementType = F32Type
```

```
tensor<1x3x224x224x!quant.calibrated<f32, [-1.0:1.0]>>
```

```c++
RankedTensorType (shape = [1,3,224,224])
 └── elementType = CalibratedQuantizedType
         ├── expressedType = F32Type   // 基础类型（真实值类型）
         ├── min = -1.0
         └── max = 1.0
```

#### 3 CalibratedQuantizedType

```c++
class CalibratedQuantizedType : public QuantizedType {
public:
  static CalibratedQuantizedType get(Type expressedType,
                                     double min, double max);
  double getMin() const;
  double getMax() const;
};
```

```c++
auto f32Type = builder.getF32Type();
auto qType = quant::CalibratedQuantizedType::get(f32Type, -1.0, 1.0);
auto tensorType = RankedTensorType::get({1, 3, 224, 224}, qType);
```

```c++
tensor<1x3x224x224x!quant.calibrated<f32, [-1.0:1.0]>>
```

