【玩法说明】
1.政策调整：使用利率滑块调整基准利率；使用货币供应量增长率滑块调整货币供应。
2.时间推进：点击“进入下一季度”按钮推进时间；经济指标将根据当前政策更新。
3.指标监控：观察顶部卡片区的经济指标；绿色表示在目标范围内，红色表示超出范围；查看历史图表分析趋势变化。
4.游戏结束：20个季度后游戏自动结束；显示最终得分和评级；按R键重新开始游戏。

【背景介绍】
中央银行（Central Bank）是国家中居主导地位的金融中心机构，是国家干预和调控国民经济发展的重要工具。负责制定并执行国家货币信用政策，独具货币发行权，实行金融监管。中央银行的宏观经济政策目标为稳定物价、充分就业、经济增长和国际收支平衡等四个方面。

【经济学原理】
1.菲利普斯曲线：萨缪尔森和索罗将原来的菲利普斯曲线发展成为用来表示失业率和通货膨胀率关系的曲线。失业率和通货膨胀率存在反向变动关系关系。
2.货币理论：凯恩斯学派认为货币需求由三部分构成：交易性动机、预防性动机和投机性动机。交易性和预防性动机受收入影响，投机性货币需求是利率的反函数。当利率降低为了实现货币市场的均衡，央行要增加货币供给。
3.通货膨胀：货币主义经济学认为，当市场上货币发行量超过流通中所需要的货币量，就会出现纸币贬值，物价上涨，导致购买力下降，MV=PT。
4.市场预期理论：利率期限结构 完全取决于对未来利率的市场预期。如果预期未来利率上升，则利率期限结构会呈上升趋势；如果预期未来利率下降，则利率期限结构会呈下降趋势。
5.利率：投资是利率的反函数，当央行降低利率，投资增加，消费增加，总产出增长，带动就业。

【中间变量】
股票市场指数和消费者信心指数都是与经济核心目标紧密关联的中间变量。
消费者信心指数是经济预期的“温度计”：当玩家通过降息或增加货币供应刺激经济时，GDP增长和就业改善会提升消费者信心，高信心进一步促进消费与投资，形成良性循环；若通胀过高（超过目标区间），消费者对物价的担忧会降低信心，进而抑制消费，可能拖累GDP增长并推高失业率。
股票市场指数是资金流向的“信号灯”：高消费者信心通常伴随股市交易活跃，游戏中CCI上升会推高股票指数；玩家降低利率可刺激股市（企业融资成本下降），但过度宽松可能引发泡沫，最终因通胀反噬导致股市崩盘。
预警：若股票指数短期暴涨而CCI过度乐观，可能预示泡沫，如2015年中国股市，需警惕后续通胀或崩盘风险。此时玩家应提前收紧货币政策。
政策效果验证：调整利率后，若CCI未如期回升，说明政策传导失效（如就业未改善），需辅助财政刺激或结构性改革。

【代码结构】
1.整块结构
1.1初始化与配置
1.2经济模型类(Economy)
1.3 UI组件类(Button, Slider)
1.4绘图函数(draw_chart, draw_metric_card, draw_policy_panel)
1.5主游戏循环(main)
2.模块功能说明
2.1经济模型类(Economy)：封装所有经济指标和更新逻辑，模拟政策变量(利率、货币供应)对其他指标的影响，维护历史数据用于图表展示.
2.2 UI组件类
2.2.1 Button类：可交互按钮，支持悬停效果
2.2.2 Slider类：滑块控件，用于调整政策变量
2.3绘图函数：draw_chart()：绘制经济指标历史图表，draw_metric_card()：绘制指标卡片，draw_policy_panel()：绘制政策调整面板。
2.4主游戏循环（main）：初始化所有游戏对象，处理用户输入事件，更新经济模型，绘制所有UI元素，管理游戏状态（进行中/结束）。
3.使用的外部库
3.1 Pygame库：创建游戏窗口和界面，处理用户输入事件，绘制图形元素（形状、文本、线条），管理游戏循环和帧率。
3.2 Random库：生成随机扰动，模拟经济不确定性，为经济模型添加随机波动。
3.3 Sys库：提供系统相关功能，优雅地退出游戏。
