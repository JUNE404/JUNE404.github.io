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
