<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>新闻展示 - 最终优化版</title>

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

    /* --- 2. 左上角图标 (SVG绘制，外框不动内翻转) --- */
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
    
    .switch-btn:active {
        transform: scale(0.95);
    }

    /* SVG 内部样式 */
    .icon-svg {
        width: 100%;
        height: 100%;
        display: block;
    }
    
    /* 外框路径样式 */
    .frame-path {
        fill: none;
        stroke: #00cc00;
        stroke-width: 3;
        stroke-linecap: round;
        stroke-linejoin: round;
    }

    /* 小人路径样式 */
    .man-path {
        fill: #00cc00;
        transform-box: fill-box; /* 确保变换基于自身中心 */
        transform-origin: center;
        transition: transform 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
    }

    /* --- 3. 舞台布局 (一大两小) --- */
    .stage {
        position: relative;
        width: 900px;  /* 总宽度 */
        height: 500px; /* 总高度 */
        /* border: 1px dashed #ccc; 调试用 */
    }

    .card {
        position: absolute;
        border-radius: 12px;
        background-color: #ddd; /* 图片加载前的底色 */
        background-size: cover;
        background-position: center;
        cursor: pointer;
        box-shadow: 0 10px 20px rgba(0,0,0,0.15);
        overflow: hidden;
        transition: all 0.6s cubic-bezier(0.25, 1, 0.5, 1); /* 平滑过渡动画 */
        display: flex;
        align-items: flex-end;
    }

    /* 卡片上的文字遮罩 */
    .card-mask {
        width: 100%;
        background: linear-gradient(to top, rgba(0,0,0,0.85), transparent);
        padding: 20px;
        color: white;
        transform: translateY(0);
        transition: padding 0.3s;
    }
    .card:hover .card-mask {
        padding-bottom: 30px;
    }

    .card-tag {
        font-size: 12px;
        background: #00cc00;
        padding: 3px 8px;
        border-radius: 4px;
        margin-bottom: 5px;
        display: inline-block;
        font-weight: bold;
    }

    .card-title {
        font-size: 18px;
        font-weight: bold;
        line-height: 1.3;
        margin: 0;
    }
    
    /* 大图时的文字样式覆盖 */
    .slot-main .card-title {
        font-size: 28px;
    }
    .slot-main .card-tag {
        font-size: 14px;
        padding: 5px 12px;
    }

    /* --- 4. 槽位定义 (关键布局) --- */
    /* 
       Slot 0: 主图 (Main)
       Slot 1: 右上小图 (Sub Top)
       Slot 2: 右下小图 (Sub Bottom)
    */

    /* LTR 模式 (默认: 左大右小) */
    
    /* 主图位置 */
    .slot-main {
        left: 0;
        top: 0;
        width: 580px;
        height: 100%;
        z-index: 10;
    }

    /* 副图1 (上) */
    .slot-sub1 {
        left: 600px; /* 580 + 20间距 */
        top: 0;
        width: 300px;
        height: 240px; /* (500 - 20) / 2 */
        z-index: 5;
    }

    /* 副图2 (下) */
    .slot-sub2 {
        left: 600px;
        top: 260px; /* 240 + 20间距 */
        width: 300px;
        height: 240px;
        z-index: 5;
    }

    /* RTL 模式 (切换后: 右大左小) */
    /* 通过给 stage 加 class 来控制 */
    
    .stage.rtl-mode .slot-main {
        left: 320px; /* 300 + 20 */
        width: 580px;
    }
    
    .stage.rtl-mode .slot-sub1 {
        left: 0;
        width: 300px;
    }
    
    .stage.rtl-mode .slot-sub2 {
        left: 0;
        width: 300px;
    }


    /* --- 5. 动画效果 --- */

    /* 擦除动画 (只对当前要消失的主图有效) */
    .erasing::after {
        content: "";
        position: absolute;
        top: -50%; left: -50%;
        width: 200%; height: 200%;
        background: linear-gradient(135deg, transparent 40%, #f4f4f9 50%, #f4f4f9 100%);
        animation: wipe 0.8s forwards;
        pointer-events: none;
    }
    @keyframes wipe {
        0% { transform: translate(-30%, -30%); }
        100% { transform: translate(30%, 30%); }
    }

    /* 果冻弹跳 (只对新变成主图的卡片有效) */
    .jelly {
        animation: jelly-pop 0.8s forwards;
    }
    @keyframes jelly-pop {
        0% { transform: scale(0.95); }
        40% { transform: scale(1.02); }
        100% { transform: scale(1.0); }
    }

    /* --- 6. 二级详情页 (顶部导航栏风格) --- */
    .overlay {
        position: fixed;
        top: 0; left: 0; width: 100%; height: 100%;
        background: white;
        z-index: 200;
        transform: translateY(100%); /* 默认藏在下面 */
        transition: transform 0.3s ease-in-out;
        display: flex;
        flex-direction: column;
    }
    .overlay.active {
        transform: translateY(0);
    }

    /* 顶部导航栏 */
    .nav-bar {
        height: 60px;
        background: white;
        border-bottom: 1px solid #eee;
        box-shadow: 0 2px 10px rgba(0,0,0,0.05);
        display: flex;
        align-items: center;
        padding: 0 20px;
        position: sticky;
        top: 0;
        z-index: 10;
    }

    .back-btn {
        background: none;
        border: none;
        font-size: 16px;
        font-weight: 600;
        color: #333;
        cursor: pointer;
        display: flex;
        align-items: center;
        padding: 10px;
        border-radius: 8px;
        transition: background 0.2s;
    }
    .back-btn:hover {
        background: #f0f0f0;
    }
    .back-btn span {
        margin-left: 5px;
    }

    /* 内容区域 */
    .content-scroll {
        flex: 1;
        overflow-y: auto;
        padding: 40px;
    }
    .article-container {
        max-width: 800px;
        margin: 0 auto;
    }
    .article-title { font-size: 32px; color: #111; margin-bottom: 10px; }
    .article-meta { color: #888; font-size: 14px; margin-bottom: 25px; border-left: 3px solid #00cc00; padding-left: 10px; }
    .article-img { width: 100%; height: auto; border-radius: 8px; margin-bottom: 30px; }
    .article-body { font-size: 18px; line-height: 1.8; color: #444; white-space: pre-wrap; }
    
</style>
</head>
<body>

    <!-- 左上角图标 (SVG) -->
    <div class="switch-btn" onclick="toggleDirection()" title="切换布局方向">
        <svg class="icon-svg" viewBox="0 0 60 60">
            <!-- 1. 外框 (固定不动) -->
            <rect class="frame-path" x="5" y="5" width="50" height="50" rx="8" ry="8" />
            
            <!-- 2. 小人 (可翻转) -->
            <g class="man-path" id="manGroup">
                <!-- 头部 -->
                <circle cx="30" cy="20" r="5" />
                <!-- 身体/奔跑姿态 -->
                <path d="M 25 30 L 35 30 L 38 40 L 45 40 M 35 30 L 35 45 L 42 52 M 35 45 L 28 52" 
                      fill="none" stroke="#00cc00" stroke-width="4" stroke-linecap="round" stroke-linejoin="round"/>
                <!-- 简化的小人图形，模拟紧急出口样子 -->
            </g>
        </svg>
    </div>

    <!-- 舞台 -->
    <div class="stage" id="stage">
        <!-- 卡片 0 (新闻1: Oracle) -->
        <div class="card" id="card0" onclick="openDetail(0)">
            <div class="card-mask">
                <span class="card-tag">TECH</span>
                <div class="card-title">Oracle Slumps & AI Worries</div>
            </div>
        </div>

        <!-- 卡片 1 (新闻2: Bank/Billionaires) -->
        <div class="card" id="card1" onclick="openDetail(1)">
            <div class="card-mask">
                <span class="card-tag">BUSINESS</span>
                <div class="card-title">Record High Billionaires</div>
            </div>
        </div>

        <!-- 卡片 2 (新闻3: Climber/Sleep) -->
        <div class="card" id="card2" onclick="openDetail(2)">
            <div class="card-mask">
                <span class="card-tag">HEALTH</span>
                <div class="card-title">Perfect Sleep Time</div>
            </div>
        </div>
    </div>

    <!-- 详情页 (Overlay) -->
    <div class="overlay" id="overlay">
        <!-- 顶部导航栏 -->
        <div class="nav-bar">
            <button class="back-btn" onclick="closeDetail()">
                <!-- 返回箭头SVG -->
                <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="19" y1="12" x2="5" y2="12"></line><polyline points="12 19 5 12 12 5"></polyline></svg>
                <span>返回首页</span>
            </button>
            <div style="flex:1; text-align:center; font-weight:bold; color:#888;">NEWS READER</div>
            <div style="width:80px;"></div> <!-- 占位保持居中 -->
        </div>

        <!-- 滚动内容 -->
        <div class="content-scroll">
            <div class="article-container">
                <h1 class="article-title" id="d-title">Title</h1>
                <div class="article-meta" id="d-meta">Meta Info</div>
                <img src="" class="article-img" id="d-img">
                <div class="article-body" id="d-content">Content...</div>
            </div>
        </div>
    </div>

<script>
    // --- 1. 数据配置 (请确保图片文件名一致) ---
    const NEWS_DATA = [
        {
            // 新闻 1：甲骨文
            id: 0,
            img: "news1.jpg", 
            title: "Oracle slumps as gloomy forecasts, soaring spending fan AI bubble worries",
            meta: "By Aditya Soni | Dec 12, 2025",
            content: `甲骨文因悲观预测和飞涨的消费而下跌，粉丝 AI 泡沫担忧。
            
**Summary 总结**
• Oracle shares drop after dour forecast.
• AI-related stocks including Nvidia fall.

Dec 11 (Reuters) - Oracle (ORCL.N) shares sank 13% on Thursday, sparking a tech selloff as the company's massive spending and weak forecasts fanned doubts over how quickly the big bets on AI will pay off.
12 月 11 日（路透社）——甲骨文（ORCL.N）股价周四下跌 13%，引发科技股抛售...`
        },
        {
            // 新闻 2：银行/地砖 (亿万富翁)
            id: 1,
            img: "news2.jpg", 
            title: "Planet Earth has never had so many billionaires",
            meta: "By Ana Nicolaci da Costa | Dec 5, 2025",
            content: `地球从未有过如此多亿万富翁。
            
London — The number of billionaires worldwide reached record highs this year, buoyed by tech stocks, Swiss bank UBS has found.
瑞士银行瑞银发现，全球亿万富翁数量及其总财富今年创下历史新高...`
        },
        {
            // 新闻 3：登山者 (睡眠)
            id: 2,
            img: "news3.jpg", 
            title: "You might not need 8 hours of rest. Finding your perfect sleep time",
            meta: "By Gina Park | Updated Dec 9, 2025",
            content: `你可能不需要8小时的休息。以下是找到理想睡眠时间的方法。
            
Most sleep experts advise that adults get seven to nine hours of sleep per night.
大多数睡眠专家建议成年人每晚睡七到九小时。`
        }
    ];

    // --- 2. 状态管理 ---
    // 卡片DOM元素
    const cards = [
        document.getElementById('card0'),
        document.getElementById('card1'),
        document.getElementById('card2')
    ];
    
    // 初始化背景图
    NEWS_DATA.forEach((item, index) => {
        cards[index].style.backgroundImage = `url('${item.img}')`;
    });

    // 逻辑变量
    let isRTL = false; // 默认 LTR (左大)
    let autoPlayTimer = null;
    let isAnimating = false;

    // 位置映射数组 [Main, Sub1, Sub2]
    // 初始状态: 0号卡在Main, 1号卡在Sub1, 2号卡在Sub2
    let order = [0, 1, 2]; 

    // --- 3. 核心功能: 渲染位置 ---
    function updateLayout(animate = false) {
        // 清理旧类
        cards.forEach(c => {
            c.classList.remove('slot-main', 'slot-sub1', 'slot-sub2', 'erasing', 'jelly');
            void c.offsetWidth; // 强制重绘
        });

        const mainIdx = order[0];
        const sub1Idx = order[1];
        const sub2Idx = order[2];

        // 添加新位置类
        cards[mainIdx].classList.add('slot-main');
        cards[sub1Idx].classList.add('slot-sub1');
        cards[sub2Idx].classList.add('slot-sub2');

        // 如果是轮播产生的变化，给新的主图加果冻效果
        if (animate) {
            cards[mainIdx].classList.add('jelly');
        }
    }

    // --- 4. 交互: 切换方向 ---
    function toggleDirection() {
        isRTL = !isRTL;
        
        const stage = document.getElementById('stage');
        const manGroup = document.getElementById('manGroup');
        
        if (isRTL) {
            stage.classList.add('rtl-mode');
            // 小人翻转 (scaleX -1)
            manGroup.style.transform = "scaleX(-1)";
        } else {
            stage.classList.remove('rtl-mode');
            // 小人复原
            manGroup.style.transform = "scaleX(1)";
        }

        // 重置计时器
        resetTimer();
    }

    // --- 5. 自动轮播 ---
    function nextSlide() {
        if (isAnimating) return;
        isAnimating = true;

        // 1. 获取当前主图，添加擦除效果
        const currentMain = cards[order[0]];
        currentMain.classList.add('erasing');

        // 2. 等待动画结束，轮转数组
        setTimeout(() => {
            // 数组轮转算法: 把第一个(Main)移到最后(Sub2)，其他的往前顶
            // [0, 1, 2] -> [1, 2, 0]
            const first = order.shift();
            order.push(first);

            // 3. 更新视图
            updateLayout(true);
            isAnimating = false;

        }, 800); // 0.8s 擦除时间
    }

    function startTimer() {
        // 5秒停留 + 0.8秒动画 ≈ 6秒
        autoTimer = setInterval(nextSlide, 5800);
    }
    
    function resetTimer() {
        clearInterval(autoTimer);
        startTimer();
    }

    // --- 6. 详情页逻辑 ---
    const overlay = document.getElementById('overlay');
    const dTitle = document.getElementById('d-title');
    const dMeta = document.getElementById('d-meta');
    const dImg = document.getElementById('d-img');
    const dContent = document.getElementById('d-content');

    function openDetail(index) {
        clearInterval(autoTimer);
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
