<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>交互式新闻展示</title>

<style>
    :root {
        --bg-color: #f5f5f5;
        --card-shadow: 0 10px 25px rgba(0,0,0,0.2);
        --erase-duration: 1s;
        --jelly-duration: 0.8s;
        /* 定义三个位置的大小变量 */
        --size-lg: 300px;
        --size-md: 200px;
        --size-sm: 140px;
    }

    body {
        font-family: 'Segoe UI', sans-serif;
        background: var(--bg-color);
        margin: 0;
        height: 100vh;
        overflow: hidden; /* 防止动画溢出滚动条 */
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
    }

    /* --- 1. 左上角切换按钮 --- */
    .switch-btn {
        position: absolute;
        top: 20px;
        left: 20px;
        width: 60px;
        height: 60px;
        border: 3px solid #00cc00;
        border-radius: 8px;
        background: white;
        cursor: pointer;
        display: flex;
        align-items: center;
        justify-content: center;
        z-index: 100;
        transition: transform 0.2s;
    }

    .switch-btn:active {
        transform: scale(0.95);
    }

    /* 小人图标容器 */
    .icon-man {
        width: 40px;
        height: 40px;
        transition: transform 0.4s ease; /* 图标翻转动画 */
    }
    
    /* 如果你需要替换为 PNG，请取消下面注释并设置背景图 */
    /* 
    .icon-man {
        background-image: url('your-man-right.png');
        background-size: contain;
        background-repeat: no-repeat;
    } 
    */

    /* SVG 样式 (模拟紧急出口小人) */
    .icon-man svg {
        width: 100%;
        height: 100%;
        fill: #00cc00;
    }

    /* --- 2. 封面容器与布局 --- */
    .stage {
        position: relative;
        width: 900px; /* 舞台宽度 */
        height: 400px;
        display: flex;
        align-items: center;
        justify-content: center;
    }

    .news-card {
        position: absolute;
        border-radius: 15px;
        box-shadow: var(--card-shadow);
        cursor: pointer;
        display: flex;
        align-items: center;
        justify-content: center;
        color: white;
        font-size: 24px;
        font-weight: bold;
        text-align: center;
        transition: all 0.5s ease-in-out; /* 默认位移过渡 */
        overflow: hidden; /* 必须开启，为了擦除效果 */
    }

    .news-card:hover {
        filter: brightness(1.1);
    }

    /* --- 3. 位置定义 (基于绝对定位) --- */
    /* 
       RTL模式(默认): Left(Small) - Center(Med) - Right(Large)
       LTR模式: Left(Large) - Center(Med) - Right(Small)
       
       我们使用 CSS 变量来动态控制位置，JS 只需要切换 body 的类名
    */

    /* 默认 (RTL) 的位置坐标 */
    body[data-dir="RTL"] .pos-small {
        width: var(--size-sm); height: var(--size-sm);
        left: 50px; z-index: 1; opacity: 0.6;
    }
    body[data-dir="RTL"] .pos-medium {
        width: var(--size-md); height: var(--size-md);
        left: 350px; /* 居中位置微调 */
        transform: translateX(-50%);
        z-index: 2; opacity: 0.8;
    }
    body[data-dir="RTL"] .pos-large {
        width: var(--size-lg); height: var(--size-lg);
        right: 50px; z-index: 3; opacity: 1;
    }

    /* LTR 模式的位置坐标 */
    body[data-dir="LTR"] .pos-small {
        width: var(--size-sm); height: var(--size-sm);
        right: 50px; z-index: 1; opacity: 0.6;
    }
    body[data-dir="LTR"] .pos-medium {
        width: var(--size-md); height: var(--size-md);
        left: 50%;
        transform: translateX(-50%);
        z-index: 2; opacity: 0.8;
    }
    body[data-dir="LTR"] .pos-large {
        width: var(--size-lg); height: var(--size-lg);
        left: 50px; z-index: 3; opacity: 1;
    }


    /* --- 4. 动画特效 --- */

    /* (A) 黑板擦除动画 (Chalk Erase) */
    /* 原理：使用一个伪元素遮罩，斜向划过 */
    .erase-anim::after {
        content: '';
        position: absolute;
        top: -50%;
        left: -50%;
        width: 200%;
        height: 200%;
        background: linear-gradient(
            135deg, 
            transparent 45%, 
            var(--bg-color) 50%, 
            var(--bg-color) 100%
        );
        opacity: 0;
        pointer-events: none;
    }

    .erasing::after {
        opacity: 1;
        animation: wipeDiagonally 1s forwards;
    }

    @keyframes wipeDiagonally {
        0% { transform: translate(-30%, -30%); }
        100% { transform: translate(30%, 30%); }
    }

    /* (B) 果冻弹跳动画 (Jelly) */
    .jelly-anim {
        animation: jellyPop 0.6s;
    }

    @keyframes jellyPop {
        0% { transform: scale(0.5); }
        40% { transform: scale(1.15); }
        60% { transform: scale(0.95); }
        80% { transform: scale(1.05); }
        100% { transform: scale(1); }
    }
    
    /* 修正 LTR/RTL 模式下 Middle 位置有 translateX 的情况，需要混合 jelly */
    body[data-dir="RTL"] .pos-medium.jelly-anim,
    body[data-dir="LTR"] .pos-medium.jelly-anim {
        /* 如果是在中间位置触发 jelly，需要保持 translateX(-50%) */
        animation: jellyPopCenter 0.6s;
    }

    @keyframes jellyPopCenter {
        0% { transform: translateX(-50%) scale(0.5); }
        40% { transform: translateX(-50%) scale(1.15); }
        60% { transform: translateX(-50%) scale(0.95); }
        80% { transform: translateX(-50%) scale(1.05); }
        100% { transform: translateX(-50%) scale(1); }
    }

    /* --- 5. 二级页面 (模拟) --- */
    .page-overlay {
        position: fixed;
        top: 0; left: 0; width: 100%; height: 100%;
        background: white;
        z-index: 200;
        display: none; /* 默认隐藏 */
        flex-direction: column;
        padding: 40px;
        box-sizing: border-box;
        overflow-y: auto;
    }
    
    .page-overlay.active {
        display: flex;
        animation: fadeIn 0.3s;
    }

    .back-btn {
        padding: 10px 20px;
        background: #333;
        color: white;
        border: none;
        cursor: pointer;
        align-self: flex-start;
        margin-bottom: 20px;
        border-radius: 5px;
    }

    @keyframes fadeIn {
        from { opacity: 0; transform: translateY(20px); }
        to { opacity: 1; transform: translateY(0); }
    }

