---
title: "情报网"
slug: "monitor"
description: "接入全球游戏资讯加密数据流。"
image: "hoshino1.jpg"
menu:
    main:
        weight: -15
        params:
            icon: activity 
---

<div id="monitor-terminal">
    <div class="monitor-header">
        <span class="status-dot"></span> 
        NETWORK STATUS: <span style="color:#4ade80">ONLINE</span>
        <span class="scan-line"></span>
    </div>
    
    <div id="system-log">
        > Initializing connection protocols...<br>
        > Bypassing region locks...<br>
        > Accessing Gematsu mainframe... <span class="blink">_</span>
    </div>

    <div id="tweet-feed" class="feed-container">
        </div>
</div>

<script src="https://unpkg.com/rss-parser/dist/rss-parser.min.js"></script>

<script>
// ================= 情报源配置 =================
// 这些是目前最稳定的 RSS 源，涵盖了主要的游戏新闻
const sources = [
    { name: 'GEMATSU', url: 'https://gematsu.com/feed' },
    { name: 'SILICONERA', url: 'https://www.siliconera.com/feed/' },
    { name: 'EUROGAMER', url: 'https://www.eurogamer.net/?format=rss' }
];

// 使用 rss2json API 绕过跨域限制
const convertToApi = (url) => {
    // 加上 api_key 参数位是为了防止未来你需要填 key，现在留空即可
    return `https://api.rss2json.com/v1/api.json?rss_url=${encodeURIComponent(url)}`; 
};

async function fetchIntel() {
    const feedContainer = document.getElementById('tweet-feed');
    const logContainer = document.getElementById('system-log');
    let allItems = [];
    let logHtml = logContainer.innerHTML;

    console.log("正在初始化情报链路..."); 

    // 并行抓取数据
    const promises = sources.map(source => 
        fetch(convertToApi(source.url))
            .then(res => {
                if (!res.ok) throw new Error('Network response was not ok');
                return res.json();
            })
            .then(data => {
                if(data.status === 'ok' && data.items) {
                    console.log(`[${source.name}] 数据获取成功: ${data.items.length} 条`);
                    // 在界面上实时更新日志
                    logHtml += `> [${source.name}] CONNECTION ESTABLISHED.<br>`;
                    logContainer.innerHTML = logHtml;
                    return data.items.map(item => ({...item, source: source.name}));
                } else {
                    console.warn(`[${source.name}] 接口返回错误:`, data);
                    return [];
                }
            })
            .catch(err => {
                console.error(`[${source.name}] 链路连接失败:`, err);
                logHtml += `<span style="color:#ef4444">> ERROR: ${source.name} CONNECTION LOST</span><br>`;
                logContainer.innerHTML = logHtml;
                return [];
            })
    );

    const results = await Promise.all(promises);
    results.forEach(items => allItems = allItems.concat(items));
    
    // 渲染逻辑
    if (allItems.length === 0) {
        feedContainer.innerHTML = `
            <div class='intel-card' style='border-left-color: #ef4444'>
                <div class='intel-body'>
                    <div class='intel-title' style='color:#ef4444'>无法建立数据流连接</div>
                    <div class='intel-desc'>
                        SYSTEM ERROR: NO DATA RECEIVED.<br><br>
                        可能原因：<br>
                        1. 您的浏览器安装了广告拦截插件 (AdBlock)，拦截了 RSS 请求。<br>
                        2. 免费 API 请求次数暂时超限，请一小时后再试。<br>
                    </div>
                </div>
            </div>`;
        return;
    }

    // 按时间倒序排序
    allItems.sort((a, b) => new Date(b.pubDate) - new Date(a.pubDate));
    
    // 只取最新的 20 条，防止页面过长
    const recentIntel = allItems.slice(0, 20);

    feedContainer.innerHTML = recentIntel.map(item => {
        const date = new Date(item.pubDate);
        const timeStr = date.toLocaleTimeString('en-US', { hour12: false });
        
        // 尝试提取图片
        const content = item.content || item.description || "";
        const imgMatch = content.match(/<img[^>]+src="([^">]+)"/);
        const imgTag = imgMatch ? `<div class="intel-img-box"><img src="${imgMatch[1]}" class="intel-img" loading="lazy"></div>` : '';

        // 清理摘要里的 HTML 标签
        let cleanDesc = (item.description || "").replace(/<[^>]+>/g, '').substring(0, 100) + '...';

        return `
            <div class="intel-card">
                <div class="intel-header">
                    <span class="source-tag">[${item.source}]</span>
                    <span class="time-tag">${timeStr}</span>
                </div>
                <div class="intel-body">
                    <a href="${item.link}" target="_blank" class="intel-title">${item.title}</a>
                    <div class="intel-meta">
                        ${imgTag}
                        <div class="intel-desc">${cleanDesc}</div>
                    </div>
                </div>
            </div>
        `;
    }).join('');
    
    // 数据加载完 1.5 秒后，自动折叠系统日志
    setTimeout(() => {
        logContainer.style.display = 'none';
    }, 1500);
}

