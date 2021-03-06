##  用户异常行为检测

本项目采用的是二分类思路，预测哪些是正常用户操作，哪些是伪装用户或者异常操作，数据全是linux bash操作符
目前核心代码和验证已完成，有关部署的一些想法：在ubuntu上装一个bash操作的log实时提取，然后将命令的参数去除，每一百条做为一个基准，有异常强制退出。谁有更稳妥有效简单的部署方案，欢迎提交issue

- ### Masquerading User Data数据集简介:
Matthias Schonlau是加拿大安大略省滑铁卢大学统计学教授他试图通过搜集Linux服务器的Bash操作日志，通过训练识别用户的操作习惯，判断主机上的账户是否被入侵、盗用。然后进行进一步识别账户的异常行为。在他的个人网站上发布了针对Linux操作的训练数据。训练数据中包括50个户的操作日志，每个日志包含15 000 条操作命令，其中前5 000 条都是正常操作，后面的10 000 条日志中随机包含有异常操作。为了便于分析，数据集每100 条操作作为个操作序列，同时进行了标注，每个操作序列只要有1条操作异常就认为这个操作序列异常。
 
- ### 特征提取与数据清洗：
碰巧兜哥的《web安全之机器学习入门》介绍了此数据集，借鉴他的特征提取方法：
   1. KNN_50.py采用了词频统计，将用户使用频率最高的前50条操作和最低的50条操作加以区分，采用KNN训练
   2. NB_all.py采用了词集模型，将所有命令作为特征，根据命令的是否命中，将操作序列向量化，采用NB训练

- ### 模型训练：
   看了兜哥的用SVM来预测XSS准确度而收到了启发，SVM在向量化的二分类问题优势很大
   1. NB_all.py   采用朴素贝叶斯，结果分别与经过交叉验证之后做对比
   2. KNN_50.py   经过SVM NB KNN，最终选择knn，并采用十折交叉验证

- ### 预测的准确率：
注意测试集的不同，准确率也不同,默认是user3测试，换作其他user可能会小幅度波动(KNN推荐：24，42，26，36，9；NB推荐：24，42，26，25，15)

   1. NB_all.py   User9 NB(98%)  十折交叉NB（100%）    User24 NB（58%） 十折交叉（100%）    User42 NB（60%） 十折交叉（94%）
   2. KNN_50.py   User9 KNN(98%) 十折交叉KNN（100%）   User24 KNN（58%）十折交叉（90%）     User42 KNN（60%）十折交叉（64%）



   普遍代表：9  十折交叉的提升典范：24 26   特征提取的影响典范：42，15

- ### 不足之处
 有些操作我都没见过,如有条件拿到更适用的数据，我会再试一次

- ### 一些思考
机器学习项目里，令人最头疼和繁琐的还是数据清洗和数据特征提取。实验证明**数据清洗和特征提取方法的不同对最后准确率的影响远远大于不同机器学习算法带来的影响**！