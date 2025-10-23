# WSGMB: weight signed graph neural network for microbial biomarker identification

<img width="1173" height="786" alt="image" src="https://github.com/user-attachments/assets/23a0a69f-386e-4486-af34-560ad56055c6" /> 

| Dataset | #Controls | #Cases |
|---------|---------|---------|
| CRC1   | 40   | 40   |
| CRC2   | 53   | 75   |
| CRC3   | 33   | 33   |
| CD1   | 365   | 600   |
| CD2   | 47   | 42   | 

对照组 (Control) 图统计： {'图数量': 500, '平均节点数': np.float64(81.0), '平均边数': np.float64(2562.496), '平均度数': np.float64(63.27150617283951), '平均边权': np.float64(-0.006435847443062812)} 

病例组 (Case) 图统计： {'图数量': 500, '平均节点数': np.float64(81.0), '平均边数': np.float64(2656.908), '平均度数': np.float64(65.60266666666665), '平均边权': np.float64(-0.005713173296069726)} 

中等稠密图 

# GNNExplanier 
<img width="1029" height="348" alt="image" src="https://github.com/user-attachments/assets/4299ca5a-edc0-4099-b74e-ac9b7d9d29df" /> 

**loss = pred_loss + size_loss + lap_loss + mask_ent_loss + feat_size_loss** 


## pipeline 

**随机采样（30个） -> sparcc求相关性（阈值0.15） -> GNN model** 
### sparcc原理 
微生物数据是组成性数据，物种A的上升或下降，一定会影响另一个物种的下降或上升，所以各物种之间的相关性是伪相关的。 
将组成型数据转化到对数比（log-ratio）空间，再估计真实的相关结构。虽然无法通过相对丰度看到真实的绝对丰度，但是可以用log-ration变换还原它们的协方差结构。 

1️. 它会在样本中多次随机挑选不同的菌作为“参考菌”，计算出所有菌相对这个参考菌的 log 比值。

2. 然后根据这些 log 比值，估计出每个菌的“变动关系”（协方差）。

3️. 再从协方差算出相关系数。  

4️. 为了让结果稳定，还会假设网络是稀疏的（不是所有菌都有关联）

<img width="1003" height="68" alt="image" src="https://github.com/user-attachments/assets/e9c72ff9-dba8-4313-9410-6fcc5d364e02" /> 

---
**LIONESS：Linear Interpolation to Obtain Network Estimates for Single Samples**

传统的网络推断方法（如 Pearson、Spearman、WGCNA、SparCC）通常在**整个样本群体**上计算得到一个平均网络。而 LIONESS 则允许我们为**每个样本**估计一个独特的网络结构，从而捕捉个体间的差异。

---

## 原理

假设我们有 $( N \)$ 个样本。

1. 首先，在所有样本上计算一个总体网络：

   $G_{\text{all}}\$

2. 然后，去掉第 $i$ 个样本，再计算一个“去掉该样本”的网络：

   $G_{-i}$
4. LIONESS 提出一个**线性插值公式**，用以估计第 $i$ 个样本的网络：

   $G_i = N \times (G_{\text{all}} - G_{-i}) + G_{-i}$

   其中：
   - $(G_{\text{all}})$：全部样本构建的群体网络  
   - $( G_{-i} \)$：去除第 $i$ 个样本后的网络  
   - $(N)$：样本总数  
   - $(G_i)$：第 $i$ 个样本的个体网络估计

- 如果去掉第 $i$ 个样本后，某条边的权重变化较大，则说明该样本对这条边的贡献很强；
- LIONESS 就是通过这种“样本扰动”思想，将总体网络的微小变化转化为**单个样本的网络估计**；
- 这种方法不依赖特定的网络构建算法，可以与任何网络推断函数结合（如相关性、互信息、回归等）。



