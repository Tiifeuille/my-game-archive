---
title: "情报监控"
slug: "monitor"
description: "实时追踪目标对象的加密通讯信号。"
image: "hoshino1.jpg"
menu:
    main:
        weight: -15
        params:
            icon: activity # 换成心跳或信号图标
---

<div id="monitor-terminal">
    <div class="monitor-header">
        <span class="status-dot"></span> 
        SIGNAL RECEIVING: <span id="target-count">3</span> TARGETS
    </div>
    <div id="tweet-feed" class="loading-text">
        正在建立加密连接...
    </div>
</div>

<script src="https://unpkg.com/rss-parser/dist/rss-parser.min.js"></script>

<script>
// =================配置区域=================
// 在这里填入你想监控的推特账号 ID (不需要 @)
const targets = [
    'HIDEO_KOJIMA_EN',  // 小岛秀夫
    'FromSoftware_PR',  // FromSoftware
    'hakonemai',        // 比如某个画师
];

// Nitter 实例列表 (如果一个挂了会自动换，这是为了稳定性)
// 你可以在 https://status.d420.de/ 找更多活着的实例
const nitterInstances = [
    'https://nitter.net',
    'https://nitter.cz',
    'https://nitter.privacydev.net'
];
// =========================================

// 随机选一个 Nitter 实例防止单点故障
const nitterBase = nitterInstances[Math.floor(Math.random() * nitterInstances.length)];

// 使用 rss2json API 绕过跨域限制 (因为是纯前端静态网页)
const convertToApi = (user) => {
    const rssUrl = `${nitterBase}/${user}/rss`;
    return `https://api.rss2json.com/v1/api.json?rss_url=${encodeURIComponent(rssUrl)}`;
};

async function fetchTweets() {
    const feedContainer = document.getElementById('tweet-feed');
    let allTweets = [];

    // 并行抓取所有人的推文
    const promises = targets.map(user => 
        fetch(convertToApi(user))
            .then(res => res.json())
            .then(data => {
                if(data.items) {
                    // 给每条推文打上作者标签
                    return data.items.map(item => ({...item, author: user}));
                }
                return [];
            })
            .catch(err => {
                console.error(`丢失信号: ${user}`, err);
                return [];
            })
    );

    const results = await Promise.all(promises);
    
    // 合并并按时间倒序排列
    results.forEach(tweets => allTweets = allTweets.concat(tweets));
    allTweets.sort((a, b) => new Date(b.pubDate) - new Date(a.pubDate));

    // 渲染
    if (allTweets.length === 0) {
        feedContainer.innerHTML = "信号丢失。请检查通讯链路或目标是否静默。";
        return;
    }

    feedContainer.innerHTML = allTweets.map(tweet => {
        // 清理一下 Nitter RSS 里自带的图片样式，用我们自己的
        let content = tweet.content.replace(/<img/g, '<img class="tweet-img"');
        
        // 格式化时间
        const date = new Date(tweet.pubDate).toLocaleString();

        return `
            <div class="tweet-card">
                <div class="tweet-meta">
                    <span class="tweet-author">@${tweet.author}</span>
                    <span class="tweet-date">${date}</span>
                </div>
                <div class="tweet-content">${content}</div>
                <a href="${tweet.link}" target="_blank" class="tweet-link">>> SOURCE LINK</a>
            </div>
        `;
    }).join('');
}

fetchTweets();
</script>

<style>
/* 这里写专门针对监控室的 CSS */
#monitor-terminal {
    background: #0f172a;
    border: 1px solid #334155;
    padding: 20px;
    box-shadow: inset 0 0 20px rgba(0,0,0,0.5);
    font-family: 'Consolas', 'Monaco', monospace; /* 强制用等宽字体 */
}

.monitor-header {
    border-bottom: 2px solid #334155;
    padding-bottom: 15px;
    margin-bottom: 20px;
    color: #4ade80; /* 终端绿 */
    font-weight: bold;
    letter-spacing: 2px;
}

.status-dot {
    display: inline-block;
    width: 10px;
    height: 10px;
    background: #4ade80;
    border-radius: 50%;
    margin-right: 10px;
    box-shadow: 0 0 5px #4ade80;
    animation: blink 2s infinite;
}

@keyframes blink { 0% {opacity: 1;} 50% {opacity: 0.3;} 100% {opacity: 1;} }

.tweet-card {
    background: #1e293b;
    border-left: 3px solid #64748b;
    padding: 15px;
    margin-bottom: 20px;
    transition: all 0.3s ease;
}

.tweet-card:hover {
    border-left-color: #fdba74; /* 悬停变橙色 */
    background: #253045;
}

.tweet-meta {
    font-size: 0.85rem;
    color: #94a3b8;
    margin-bottom: 10px;
    display: flex;
    justify-content: space-between;
}

.tweet-author {
    color: #fdba74;
    font-weight: bold;
}

.tweet-content {
    color: #e2e8f0;
    line-height: 1.6;
    font-size: 0.95rem;
}

/* 推文里的图片样式 */
.tweet-img {
    max-width: 100%;
    border: 1px solid #475569;
    margin-top: 10px;
    opacity: 0.9;
}

.tweet-link {
    display: block;
    margin-top: 10px;
    font-size: 0.8rem;
    color: #64748b;
    text-decoration: none;
    text-align: right;
}
.tweet-link:hover { color: #4ade80; }

.loading-text { color: #94a3b8; font-style: italic; }
</style>

