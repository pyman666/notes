
- [设置](#设置)
- [relplot](#relplot)
- [scatterplot](#scatterplot)
- [lineplot](#lineplot)
- [catplot](#catplot)
- [stripplot](#stripplot)
- [swarmplot](#swarmplot)
- [violinplot](#violinplot)
- [barplot](#barplot)
- [countplot](#countplot)
- [pointplot](#pointplot)
- [distplot](#distplot)
- [kdeplot](#kdeplot)
- [rugplot](#rugplot)
- [joinplot](#joinplot)
- [pairplot](#pairplot)
- [regplot](#regplot)
- [lmplot](#lmplot)
- [residplot](#residplot)
- [heatmap](#heatmap)
- [clustermap](#clustermap)

## 设置

## relplot

关系图，默认scatterplot

- kind - scatter(默认)，line

## scatterplot

散点图

## lineplot

线形图

- ci - xy一对多默认均值+95%置信区间；None禁用；sd标准差代替置信区间

- estimator - xy一对多聚合；None禁用

- dashes - 第二形状默认短横，False禁用

- markers - 每个数据点标记？True启用

- units - 多组采样的分类依据（hue的hue）

# 分类

## catplot

类别图，默认stripplot

- dodge - hue会在一个x用颜色区分几个类(dodge)，False禁用

- edgecolor - float边框颜色？

- order - 排种类？

- kind - strip(默认)、swarm、box、violin、boxen、point、bar、count

## stripplot

小数据

- jitter - 抖动大小，False禁用

## swarmplot

减少重叠，小数据

# boxplot

箱型图，大数据

# boxenplot

加强箱型图，适合更大的数据

## violinplot

小提琴图，大数据

- bw：{'scott'，'silverman'，float} 计算内核带宽时使用的引用规则的名称或比例因子。 实际内核大小将通过将比例因子乘以每个bin中数据的标准差来确定

- cut - float 以带宽大小为单位的距离，用于将密度扩展到超过极端数据点。 设置为0可将小提琴范围限制在观测数据范围内 （即，与ggplot中的trim=true具有相同的效果）。

- scale - 用于缩放每个小提琴宽度。area每个小提琴都会有相同的区域；count小提琴的宽度将按照该箱中的观察次数进行缩放；width每个小提琴将具有相同的宽度

- gridsize - int 计算核密度估计的离散网格中的点数

- inner - 小提琴内部的数据点。box画一个微型箱图。 quartile绘制分布的四分位数。point、stick显示每个基础数据点。None将绘制未经修饰的小提琴

- split - True多级别将为各级别绘制一半小提琴

- saturation：float 饱和度

- dodge:bool 使用色调嵌套时，是否应沿分类轴移动元素

## barplot

柱状图

## countplot

直方图

## pointplot

点图

# 分布

## distplot

直方图+核密度估计(kde)

- bin - 柱个数

- hist - False禁用柱子

- ked - 核密度估计，False禁用

- rug - bool地毯图

- fit - 分布的类型

## kdeplot

密度图

- shade - 阴影

- bw - bandwidth 带宽，类似于bin的宽度

- n\_levels？

## rugplot

地毯图

## joinplot

双变量分布图

- kind - scatter默认；hex六边形；reg带回归；kde类似密度图；resid

- size : 默认 6，尺度

- radio : 中心图与侧边图的比例，越大、中心图占比越大

- space : 中心图与侧边图的间隔大小。参数类型：numeric

- edgecolor : 点的边界颜色，默认无色

- {joint, marginal, annot}\_kws

- marginal\_kws : 侧边图的信息。例如：dict(bins=15, rug=True)

- annot\_kws : 注释的信息。例如：dict(stat="r")

## pairplot

多变量两两分析图

# 线性

## regplot

## lmplot

- order - 模型阶数

- robust - 鲁棒性，True忽略异常值

- logistic - True逻辑回归

- lowess - 低水平平滑器？

- x\_jiter - x抖动

- x\_estimator

- ci

- scatter\_kws={"s": 80}

- spect？

## residplot

残差图

# 其他

## heatmap

热力图

- annot - 是否加数值

- cbar : 是否显示右侧色条

- vmin

- vmax

- center - 色彩居中值

- robust - 如果是 True，并且vmin或vmax为空，则使用稳健分位数而不是极值来计算色彩映射范围

- square - 是否显示为方形

- fmt : 指定所显示具体数值的格式

## clustermap

热力图+相关系数聚类
