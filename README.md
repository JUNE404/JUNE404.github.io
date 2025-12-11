<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>互动示例页面</title>

<style>
    body {
        font-family: sans-serif;
        background: #f5f5f5;
        padding: 40px;
        max-width: 800px;
        margin: auto;
    }

    h1 {
        text-align: center;
        margin-bottom: 40px;
        font-size: 28px;
    }

    .color-block {
        width: 150px;
        height: 150px;
        border-radius: 12px;
        background: #4da6ff;
        margin: 20px auto;
        box-shadow: 0 4px 12px rgba(0,0,0,0.15);
        cursor: pointer;
        transition: background 0.3s, transform 0.2s;
    }

    .color-block:hover {
        transform: scale(1.05);
    }
</style>
</head>

<body>

<h1>点击下面这个颜色块看看互动效果</h1>

<div class="color-block" onclick="changeColor()"></div>

<script>
function changeColor() {
    const block = document.querySelector('.color-block');
    const colors = ["#4da6ff", "#ff6666", "#66cc66", "#ffcc33", "#9933ff"];
    // 随机换颜色
    block.style.background = colors[Math.floor(Math.random() * colors.length)];
}
</script>

</body>
</html>
