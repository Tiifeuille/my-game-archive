---
title: "情报网 "
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
        > Accessing Gematsu mainframe... [SUCCESS]<br>
        > Accessing Siliconera database... [SUCCESS]<br>
        > Stream decrypted. Rendering feed...
    </div>

    <div id="tweet-feed" class="feed-container">
        </div>
</div>

<script src="https://unpkg.com/rss-parser/dist/rss-parser.min.js"></script>

<script>
// ================= 情报源配置 (这里换成了超稳的新闻源) =================
const sources = [
    { name: 'GEMATSU', url: 'https://gematsu.com/feed' },
    { name: 'SILICONERA', url: 'https://www.siliconera.com/feed/' },
    { name: 'EUROGAMER', url: 'https://www.eurogamer.net/?format=rss' }
];
// ====================================================================

const convertToApi = (url) => {
    return `https://api.rss2json.com/v1/api.json?rss_url=${encodeURIComponent(url)}`;
};

async function fetchIntel() {
    const feedContainer = document.getElementById('tweet-feed');
    let allItems = [];

    const promises = sources.map(source => 
        fetch(convertToApi(source.url))
            .then(res => res.json())
            .then(data => {
                if(data.items) {
                    return data.items.map(item => ({...item, source: source.name}));
                }
                return [];
            })
            .catch(err => {
                console.error(`源失效: ${source.name}`, err);
                return [];
            })
    );

    const results = await Promise.all(promises);
    results.forEach(items => allItems = allItems.concat(items));
    
    // 按时间倒序
    allItems.sort((a, b) => new Date(b.pubDate) - new Date(a.pubDate));

    // 只显示最新的 15 条情报，避免页面太长
    const recentIntel = allItems.slice(0, 15);

    if (recentIntel.length === 0) {
        feedContainer.innerHTML = "<div class='error'>所有链路均已离线。</div>";
        return;
    }

    feedContainer.innerHTML = recentIntel.map(item => {
        // 处理时间
        const date = new Date(item.pubDate);
        const timeStr = date.toLocaleTimeString('en-US', { hour12: false }) + ' PST';
        
        // 提取第一张图片作为缩略图 (如果有)
        const imgMatch = item.content.match(/<img[^>]+src="([^">]+)"/);
        const imgTag = imgMatch ? `<div class="intel-img-box"><img src="${imgMatch[1]}" class="intel-img"></div>` : '';

        // 清理摘要里的 HTML 标签
        let cleanDesc = item.description.replace(/<[^>]+>/g, '').substring(0, 120) + '...';

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
    
    // 隐藏系统日志
    setTimeout(() => {
        document.getElementById('system-log').style.display = 'none';
    }, 1500);
}

fetchIntel();
</script>

<style>
/* 工业风 CSS 2.0 */
#monitor-terminal {
    background: #0b1120;
    border: 1px solid #334155;
    padding: 20px;
    font-family: 'Consolas', 'Monaco', monospace;
    position: relative;
    overflow: hidden;
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

/* 情报卡片 */
.intel-card {
    background: #1e293b;
    border-left: 3px solid #475569;
    margin-bottom: 15px;
    padding: 12px;
    transition: all 0.2s;
}
.intel-card:hover {
    border-left-color: #fdba74; /* 悬停橙色 */
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
.intel-img-box { flex: 0 0 100px; height: 60px; overflow: hidden; border: 1px solid #475569; }
.intel-img { width: 100%; height: 100%; object-fit: cover; }
.intel-desc { font-size: 0.9rem; color: #cbd5e1; line-height: 1.4; }

@media (max-width: 600px) {
    .intel-meta { flex-direction: column; }
    .intel-img-box { flex: 0 0 auto; height: 120px; }
}
</style>