<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>新闻展示 - 深度优化版</title>

<style>
    /* --- 全局与排版 --- */
    body {
        margin: 0;
        padding: 0;
        background-color: #f4f4f9; /* 柔和的灰白背景 */
        height: 100vh;
        font-family: 'Segoe UI', Roboto, 'Helvetica Neue', sans-serif;
        overflow: hidden; /*以此保证主页不出现滚动条*/
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
    }

    /* --- 左上角切换按钮 --- */
    .switch-btn {
        position: absolute;
        top: 20px;
        left: 20px;
        width: 60px;
        height: 60px;
        background: white;
        border: 3px solid #00cc00;
        border-radius: 12px;
        cursor: pointer;
        display: flex;
        align-items: center;
        justify-content: center;
        z-index: 100;
        box-shadow: 0 4px 12px rgba(0,0,0,0.15);
        transition: transform 0.2s;
    }

    .switch-btn:active {
        transform: scale(0.95);
    }

    .icon-man {
        width: 40px;
        height: 40px;
        transition: transform 0.5s cubic-bezier(0.34, 1.56, 0.64, 1);
    }
    
    .icon-man svg {
        width: 100%;
        height: 100%;
        fill: #00cc00; 
    }

    /* --- 舞台区域 --- */
    .stage {
        position: relative;
        width: 100%;
        max-width: 1000px;
        height: 500px;
        display: flex;
        align-items: center;
        justify-content: center;
        perspective: 1000px; /* 增加一点透视感 */
    }

    /* --- 新闻封面卡片 --- */
    .card {
        position: absolute;
        width: 300px; 
        height: 420px;
        border-radius: 20px;
        box-shadow: 0 15px 35px rgba(0,0,0,0.3);
        cursor: pointer;
        transition: transform 0.6s ease, left 0.6s ease, filter 0.6s;
        overflow: hidden; 
        background-size: cover;
        background-position: center;
        display: flex;
        flex-direction: column;
        justify-content: flex-end; /* 文字沉底 */
    }

    /* 卡片上的文字遮罩 */
    .card-content {
        background: linear-gradient(to top, rgba(0,0,0,0.9), transparent);
        padding: 20px;
        color: white;
        transform: translateY(0);
        transition: transform 0.3s;
    }

    .card:hover .card-content {
        padding-bottom: 30px; /* 悬停稍微上浮 */
    }

    .card-title {
        font-size: 22px;
        font-weight: bold;
        margin-bottom: 5px;
        text-shadow: 0 2px 4px rgba(0,0,0,0.5);
    }

    .card-tag {
        font-size: 12px;
        text-transform: uppercase;
        letter-spacing: 1px;
        background: #00cc00;
        display: inline-block;
        padding: 2px 8px;
        border-radius: 4px;
        margin-bottom: 8px;
    }

    /* --- 位置槽位 (Slot) --- */
    /* 定义三个位置: 左、中、右 */
    
    /* 槽位1: 左侧 */
    .slot-left {
        left: 15%; 
        transform: scale(0.7) translateX(-50%);
        z-index: 1;
        filter: brightness(0.6) blur(1px);
    }

    /* 槽位2: 中间 (默认聚焦) */
    .slot-center {
        left: 50%;
        transform: scale(1.0) translateX(-50%);
        z-index: 10;
        filter: brightness(1);
        box-shadow: 0 20px 50px rgba(0,0,0,0.4);
    }

    /* 槽位3: 右侧 */
    .slot-right {
        left: 85%;
        transform: scale(0.7) translateX(-50%);
        z-index: 1;
        filter: brightness(0.6) blur(1px);
    }

    /* --- LTR 模式适配 --- */
    /* 当 Body 有 .mode-ltr 时，我们反转阅读逻辑，但视觉位置保持左中右 */
    /* 此处我们通过JS改变 currentOrder 数组的流转方向来实现视觉变化，CSS保持位置定义不变 */


    /* --- 动画特效 --- */

    /* 1. 擦除动画 (Chalk Erase) */
    .erasing::after {
        content: "";
        position: absolute;
        top: -50%; left: -50%;
        width: 200%; height: 200%;
        /* 这里的颜色需要匹配背景色 */
        background: linear-gradient(135deg, transparent 40%, #f4f4f9 50%, #f4f4f9 100%);
        animation: wipe 1s forwards;
        pointer-events: none;
    }

    @keyframes wipe {
        0% { transform: translate(-30%, -30%); }
        100% { transform: translate(30%, 30%); }
    }

    /* 2. 果冻弹跳动画 (Jelly) */
    .jelly {
        animation: jelly-jump 0.8s forwards;
    }

    @keyframes jelly-jump {
        0% { transform: scale(0.5) translateX(-50%); }
        30% { transform: scale(1.15) translateX(-50%); }
        50% { transform: scale(0.9) translateX(-50%); }
        100% { transform: scale(1.0) translateX(-50%); }
    }


    /* --- 二级页面 (详情页) --- */
    .overlay {
        position: fixed;
        top: 0; left: 0;
        width: 100%; height: 100%;
        background: white;
        z-index: 200;
        transform: translateY(100%); /* 默认向下隐藏 */
        transition: transform 0.4s cubic-bezier(0.2, 0.8, 0.2, 1);
        display: flex;
        flex-direction: column;
    }
    
    .overlay.active {
        transform: translateY(0);
    }

    /* 详情页头部 */
    .detail-header {
        padding: 20px 40px;
        background: #fff;
        border-bottom: 1px solid #eee;
        display: flex;
        justify-content: space-between;
        align-items: center;
        box-shadow: 0 2px 10px rgba(0,0,0,0.05);
    }

    .close-btn {
        padding: 10px 25px;
        background: #333;
        color: white;
        border: none;
        border-radius: 30px;
        font-size: 16px;
        font-weight: bold;
        cursor: pointer;
        transition: background 0.2s;
    }
    .close-btn:hover { background: #555; }

    /* 详情页内容区 (可滚动) */
    .detail-body {
        flex: 1;
        overflow-y: auto;
        padding: 40px;
        max-width: 800px;
        margin: 0 auto;
        width: 100%;
        box-sizing: border-box;
    }

    .article-meta {
        color: #888;
        font-size: 14px;
        margin-bottom: 20px;
        border-left: 4px solid #00cc00;
        padding-left: 15px;
    }

    .article-title {
        font-size: 32px;
        margin: 0 0 10px 0;
        color: #222;
    }

    .article-img {
        width: 100%;
        height: 300px;
        object-fit: cover;
        border-radius: 12px;
        margin: 20px 0;
    }

    .article-text {
        line-height: 1.8;
        font-size: 18px;
        color: #444;
        white-space: pre-wrap; /* 保留换行符 */
    }

    .article-text strong {
        color: #000;
        display: block;
        margin-top: 20px;
    }

    /* 滚动条美化 */
    .detail-body::-webkit-scrollbar { width: 8px; }
    .detail-body::-webkit-scrollbar-thumb { background: #ccc; border-radius: 4px; }

</style>
</head>
<body data-dir="LTR">

    <!-- 1. 交互按钮 (小人) -->
    <div class="switch-btn" onclick="toggleDirection()" title="切换阅读方向">
        <div class="icon-man" id="manIcon">
            <svg viewBox="0 0 24 24">
                <path d="M19 13v-2c0-1.8-1.3-3.3-3-3.8V5c0-1.1-.9-2-2-2s-2 .9-2 2v1.3C10.8 6.1 10 7 10 8v2h-1v-2h-2v2c0 1.4.8 2.6 2 3.2V19h2v-5h2v5h2v-6h2z"/>
                <path d="M7 12l-4-2 2-3 3 1z" />
                <path d="M5 20l3-4h2l-2 5z"/>
            </svg>
        </div>
    </div>

    <!-- 标题 -->
    <div style="position:absolute; top:30px; font-weight:bold; color:#aaa; letter-spacing:2px;">GLOBAL NEWS</div>

    <!-- 2. 舞台 -->
    <div class="stage">
        <!-- 卡片由 JS 动态生成，或者写死在这里，我们用 HTML 写死方便理解 -->
        
        <!-- 新闻1: Tech/Oracle -->
        <div class="card" id="c0" onclick="openDetail(0)">
            <div class="card-content">
                <span class="card-tag">TECH</span>
                <div class="card-title">Oracle Slumps & AI Worries</div>
            </div>
        </div>

        <!-- 新闻2: Money/Billionaires -->
        <div class="card" id="c1" onclick="openDetail(1)">
            <div class="card-content">
                <span class="card-tag">BUSINESS</span>
                <div class="card-title">Record High Billionaires</div>
            </div>
        </div>

        <!-- 新闻3: Health/Sleep -->
        <div class="card" id="c2" onclick="openDetail(2)">
            <div class="card-content">
                <span class="card-tag">HEALTH</span>
                <div class="card-title">Perfect Sleep Time</div>
            </div>
        </div>
    </div>

    <!-- 3. 二级详情页 -->
    <div class="overlay" id="detailOverlay">
        <div class="detail-header">
            <h2 style="margin:0; font-size:18px; color:#555;">News Details</h2>
            <button class="close-btn" onclick="closeDetail()">✕ 关闭返回</button>
        </div>
        <div class="detail-body">
            <h1 class="article-title" id="d-title">标题</h1>
            <div class="article-meta" id="d-meta">作者 | 时间</div>
            <img src="" alt="News Image" class="article-img" id="d-img">
            <div class="article-text" id="d-content">内容...</div>
        </div>
    </div>

<script>
    // --- 新闻数据配置 (在此处修改图片链接) ---
    // 如果您有本地图片，请将 url 改为 'images/news1.jpg' 等
    const NEWS_DATA = [
        {
            // 新闻 1: Oracle
            id: 0,
            title: "Oracle slumps as gloomy forecasts, soaring spending fan AI bubble worries",
            tag: "TECH",
            // 这是一个网络的 Oracle/Tech 相关图片，您可以替换
            imgUrl: "https://images.unsplash.com/photo-1620712943543-bcc4688e7485?q=80&w=1000&auto=format&fit=crop",
            meta: "By Aditya Soni & Kanchana Chakravarty | Dec 12, 2025",
            content: `甲骨文因悲观预测和飞涨的消费而下跌，加剧了 AI 泡沫担忧。
            
**Summary 总结**
• Oracle shares drop after dour forecast, higher capex (甲骨文股价因预测阴郁及资本支出上升而下跌)
• AI-related stocks including Nvidia fall (包括英伟达在内的 AI 相关股票下跌)
• At least 13 brokerages slash PT on stock following results (至少有 13 家券商下调了目标价)

Dec 11 (Reuters) - Oracle (ORCL.N) shares sank 13% on Thursday, sparking a tech selloff as the company's massive spending and weak forecasts fanned doubts over how quickly the big bets on AI will pay off.
12 月 11 日（路透社）——甲骨文（ORCL.N）股价周四下跌 13%，引发科技股抛售，公司巨额支出和疲弱的预测加剧了人们对人工智能大注回报速度的质疑。

The disappointing predictions from the crucial OpenAI cloud-computing partner show the uneven returns from the nascent technology that many business leaders believe is the future but has so far yielded limited productivity gains.
来自关键 OpenAI 云计算合作伙伴的令人失望的预测显示，许多企业领导者认为是未来但迄今生产率提升有限的初兴技术回报不均。

Oracle, long a smaller cloud player, has vaulted into the artificial intelligence infrastructure race this year thanks to a $300 billion OpenAI deal. But the pact has also tethered the company's fortunes to the ChatGPT maker, with shares sliding in recent weeks on concern Google is pulling ahead of OpenAI.
甲骨文长期以来一直是较小的云平台，今年凭借一笔价值 3000 亿美元的 OpenAI 交易，跃入人工智能基础设施竞赛。但这项协议也将公司与 ChatGPT 制造商的命运紧密相连。

Investors have also dumped Oracle bonds on worries about its debt-fueled AI build-out. Big Tech's rising appetite for debt marked a departure for the companies that long depended on strong cash flows.
投资者也因担忧甲骨文债券的债务驱动 AI 建设而抛售。大型科技公司对债务的增长标志着长期依赖强现金流资助新项目支出的公司走向转变。

Tech executives have said the outlays are necessary for a technology that will transform work and make businesses more efficient, arguing the bigger risk is underinvesting, not overspending.
科技高管表示，这些投入对于一项能够改变工作、提升企业效率的技术来说是必要的，他们认为更大的风险是投资不足，而非超支。`
        },
        {
            // 新闻 2: Billionaires
            id: 1,
            title: "Planet Earth has never had so many billionaires",
            tag: "BUSINESS",
            // 金融/银行图片
            imgUrl: "https://images.unsplash.com/photo-1579621970563-ebec7560ff3e?q=80&w=1000&auto=format&fit=crop",
            meta: "By Ana Nicolaci da Costa | Dec 5, 2025",
            content: `地球从未有过如此多亿万富翁。
            
London — The number of billionaires worldwide – and their combined wealth – reached record highs this year, buoyed in particular by gains in tech stocks, Swiss bank UBS has found.
伦敦 — 瑞士银行瑞银发现，全球亿万富翁数量及其总财富今年创下历史新高，尤其得益于科技股的上涨。

The world had 2,919 billionaires on April 4, when UBS completed that part of its research, the largest number since the bank started collecting the data in 1995.
截至4月4日，全球亿万富翁为2,919人，这是自1995年银行开始收集数据以来的最高数字。

“With tech shares and financial asset prices generally rising, despite volatility, billionaires’ total wealth reached a new high,” UBS wrote in its annual report.
瑞银在年度报告中写道：“尽管存在波动，科技股和金融资产价格普遍上涨，亿万富翁的总财富达到了新的高点。”

Billionaires’ combined wealth reached $15.8 trillion, an increase of 13% over 12 months. Tech billionaires’ assets grew by almost a quarter to $3 trillion, bolstered by AI-linked companies such as Meta, Oracle and Nvidia.
亿万富翁的总财富达到15.8万亿美元，过去12个月增长13%。科技亿万富翁的资产增长近四分之一，达到 3 万亿美元，这得益于 Meta、甲骨文和英伟达等与人工智能相关公司的股价上涨。

The total number of US billionaires rose to 924, accounting for almost a third of the global billionaire population. China now has 470 billionaires, second only to the United States.
美国亿万富翁总数上升到924人，几乎占全球亿万富翁总数的三分之一。中国目前拥有470名亿万富翁，仅次于美国。

UBS surveyed its billionaire clients and they identified tariffs, geopolitics and policy uncertainty as the top risks of 2026.
瑞银调查发现，客户将关税、地缘政治和政策不确定性列为2026年的主要风险。`
        },
        {
            // 新闻 3: Sleep
            id: 2,
            title: "You might not need 8 hours of rest. Here’s how to find your perfect sleep time",
            tag: "HEALTH",
            // 睡眠/卧室图片
            imgUrl: "https://images.unsplash.com/photo-1541781777621-af1394378813?q=80&w=1000&auto=format&fit=crop",
            meta: "By Gina Park | Updated Dec 9, 2025",
            content: `你可能不需要8小时的休息。以下是找到理想睡眠时间的方法。

With so much to do heading into the busy holiday season, is anyone getting enough sleep? Most sleep experts advise that adults get seven to nine hours of sleep per night.
在忙碌的假日季临近之际，有没有人睡得够？大多数睡眠专家建议成年人每晚睡七到九小时。

**Sleep quality is just as important as sleep time (睡眠质量和睡眠时间同样重要)**
A lot of people tend to focus on how many hours of sleep they’re getting but neglect the quality of their sleep.
很多人关注的是自己睡了多少小时，却忽视了睡眠质量。

“There’s two different things going on in our bodies... our sleep pressure and circadian rhythms,” said Cunningham, assistant professor of psychology at Harvard Medical School.
“我们身体中有两种不同的因素... 那就是我们的睡眠压力和昼夜节律，”哈佛医学院心理学助理教授坎宁安说。

Sleep pressure builds up the longer that you’re awake. To get a good night’s worth of sleep, you want to get into bed when you’ve built up a lot of sleep pressure.
睡眠压力随着清醒时间增加而增加。要保证一夜好眠，最好在积累大量睡眠压力后上床。

**How many hours of sleep do you need? (你需要睡多少小时？)**
“That does not mean that every single person on the planet needs eight hours of sleep,” Cunningham said. “There are some people that really, truly only need five or six hours.”
“这并不意味着地球上的每个人都需要八小时的睡眠。有些人真的只需要五六个小时。”

To figure out how much sleep you need, keep a consistent bedtime. Go to bed when you feel sleepy. Then, find a period (like holidays) where you can sleep until you wake up naturally with no alarm.
要计算你需要多少睡眠，保持固定的睡觉时间。在你感到困倦时上床。然后，找一段时间（如假期），让自己自然醒来，不设闹钟。

After a few days of catching up, you’ll know your sleep time when you wake up consistently at the same time without external cues.
经过几天的补觉后，如果你连续几天在没有外部信号的情况下同一时间醒来，你就找到了你的睡眠时间。`
        }
    ];

    // --- 初始化逻辑 ---
    
    // 初始化卡片背景图
    NEWS_DATA.forEach((item, index) => {
        const card = document.getElementById('c' + index);
        card.style.backgroundImage = `url('${item.imgUrl}')`;
    });

    // 状态管理
    let direction = 'LTR'; // 默认从左到右 (Left To Right)
    // 顺序数组：[左边卡ID, 中间卡ID, 右边卡ID]
    // 初始要求: 新闻1(0)左, 新闻2(1)中, 新闻3(2)右
    let currentOrder = [0, 1, 2]; 
    let autoTimer = null;
    let isAnimating = false;

    const cardElements = [
        document.getElementById('c0'),
        document.getElementById('c1'),
        document.getElementById('c2')
    ];

    // --- 核心：渲染位置 ---
    function renderView(animate = false) {
        // 清除所有动画和位置类
        cardElements.forEach(el => {
            el.className = 'card'; // 重置
            void el.offsetWidth; // 强制重绘
        });

        const leftId = currentOrder[0];
        const centerId = currentOrder[1];
        const rightId = currentOrder[2];

        // 添加位置类
        cardElements[leftId].classList.add('slot-left');
        cardElements[centerId].classList.add('slot-center');
        cardElements[rightId].classList.add('slot-right');

        // 如果是动画切换产生的渲染，给新进入焦点的卡片加果冻效果
        if (animate) {
            // 谁是焦点？Center 总是视觉焦点
            cardElements[centerId].classList.add('jelly');
        }
    }

    // --- 核心：切换方向 ---
    function toggleDirection() {
        direction = direction === 'LTR' ? 'RTL' : 'LTR';
        
        const icon = document.getElementById('manIcon');
        if (direction === 'RTL') {
            // 变成从右向左：小人向右跑 (默认SVG是向右，不用翻转)
            icon.style.transform = 'scaleX(1)';
            document.body.setAttribute('data-dir', 'RTL');
        } else {
            // 变成从左向右：小人向左跑
            icon.style.transform = 'scaleX(-1)';
            document.body.setAttribute('data-dir', 'LTR');
        }
        
        resetTimer();
    }

    // --- 核心：自动轮播 ---
    function nextSlide() {
        if (isAnimating) return;
        isAnimating = true;

        // 1. 谁是当前的主角(Center)?
        const centerId = currentOrder[1];
        const centerCard = cardElements[centerId];

        // 2. 擦除动画 (当前主角消失)
        centerCard.classList.add('erasing');

        setTimeout(() => {
            // 3. 数组位移
            if (direction === 'LTR') {
                // 阅读方向向右 -> 内容向左流动 (左边的进来)
                // [0, 1, 2] -> [1, 2, 0] (错，这样是右边进来)
                // 应该是：把第一个移到最后
                const first = currentOrder.shift();
                currentOrder.push(first);
            } else {
                // 阅读方向向左 -> 内容向右流动
                const last = currentOrder.pop();
                currentOrder.unshift(last);
            }

            // 4. 更新视图 (新的 Center 会弹出来)
            renderView(true);
            isAnimating = false;

        }, 1000); // 1秒擦除
    }

    // --- 计时器 ---
    function startTimer() {
        // 5秒停留 + 1秒动画
        autoTimer = setInterval(nextSlide, 6000);
    }
    
    function resetTimer() {
        clearInterval(autoTimer);
        startTimer();
    }

    // --- 详情页逻辑 ---
    const overlay = document.getElementById('detailOverlay');
    const dTitle = document.getElementById('d-title');
    const dMeta = document.getElementById('d-meta');
    const dImg = document.getElementById('d-img');
    const dContent = document.getElementById('d-content');

    function openDetail(id) {
        clearInterval(autoTimer); // 暂停轮播
        const data = NEWS_DATA[id];
        
        dTitle.innerText = data.title;
        dMeta.innerText = data.meta;
        dImg.src = data.imgUrl;
        dContent.innerHTML = data.content; // 使用 innerHTML 解析换行

        overlay.classList.add('active');
    }

    function closeDetail() {
        overlay.classList.remove('active');
        resetTimer(); // 恢复轮播
    }

    // --- 启动 ---
    // 初始渲染
    renderView();
    // 默认小人向左跑 (LTR)
    document.getElementById('manIcon').style.transform = 'scaleX(-1)'; 
    startTimer();

</script>
</body>
</html>