</style>
</head>

<!-- 默认方向 RTL -->
<body data-dir="RTL">

    <!-- 切换按钮 -->
    <div class="switch-btn" onclick="toggleDirection()">
        <div class="icon-man" id="manIcon">
            <!-- 模拟 SVG 图标：向右跑的小人 -->
            <svg viewBox="0 0 24 24">
                <path d="M19 13v-2c0-1.8-1.3-3.3-3-3.8V5c0-1.1-.9-2-2-2s-2 .9-2 2v1.3C10.8 6.1 10 7 10 8v2h-1v-2h-2v2c0 1.4.8 2.6 2 3.2V19h2v-5h2v5h2v-6h2z"/>
                <path d="M7 12l-4-2 2-3 3 1z" /> <!-- 手臂 -->
                <path d="M5 20l3-4h2l-2 5z"/> <!-- 腿 -->
            </svg>
        </div>
    </div>

    <h1>NEWS FLASH</h1>

    <!-- 舞台容器 -->
    <div class="stage">
        <!-- 新闻块 1 -->
        <div class="news-card pos-small erase-anim" id="card1" style="background: #4da6ff;" onclick="openPage(1)">
            TECH
        </div>
        <!-- 新闻块 2 -->
        <div class="news-card pos-medium erase-anim" id="card2" style="background: #ff6666;" onclick="openPage(2)">
            SPORT
        </div>
        <!-- 新闻块 3 -->
        <div class="news-card pos-large erase-anim" id="card3" style="background: #66cc66;" onclick="openPage(3)">
            NATURE
        </div>
    </div>

    <!-- 提示信息 -->
    <p style="color:#999; margin-top:50px;">当前最大封面将在 5秒 后自动切换</p>

    <!-- 二级页面内容 -->
    <div class="page-overlay" id="newsPage">
        <button class="back-btn" onclick="closePage()">← 返回首页</button>
        <h1 id="pageTitle">新闻标题</h1>
        <div id="pageContent" style="line-height:1.6; color:#555;">
            这里是新闻内容...
        </div>
    </div>

