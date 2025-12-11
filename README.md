<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>我的新闻展示页</title>

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
    }

    .card {
        background: white;
        padding: 20px 24px;
        border-radius: 12px;
        margin-bottom: 20px;
        box-shadow: 0 4px 12px rgba(0,0,0,0.1);
        cursor: pointer;
        transition: transform 0.2s, box-shadow 0.2s;
    }

    .card:hover {
        transform: translateY(-3px);
        box-shadow: 0 6px 18px rgba(0,0,0,0.15);
    }

    .content {
        margin-top: 10px;
        display: none;
        line-height: 1.6;
        color: #444;
    }

    .show {
        display: block;
    }
</style>
</head>

<body>

<h1>欢迎来到我的新闻展示页</h1>

<!-- 新闻 1 -->
<div class="card" onclick="toggleContent('c1')">
    <h2>示例新闻标题 A</h2>
    <div id="c1" class="content">
        这是新闻内容的一个示例。点击卡片可以展开或者收起内容。
    </div>
</div>

<!-- 新闻 2 -->
<div class="card" onclick="toggleContent('c2')">
    <h2>示例新闻标题 B</h2>
    <div id="c2" class="content">
        新闻 B 的内容展示区域。可以放更长的新闻文本，也可以放图片或链接。
    </div>
</div>

<!-- 新闻 3 -->
<div class="card" onclick="toggleContent('c3')">
    <h2>示例新闻标题 C</h2>
    <div id="c3" class="content">
        新闻 C 的详细内容展示。
    </div>
</div>

<script>
function toggleContent(id) {
    const el = document.getElementById(id);
    el.classList.toggle("show");
}
</script>

</body>
</html>

