
Emergence of molecular structures from repository-scale self-supervised learning on tandem mass spectra串联质谱存储库规模自监督学习中分子结构的出现

从质谱到Molecular fingerprints分子指纹预测，并实现光谱相似性的比较以及 F 元素的存在性检测

质谱解释的现有方法：质谱相似性、正向注释和逆向注释


原则上，可以从质谱中推导出分子结构而无需依赖注释的光谱库。为实现这一目标，我们设想三个主要的未来步骤。首先，通过从MassIVE和其他资源库（如MetaboLights [55]）挖掘更多数据来扩展GeMS数据集。其次，通过利用LC-MS实验中的所有可用信息，如同位素模式或在MS1光谱中的加合物分布，来提升我们的方法。第三，将大型化合物数据库（如PubChem）纳入我们的自监督方法。受无监督翻译[56]和文本-图像对齐[57]工作启发，我们计划探索将DreaMS嵌入与小分子基础模型[58]的表示空间对齐。