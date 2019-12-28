# Pytorch

## 乱数生成について

```python
m, k = 2, 3
torch.randn(m, k)
torch.normal(0, 1, size=(m, k))
```

上記の2つの乱数生成は同じ方法で乱数が生成される．
$m \times k$ 行列の成分は平均0，分散1の正規分布から生成される．
