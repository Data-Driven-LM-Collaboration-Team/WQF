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

**loss = pred_loss + edge_size_loss + node_size_loss + mask_ent_loss** 



