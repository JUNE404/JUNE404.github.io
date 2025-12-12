<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>新闻展示 - 最终完美版</title>

<style>
    /* --- 1. 全局设置 --- */
    body {
        margin: 0;
        padding: 0;
        background-color: #f4f4f9;
        height: 100vh;
        font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
        overflow: hidden; 
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
    }

    /* --- 2. 左上角图标 (框不动，人翻转) --- */
    .switch-btn {
        position: absolute;
        top: 25px;
        left: 25px;
        width: 60px;
        height: 60px;
        background: transparent;
        cursor: pointer;
        z-index: 100;
        transition: transform 0.1s;
    }
    .switch-btn:active { transform: scale(0.95); }

    .icon-svg { width: 100%; height: 100%; display: block; }
    .frame-path { fill: none; stroke: #00cc00; stroke-width: 3; stroke-linecap: round; stroke-linejoin: round; }
    .man-path { 
        fill: #00cc00; 
        transform-box: fill-box; 
        transform-origin: center;
        transition: transform 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
    }

    /* --- 3. 舞台布局 (水平居中，小->中->大) --- */
    .stage {
        position: relative;
        width: 1000px;  
        height: 500px;
        display: flex;
        align-items: center; /* 垂直居中 */
        justify-content: center;
    }

    .card {
        position: absolute;
        border-radius: 15px;
        background-color: #ddd; 
        background-size: cover;
        background-position: center;
        cursor: pointer;
        box-shadow: 0 10px 25px rgba(0,0,0,0.2);
        overflow: hidden;
        transition: all 0.6s cubic-bezier(0.25, 0.8, 0.25, 1);
        display: flex;
        align-items: flex-end;
    }

    /* 文字遮罩 */
    .card-mask {
        width: 100%;
        background: linear-gradient(to top, rgba(0,0,0,0.9), transparent);
        padding: 20px;
        color: white;
        transform: translateY(0);
        transition: padding 0.3s;
    }
    .card:hover .card-mask { padding-bottom: 40px; }

    .card-tag {
        font-size: 12px; background: #00cc00; padding: 3px 8px;
        border-radius: 4px; margin-bottom: 5px; display: inline-block; font-weight: bold;
    }
    .card-title { font-size: 18px; font-weight: bold; line-height: 1.3; margin: 0; }

    /* --- 4. 位置槽位定义 (核心布局修改) --- */
    /* 
       我们需要三个位置，中心点都在同一水平线 (top: 50%, translateY(-50%))
       LTR模式：左(小) -> 中(中) -> 右(大)
    */

    /* 位置1: 左侧 (最小) */
    .slot-small {
        width: 200px; height: 280px;
        left: 50px;
        top: 50%; transform: translateY(-50%);
        z-index: 1;
        filter: brightness(0.8);
    }

    /* 位置2: 中间 (中等) */
    .slot-medium {
        width: 280px; height: 380px;
        left: 300px; /* 位于左侧和右侧之间 */
        top: 50%; transform: translateY(-50%);
        z-index: 5;
        filter: brightness(0.9);
    }

    /* 位置3: 右侧 (最大 - 焦点) */
    .slot-large {
        width: 380px; height: 500px;
        left: 620px;
        top: 50%; transform: translateY(-50%);
        z-index: 10;
        filter: brightness(1);
        box-shadow: 0 20px 40px rgba(0,0,0,0.3);
    }
    .slot-large .card-title { font-size: 26px; } /* 大图文字变大 */

    /* --- RTL 模式适配 (翻转布局) --- */
    /* 右(小) <- 中(中) <- 左(大) */
    .stage.rtl-mode .slot-small {
        left: 750px; /* 移到右边 */
    }
    .stage.rtl-mode .slot-medium {
        left: 420px; /* 居中微调 */
    }
    .stage.rtl-mode .slot-large {
        left: 0px; /* 移到左边 */
    }

    /* --- 5. 动画效果 --- */
    /* 擦除 (斜向消失) */
    .erasing::after {
        content: ""; position: absolute; top: -50%; left: -50%; width: 200%; height: 200%;
        background: linear-gradient(135deg, transparent 40%, #f4f4f9 50%, #f4f4f9 100%);
        animation: wipe 0.8s forwards; pointer-events: none;
    }
    @keyframes wipe { 0% { transform: translate(-30%, -30%); } 100% { transform: translate(30%, 30%); } }

    /* 果冻 (弹性出现) */
    .jelly { animation: jelly-pop 0.8s forwards; }
    @keyframes jelly-pop {
        0% { transform: translateY(-50%) scale(0.9); }
        40% { transform: translateY(-50%) scale(1.05); }
        100% { transform: translateY(-50%) scale(1.0); }
    }

    /* --- 6. 二级详情页 (顶部导航栏) --- */
    .overlay {
        position: fixed; top: 0; left: 0; width: 100%; height: 100%;
        background: white; z-index: 200;
        transform: translateY(100%); transition: transform 0.3s ease-in-out;
        display: flex; flex-direction: column;
    }
    .overlay.active { transform: translateY(0); }

    .nav-bar {
        height: 60px; background: white; border-bottom: 1px solid #eee;
        box-shadow: 0 2px 10px rgba(0,0,0,0.05);
        display: flex; align-items: center; padding: 0 20px;
        position: sticky; top: 0; z-index: 10;
    }
    .back-btn {
        background: #f0f0f0; border: none; font-size: 16px; font-weight: 600; color: #333;
        cursor: pointer; display: flex; align-items: center; padding: 8px 16px;
        border-radius: 20px; transition: background 0.2s;
    }
    .back-btn:hover { background: #e0e0e0; }

    .content-scroll { flex: 1; overflow-y: auto; padding: 40px 20px; }
    .article-container { max-width: 800px; margin: 0 auto; padding-bottom: 50px; }
    .article-title { font-size: 32px; color: #111; margin-bottom: 15px; }
    .article-meta { color: #888; font-size: 14px; margin-bottom: 25px; border-left: 4px solid #00cc00; padding-left: 10px; }
    .article-img { width: 100%; height: auto; border-radius: 12px; margin-bottom: 30px; object-fit: cover; max-height: 400px; }
    .article-body { font-size: 18px; line-height: 1.8; color: #333; white-space: pre-wrap; font-family: 'Georgia', serif; }
    
</style>
</head>
<body>

    <!-- 左上角图标 -->
    <div class="switch-btn" onclick="toggleDirection()" title="切换布局方向">
        <svg class="icon-svg" viewBox="0 0 60 60">
            <!-- 框不动 -->
            <rect class="frame-path" x="5" y="5" width="50" height="50" rx="8" ry="8" />
            <!-- 人翻转 -->
            <g class="man-path" id="manGroup">
                <circle cx="30" cy="20" r="5" />
                <path d="M 25 30 L 35 30 L 38 40 L 45 40 M 35 30 L 35 45 L 42 52 M 35 45 L 28 52" 
                      fill="none" stroke="#00cc00" stroke-width="4" stroke-linecap="round" stroke-linejoin="round"/>
            </g>
        </svg>
    </div>

    <!-- 舞台 -->
    <div class="stage" id="stage">
        <!-- 卡片0 -->
        <div class="card" id="card0" onclick="openDetail(0)">
            <div class="card-mask">
                <span class="card-tag">TECH</span>
                <div class="card-title">Oracle Slumps</div>
            </div>
        </div>
        <!-- 卡片1 -->
        <div class="card" id="card1" onclick="openDetail(1)">
            <div class="card-mask">
                <span class="card-tag">BUSINESS</span>
                <div class="card-title">Record Billionaires</div>
            </div>
        </div>
        <!-- 卡片2 -->
        <div class="card" id="card2" onclick="openDetail(2)">
            <div class="card-mask">
                <span class="card-tag">HEALTH</span>
                <div class="card-title">Perfect Sleep Time</div>
            </div>
        </div>
    </div>

    <!-- 详情页 -->
    <div class="overlay" id="overlay">
        <div class="nav-bar">
            <button class="back-btn" onclick="closeDetail()">
                <span style="margin-right:5px;">←</span> 返回首页
            </button>
            <div style="flex:1;"></div>
        </div>
        <div class="content-scroll">
            <div class="article-container">
                <h1 class="article-title" id="d-title"></h1>
                <div class="article-meta" id="d-meta"></div>
                <img src="" class="article-img" id="d-img">
                <div class="article-body" id="d-content"></div>
            </div>
        </div>
    </div>

<script>
    // --- 1. 完整数据 (包含所有文本) ---
    // 为了显示效果，我使用了网络图片链接。
    // 如果您想用本地图片，请将 img 属性改为 "news1.jpg" 等，并将文件放在同一目录下。
    const NEWS_DATA = [
        {
            id: 0,
            // Oracle 图片
            img: "https://images.unsplash.com/photo-1642132652859-3ef5a9290aa8?q=80&w=1000&auto=format&fit=crop", 
            title: "Oracle slumps as gloomy forecasts, soaring spending fan AI bubble worries",
            meta: "By Aditya Soni and Kanchana Chakravarty | Dec 12, 2025 | 7:48 AM GMT+8",
            content: `甲骨文因悲观预测和飞涨的消费而下跌，粉丝 AI 泡沫担忧

• Summary 总结
• Companies 公司
• Oracle shares drop after dour forecast, higher capex (甲骨文股价因阴郁预测及资本支出上升而下跌)
• AI-related stocks including Nvidia fall (包括英伟达在内的 AI 相关股票下跌)
• At least 13 brokerages slash PT on stock following results (至少有 13 家券商在业绩公布后对股票进行 PT 调整)

Dec 11 (Reuters) - Oracle (ORCL.N), opens new tab shares sank 13% on Thursday, sparking a tech selloff as the company's massive spending and weak forecasts fanned doubts over how quickly the big bets on AI will pay off.
12 月 11 日（路透社）——甲骨文（ORCL.N）股价周四下跌 13%，引发科技股抛售，公司巨额支出和疲弱的预测加剧了人们对人工智能大注回报速度的质疑。

The disappointing predictions from the crucial OpenAI cloud-computing partner show the uneven returns from the nascent technology that many business leaders believe is the future but has so far yielded limited productivity gains.
来自关键 OpenAI 云计算合作伙伴的令人失望的预测显示，许多企业领导者认为是未来但迄今生产率提升有限的初兴技术回报不均。

Oracle, long a smaller cloud player, has vaulted into the artificial intelligence infrastructure race this year thanks to a $300 billion OpenAI deal. But the pact has also tethered the company's fortunes to the ChatGPT maker, with shares sliding in recent weeks on concern Google is pulling ahead of OpenAI.
甲骨文长期以来一直是较小的云平台，今年凭借一笔价值 3000 亿美元的 OpenAI 交易，跃入人工智能基础设施竞赛。但这项协议也将公司与 ChatGPT 制造商的命运紧密相连，近几周因担心谷歌领先 OpenAI 而股价下滑。

Investors have also dumped Oracle bonds on worries about its debt-fueled AI build-out, while piling into credit-default swaps (CDS) that offer bondholders a hedge against default. CDS for the company, which has around $100 billion in debt, rose by nearly 12 basis points on Thursday to at least five-year highs.
投资者也因担忧甲骨文债券的债务驱动 AI 建设而抛售，同时投入信用违约掉期（CDS），为债券持有人提供对冲违约风险。该公司的 CDS 指数周四上涨近 12 个基点， 达到至少五年来的最高点。

Big Tech's rising appetite for debt, including bond sales of more than $30 billion by Meta and $15 billion from Amazon.com, has marked a departure for the companies that long depended on strong cash flows to fund spending on new initiatives.
大型科技公司对债务的增长，包括 Meta 发行的债券超过 300 亿美元和 Amazon.com 发行的 150 亿美元 ，标志着长期依赖强现金流资助新项目支出的公司走向转变。

Tech executives have said the outlays are necessary for a technology that will transform work and make businesses more efficient, arguing the bigger risk is underinvesting, not overspending.
科技高管表示，这些投入对于一项能够改变工作、提升企业效率的技术来说是必要的，他们认为更大的风险是投资不足，而非超支。

Oracle, which has burned around $10 billion in cash in the first half of its fiscal year due to the AI investments, was set to lose more than $90 billion in market value if the losses hold.
甲骨文公司在财年上半年因人工智能投资损失了约 100 亿美元现金，如果损失持续，预计市值将损失超过 900 亿美元。

Ellison, who is the world's second-richest man mostly due to his roughly 40% stake in Oracle, has a net worth $276 billion, according to the Forbes real-time billionaire list.
埃里森是全球第二富有的人，主要因为他在甲骨文公司约40%的股份，根据《福布斯》实时亿万富翁榜单，他的净资产达到2760亿美元。

TECH SHARES FALL 科技股下跌
Other AI stocks, including chip giant Nvidia (NVDA.O), Advanced Micro Devices (AMD.O), Micron (MU.O), Broadcom (AVGO.O) and Arm Holdings, also fell between 3.1% and 4.2%. That pushed down the tech-heavy Nasdaq (.IXIC) to a one-week low.
其他人工智能股票，包括芯片巨头英伟达（NVDA）。O），先进微型器件（AMD.O）、Micron（MU.O）、博通（AVGO）。O）和 Arm Holdings 也下跌在 3.1%至 4.2%之间。这压低了以科技股为主的纳斯达克股价（.IXIC）降至一周低点。

Some of those fears have stemmed from circular deals involving OpenAI, which has a valuation of about $500 billion but still loses money and has not detailed how it plans to fund AI spending commitments of more than $1 trillion by 2030.
其中一些担忧源自涉及 OpenAI 的循环交易，OpenAI 估值约 5000 亿美元，但仍亏损，且未详细说明到 2030 年如何资助超过 1 万亿美元的 AI 支出承诺。

At least 13 brokerages slashed their price targets on Oracle's stock. But some brushed off concerns, saying the spending was necessary.
至少有13家券商下调了甲骨文股票的目标价格。但也有人不以为意，称这些支出是必要的。`
        },
        {
            id: 1,
            // Bank/Billionaires 图片
            img: "https://images.unsplash.com/photo-1559526324-4b87b5e36e44?q=80&w=1000&auto=format&fit=crop",
            title: "Planet Earth has never had so many billionaires",
            meta: "By Ana Nicolaci da Costa | Dec 5, 2025",
            content: `地球从未有过如此多亿万富翁

London — The number of billionaires worldwide – and their combined wealth – reached record highs this year, buoyed in particular by gains in tech stocks, Swiss bank UBS has found.
伦敦 — 瑞士银行瑞银发现，全球亿万富翁数量及其总财富今年创下历史新高，尤其得益于科技股的上涨。

The world had 2,919 billionaires on April 4, when UBS completed that part of its research, the largest number since the bank started collecting the data in 1995.
截至4月4日，全球亿万富翁为2,919人，瑞银完成了这部分研究，这是自1995年银行开始收集数据以来的最高数字。

“With tech shares and financial asset prices generally rising, despite volatility, billionaires’ total wealth reached a new high,” UBS wrote in its annual report on billionaires published Thursday.
瑞银在其周四发布的亿万富翁年度报告中写道：“尽管存在波动，科技股和金融资产价格普遍上涨，亿万富翁的总财富达到了新的高点。”

The MSCI World Index – which tracks more than 1,300 large and mid-sized public companies in developed markets – rose around 7% in the year to early April.
MSCI 世界指数 ——追踪发达市场 1300 多家大型和中型上市公司——在截至四月初的一年内上涨了约 7%。

Billionaires’ combined wealth reached $15.8 trillion, an increase of 13% over 12 months, lifted by existing tech billionaires’ appreciating assets and by new billionaires across a range of sectors, the bank said. The number of billionaires jumped by 237 in the 12 months to April 4.
银行表示，亿万富翁的总财富达到15.8万亿美元，过去12个月增长13%，这得益于现有科技亿万富翁资产的增值以及来自多个行业的新亿万富翁。截至4月4日的12个月内，亿万富翁人数增加了237人。

Tech billionaires’ assets grew by almost a quarter to $3 trillion, bolstered by the increasing value of their shares in artificial intelligence-linked companies such as Meta (META), Oracle (ORCL) and Nvidia (NVDA).
科技亿万富翁的资产增长近四分之一，达到 3 万亿美元，这得益于他们在 Meta（META）、甲骨文（ORCL）和英伟达（NVDA）等与人工智能相关公司的股价不断上涨。

The six US tech billionaires whose wealth increased the most became $171 billion richer between them, UBS said, meaning that each became $28.5 billion richer on average.
瑞银表示，财富增长最多的六位美国科技亿万富翁加起来增加了1710亿美元，平均每人都增加了285亿美元。

“This surge is closely tied to their companies’ AI capabilities, from foundational chipmaking to AI-powered cloud infrastructure,” the bank wrote.
“这股激增与其公司的人工智能能力密切相关，从基础芯片制造到基于人工智能的云基础设施，”银行写道。

The total number of US billionaires rose to 924, accounting for almost a third of the global billionaire population. China now has 470 billionaires, second only to the United States.
美国亿万富翁总数上升到924人，几乎占全球亿万富翁总数的三分之一。中国目前拥有470名亿万富翁，仅次于美国。

UBS surveyed its billionaire clients between July and September 2025 and they identified tariffs, geopolitics and policy uncertainty as the top risks of 2026.
瑞银于2025年7月至9月期间对其亿万富翁客户进行了调查，他们将关税、地缘政治和政策不确定性列为2026年的主要风险。

“What billionaires worry about most depends on where they live,” the bank wrote. For instance, 75% of billionaires in Asia-Pacific were concerned about tariffs, while 70% of American billionaires worried about higher inflation or a major geopolitical conflict.
“亿万富翁最担心的事取决于他们居住在哪里，”该银行写道。例如，亚太地区75%的亿万富翁担心关税，而70%的美国亿万富翁则担心通胀上升或重大地缘政治冲突。`
        },
        {
            id: 2,
            // Sleep/Nature 图片
            img: "https://images.unsplash.com/photo-1520206183501-b80df610434f?q=80&w=1000&auto=format&fit=crop",
            title: "You might not need 8 hours of rest. Here’s how to find your perfect sleep time",
            meta: "By Gina Park | Updated Dec 9, 2025",
            content: `你可能不需要8小时的休息。以下是找到理想睡眠时间的方法

Want to nap like a professional athlete? Here’s how to do it anytime, anywhere
想像职业运动员一样小憩吗？以下是随时随地都能做到的方法

With so much to do heading into the busy holiday season, is anyone getting enough sleep?
在忙碌的假日季临近之际，有没有人睡得够？

Most sleep experts advise that adults get seven to nine hours of sleep per night for good health and emotional well-being (although that changes as you get older). And studies warn that sleeping for less than seven hours a day can increase the risk of obesity, high blood pressure, heart disease and other issues that come with sleep deprivation.
大多数睡眠专家建议成年人每晚睡七到九小时 ，以促进健康和情绪健康（尽管随着年龄增长，这一时间有所变化）。研究警告说，每天睡眠少于七小时会增加肥胖、高血压、心脏病及其他睡眠不足问题的风险。

CNN has reported on those risks, too, including how getting five hours or less of sleep can increase the likelihood of developing chronic disease.
CNN 也报道了这些风险，包括睡眠不足五小时或更少会增加患慢性病的可能性。

Dr. Tony Cunningham, clinical psychologist and director of the Center for Sleep and Cognition in Boston, said it isn’t as simple as getting the “right” number of hours of sleep. In a conversation with CNN, he explained his thinking.
波士顿睡眠与认知中心临床心理学家兼主任托尼·坎宁安博士表示，这并不像获得“正确”睡眠小时那么简单。在接受 CNN 采访时，他解释了自己的想法。

Sleep quality is just as important as sleep time
睡眠质量和睡眠时间同样重要

A lot of people tend to focus on how many hours of sleep they’re getting but neglect the quality of their sleep, which can be even more important than sleep time.
很多人关注的是自己睡了多少小时，却忽视了睡眠质量，而睡眠质量甚至比睡眠时间更重要。

“There’s two different things going on in our bodies that dictate both the type of sleep that we’re getting and the quality of sleep that we’re getting, and that is our sleep pressure and circadian rhythms,” said Cunningham, who is also an assistant professor of psychology at Harvard Medical School.
“我们身体中有两种不同的因素决定我们所获得的睡眠类型和睡眠质量，那就是我们的睡眠压力和昼夜节律，”坎宁安说，他同时也是哈佛医学院心理学助理教授。

Sleep pressure or sleep drive builds up the longer that you’re awake and decreases while you’re asleep. It’s what causes you to start feeling tired after being awake for an extended period.
睡眠压力或睡眠驱动力随着清醒时间增加而增加，睡眠时减弱。这也是导致你长时间清醒后开始感到疲倦的原因。

“It’s just like eating,” Cunningham said. “The longer it’s been since you’ve eaten, the hungrier you get.”
“这就像吃饭一样，”坎宁安说。“你越久没吃东西，越饿。”

To get a good night’s worth of sleep, you want to get into bed when you’ve built up a lot of sleep pressure.
要保证一夜好眠，最好在积累大量睡眠压力后上床。

Your circadian rhythm is your body’s internal clock. Don’t let the name fool you though. Although external factors, such as light, can affect your circadian rhythm, the pattern that your body follows is guided by your brain.
你的昼夜节律是你身体的内部时钟。不过别被名字骗了。虽然外部因素如光线会影响你的昼夜节律，但身体遵循的规律由大脑引导 。

For higher sleep quality, sleep pressure and your circadian rhythm should be working together. That means that any abrupt changes or an irregular sleep schedule can affect your ability to sleep and lower sleep quality.
为了提高睡眠质量，睡眠压力和昼夜节律应协同工作。这意味着任何突然的变化或不规律的睡眠时间都会影响你的睡眠能力，降低睡眠质量。

How many hours of sleep do you need?
你需要睡多少小时？

There is nothing wrong about advising people to get an average of seven to nine hours of sleep, but it’s important to remember that this range is the average.
建议人们平均睡眠七到九小时没有错，但重要的是要记住，这个范围只是平均值。

“That does not mean that every single person on the planet needs eight hours of sleep,” Cunningham said. “There are some people that really, truly only need five or six hours — like their biology and physiology only would allow them at optimal functioning to get five to six hours of sleep.”
“这并不意味着地球上的每个人都需要八小时的睡眠，”坎宁安说。“有些人真的只需要五六个小时——他们的生理和生理结构只能让他们在最佳状态下睡五到六个小时。”

This goes both ways though. “There are also people out there that need nine, 10, 11 hours of sleep per night,” he added.
不过这也是双向的。“也有些人每晚需要九、十、十一小时的睡眠，”他补充道。

To figure out how much sleep you need, you can do these two things.
要计算你需要多少睡眠，可以做以下两件事。

“You’re going to keep a consistent bedtime. I want it to be a bedtime that you are pretty confident you’ll be able to fall asleep within no more than 20 to 30 minutes,” Cunningham said.
“你得保持固定的睡觉时间。我希望你能在20到30分钟内入睡，“坎宁安说。

Your bedtime should be when you’re feeling sleepy, not just tired. If you get into bed and can’t fall asleep within that 20-to-30-minute range, then it’s likely that you haven’t built up enough sleep pressure.
你的睡觉时间应该是在你感到困倦的时候，而不是单纯疲惫的时候。如果你上床后20到30分钟内无法入睡，很可能是睡眠压力还不够。

If that’s the case, it’s better to engage in some low-arousing activities, such as taking a bath or meditating with the lights dimmed, until you start to feel sleepy.
如果是这样，最好进行一些低刺激的活动，比如泡澡或调暗灯光冥想，直到你开始感到困倦。

“Then you need to find a period of time in your life where you can sleep until you wake up naturally with no alarm,” Cunningham said.
“那你需要找到一段你可以自然醒来、没有闹钟的时段，”坎宁安说。

“Go around your room, hide your clocks, block out the curtains, maybe do noise machines, have an eye mask on. Everything you can do so that you can be in your room, you have no sense of what time it is, and then you go to sleep, and you sleep until you wake up,” he added.
“在房间里走动，藏好钟表， 挡住窗帘 ，也许用噪音机，戴上眼罩。你能做的一切都是让你能待在房间里，完全不知道现在几点了，然后你就睡着了，一直睡到醒来，“他补充道。

After those first few days of catching up, you’ll know you’ve found your sleep time “when you wake up for three or four days in a row at approximately the same time with no external cues, no light cues and no alarm,” he said.
经过最初几天的补课后，你会知道自己找到了睡眠时间，“当你连续三四天几乎在同一时间醒来，没有外部信号、没有光线、没有闹钟时”，他说。`
        }
    ];

    // --- 2. 状态与配置 ---
    const cards = [
        document.getElementById('card0'),
        document.getElementById('card1'),
        document.getElementById('card2')
    ];
    
    // 初始化背景
    NEWS_DATA.forEach((item, index) => {
        cards[index].style.backgroundImage = `url('${item.img}')`;
    });

    let isRTL = false; // 默认 LTR: 左->右
    let autoPlayTimer = null;
    let isAnimating = false;
    // 顺序数组: 0在Small, 1在Medium, 2在Large
    let order = [0, 1, 2];

    // --- 3. 渲染视图 ---
    function updateLayout(animate = false) {
        cards.forEach(c => {
            c.classList.remove('slot-small', 'slot-medium', 'slot-large', 'erasing', 'jelly');
            void c.offsetWidth; // 强制重绘
        });

        // 绑定位置类
        cards[order[0]].classList.add('slot-small');
        cards[order[1]].classList.add('slot-medium');
        cards[order[2]].classList.add('slot-large');

        // 只有在自动轮播触发时，新的大图才会有果冻效果
        if (animate) {
            cards[order[2]].classList.add('jelly');
        }
    }

    // --- 4. 切换方向 ---
    function toggleDirection() {
        isRTL = !isRTL;
        const stage = document.getElementById('stage');
        const manGroup = document.getElementById('manGroup');
        
        if (isRTL) {
            stage.classList.add('rtl-mode');
            manGroup.style.transform = "scaleX(-1)"; // 小人翻转
        } else {
            stage.classList.remove('rtl-mode');
            manGroup.style.transform = "scaleX(1)";
        }
        resetTimer();
    }

    // --- 5. 自动轮播逻辑 ---
    function nextSlide() {
        if (isAnimating) return;
        isAnimating = true;

        // 逻辑: 我们永远擦除当前的“最大图”
        // 在LTR模式下，最大图是数组最后一个 (index 2)
        // 在RTL模式下，最大图视觉上在左边，但逻辑上我们还是操作数组的末尾作为“焦点”
        
        const focusCardIndex = order[2]; 
        const focusCard = cards[focusCardIndex];
        
        focusCard.classList.add('erasing');

        setTimeout(() => {
            // 数组位移: [0, 1, 2] -> [2, 0, 1] 
            // 把最后一个移到最前面，这样原来的Medium(1)变成Large(2)
            const last = order.pop();
            order.unshift(last);

            updateLayout(true);
            isAnimating = false;
        }, 800);
    }

    function startTimer() { autoPlayTimer = setInterval(nextSlide, 5800); }
    function resetTimer() { clearInterval(autoPlayTimer); startTimer(); }

    // --- 6. 详情页交互 ---
    const overlay = document.getElementById('overlay');
    const dTitle = document.getElementById('d-title');
    const dMeta = document.getElementById('d-meta');
    const dImg = document.getElementById('d-img');
    const dContent = document.getElementById('d-content');

    function openDetail(index) {
        clearInterval(autoPlayTimer);
        const data = NEWS_DATA[index];
        dTitle.innerText = data.title;
        dMeta.innerText = data.meta;
        dImg.src = data.img;
        dContent.innerText = data.content;
        overlay.classList.add('active');
    }

    function closeDetail() {
        overlay.classList.remove('active');
        resetTimer();
    }

    // --- 初始化 ---
    updateLayout();
    startTimer();

</script>
</body>
</html>