// 启动程序
fetchIntel();
</script>

<style>
#monitor-terminal {
    background: #0b1120;
    border: 1px solid #334155;
    padding: 20px;
    font-family: 'Consolas', 'Monaco', monospace;
    position: relative;
    overflow: hidden;
    min-height: 400px;
}

/* 扫描线特效 */
.monitor-header {
    border-bottom: 2px solid #334155;
    padding-bottom: 10px;
    margin-bottom: 15px;
    color: #e2e8f0;
    font-weight: bold;
    letter-spacing: 1px;
    position: relative;
}
.scan-line {
    position: absolute;
    top: 0; left: 0; width: 100%; height: 2px;
    background: rgba(74, 222, 128, 0.5);
    animation: scan 3s linear infinite;
    opacity: 0.5;
}
@keyframes scan { 0% {top:0} 100% {top:100%} }

.status-dot {
    display: inline-block; width: 8px; height: 8px;
    background: #4ade80; border-radius: 50%;
    margin-right: 8px;
    box-shadow: 0 0 8px #4ade80;
    animation: pulse 2s infinite;
}

#system-log {
    color: #64748b; font-size: 0.85rem; line-height: 1.5;
    margin-bottom: 20px; font-family: monospace;
}
.blink { animation: blink 1s infinite; }
@keyframes blink { 0% {opacity:1} 50% {opacity:0} 100% {opacity:1} }

/* 情报卡片 */
.intel-card {
    background: #1e293b;
    border-left: 3px solid #475569;
    margin-bottom: 15px;
    padding: 12px;
    transition: all 0.2s;
}
.intel-card:hover {
    border-left-color: #fdba74; /* 悬停变橙色 */
    background: #253045;
    transform: translateX(5px);
}

.intel-header {
    font-size: 0.75rem; color: #94a3b8;
    margin-bottom: 5px; display: flex; justify-content: space-between;
}
.source-tag { color: #fdba74; font-weight: bold; }

.intel-title {
    color: #e2e8f0; font-weight: bold; font-size: 1.1rem;
    text-decoration: none; display: block; margin-bottom: 8px;
}
.intel-title:hover { color: #4ade80; text-decoration: underline; }

.intel-meta { display: flex; gap: 15px; }
.intel-img-box { flex: 0 0 100px; height: 60px; overflow: hidden; border: 1px solid #475569; background: #000;}
.intel-img { width: 100%; height: 100%; object-fit: cover; }
.intel-desc { font-size: 0.9rem; color: #cbd5e1; line-height: 1.4; word-break: break-word; }

@media (max-width: 600px) {
    .intel-meta { flex-direction: column; }
    .intel-img-box { flex: 0 0 auto; height: 120px; width: 100%; }
}
</style>