<script>
    // 配置
    const CARDS = [
        document.getElementById('card1'),
        document.getElementById('card2'),
        document.getElementById('card3')
    ];
    
    // 状态
    let currentDir = 'RTL'; // 'RTL' (右大) or 'LTR' (左大)
    let autoPlayTimer = null;
    let isAnimating = false;

    // 初始位置顺序：[Small, Medium, Large]
    // 数组 index 0 是小位置, 1 是中位置, 2 是大位置
    let positionSlots = [0, 1, 2]; // 卡片索引，0号卡片在小位置，1号在中位置...

    // 内容数据
    const NEWS_DATA = {
        1: { title: "科技前沿: TECH", content: "这里是关于蓝色科技板块的详细报道。点击左上角按钮可以切换首页阅读顺序。" },
        2: { title: "体育赛事: SPORT", content: "这里是红色体育板块的详细报道。注意观察封面的果冻弹跳效果。" },
        3: { title: "自然探索: NATURE", content: "这里是绿色自然板块的详细报道。当阅读方向改变时，小人图标也会翻转。" }
    };

    // 初始化
    function init() {
        renderPositions();
        startAutoPlay();
    }

    // 渲染卡片位置类名
    function renderPositions() {
        const positions = ['pos-small', 'pos-medium', 'pos-large'];
        
        CARDS.forEach((card, i) => {
            // 移除所有位置类和动画类
            card.classList.remove('pos-small', 'pos-medium', 'pos-large', 'erasing', 'jelly-anim');
            
            // 找到这张卡片当前在哪个槽位
            const slotIndex = positionSlots.indexOf(i);
            card.classList.add(positions[slotIndex]);
        });
    }

    // --- 核心逻辑：切换阅读方向 ---
    function toggleDirection() {
        currentDir = currentDir === 'RTL' ? 'LTR' : 'RTL';
        document.body.setAttribute('data-dir', currentDir);
        
        // 翻转小人图标 (Scale X)
        const manIcon = document.getElementById('manIcon');
        if (currentDir === 'LTR') {
            // LTR模式：阅读从左到右，小人朝左跑 (翻转)
            manIcon.style.transform = 'scaleX(-1)'; 
        } else {
            // RTL模式：阅读从右到左，小人朝右跑 (默认)
            manIcon.style.transform = 'scaleX(1)';
        }

        // 重置计时器，从新方向的最大封面开始看
        resetAutoPlay();
    }

    // --- 核心逻辑：自动轮播 ---
    function nextCycle() {
        if (isAnimating) return;
        isAnimating = true;

        // 1. 找到当前在 "Large" 位置的卡片索引
        const largeSlotIndex = 2; // 定义中 2 号位是大图
        const largeCardIndex = positionSlots[largeSlotIndex];
        const largeCard = CARDS[largeCardIndex];

        // 2. 执行“擦除”动画 (1秒)
        largeCard.classList.add('erasing');

        setTimeout(() => {
            // 3. 动画结束后，更新数组顺序
            // 逻辑：Small -> Medium, Medium -> Large, Large -> Small (循环)
            // 也就是数组整体向左移一位
            const first = positionSlots.shift(); 
            positionSlots.push(first);
            
            // 4. 重新渲染位置（瞬间切换，但视觉上因为有动画类会平滑）
            renderPositions();

            // 5. 给新上位的 Large 卡片（原 Medium）添加果冻动画
            const newLargeCardIndex = positionSlots[largeSlotIndex];
            const newLargeCard = CARDS[newLargeCardIndex];
            
            // 强制重绘以触发动画
            void newLargeCard.offsetWidth; 
            newLargeCard.classList.add('jelly-anim');

            isAnimating = false;
        }, 1000); // 1秒擦除时间
    }

    function startAutoPlay() {
        if (autoPlayTimer) clearInterval(autoPlayTimer);
        // 5秒停留 + 1秒动画 = 6秒周期，但题目要求停留5秒后触发
        autoPlayTimer = setInterval(nextCycle, 5000 + 1000); 
    }

    function resetAutoPlay() {
        clearInterval(autoPlayTimer);
        // 立即执行一次还是重新计时？通常是重新开始计时
        startAutoPlay();
    }

    // --- 页面交互 ---
    function openPage(id) {
        // 停止轮播
        clearInterval(autoPlayTimer);
        
        const data = NEWS_DATA[id];
        document.getElementById('pageTitle').innerText = data.title;
        document.getElementById('pageContent').innerText = data.content;
        
        document.getElementById('newsPage').classList.add('active');
    }

    function closePage() {
        document.getElementById('newsPage').classList.remove('active');
        // 恢复轮播
        startAutoPlay();
    }

    // 启动
    init();

</script>

</body>
</html>
