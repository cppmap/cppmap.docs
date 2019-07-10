description: IEEE 754 の演算のルール

# IEEE 754 演算のルール

浮動小数点数の丸めモードが `FE_TONEAREST` の場合の結果です。

## 四則演算

![](images/ieee754-plus.png)

![](images/ieee754-minus.png)

![](images/ieee754-mul.png)

![](images/ieee754-div.png)

## 比較演算

![](images/ieee754-eq.png)

![](images/ieee754-ne.png)

![](images/ieee754-lt.png)

![](images/ieee754-lte.png)

![](images/ieee754-gt.png)

![](images/ieee754-gte.png)

## 三方比較演算
結果は `std::partial_ordering` 型

![](images/ieee754-threeway.png)

