---
title: "情报网"
slug: "monitor"
description: "接入全球游戏资讯加密数据流。"
image: "hoshino1.jpg"
toc: false
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

    <div id="tweet-feed" class="feed-container"></div>
</div>

<script src="https://unpkg.com/rss-parser/dist/rss-parser.min.js"></script>

<script>
document.addEventListener("DOMContentLoaded", function() {
    // ================= 情报源配置 =================
    const sources = [
        { name: 'GEMATSU', url: 'https://gematsu.com/feed' },
        { name: 'SILICONERA', url: 'https://www.siliconera.com/feed/' },
        { name: 'EUROGAMER', url: 'https://www.eurogamer.net/?format=rss' }
    ];

    const convertToApi = (url) => {
        return `https://api.rss2json.com/v1/api.json?rss_url=${encodeURIComponent(url)}`; 
    };

    async function fetchIntel() {
        const feedContainer = document.getElementById('tweet-feed');
        const logContainer = document.getElementById('system-log');
        
        // 安全检查：防止页面还没画好脚本就跑了
        if (!feedContainer || !logContainer) return;

        let allItems = [];
        let logHtml = logContainer.innerHTML;

        console.log("正在初始化情报链路..."); 

        const promises = sources.map(source => 
            fetch(convertToApi(source.url))
                .then(res => {
                    if (!res.ok) throw new Error('Network response was not ok');
                    return res.json();
                })
                .then(data => {
                    if(data.status === 'ok' && data.items) {
                        logHtml += `> [${source.name}] CONNECTION ESTABLISHED.<br>`;
                        logContainer.innerHTML = logHtml;
                        return data.items.map(item => ({...item, source: source.name}));
                    } else {
                        return [];
                    }
                })
                .catch(err => {
                    console.error(`[${source.name}] 链接失败:`, err);
                    logHtml += `<span style="color:#ef4444">> ERROR: ${source.name} CONNECTION LOST</span><br>`;
                    logContainer.innerHTML = logHtml;
                    return [];
                })
        );

        const results = await Promise.all(promises);
        results.forEach(items => allItems = allItems.concat(items));
        
        // 如果没有抓到任何数据
        if (allItems.length === 0) {
            feedContainer.innerHTML = `
                <div class='intel-card' style='border-left-color: #ef4444'>
                    <div class='intel-body'>
                        <div class='intel-title' style='color:#ef4444'>无法建立数据流连接</div>
                        <div class='intel-desc'>
                            SYSTEM ERROR: NO DATA RECEIVED.<br>
                            请检查浏览器是否开启了广告拦截 (AdBlock)，或者等待 API 配额重置。
                        </div>
                    </div>
                </div>`;
            // 也要隐藏日志，不然卡住很难看
            setTimeout(() => { logContainer.style.display = 'none'; }, 2000);
            return;
        }

        // 排序与截取
        allItems.sort((a, b) => new Date(b.pubDate) - new Date(a.pubDate));
        const recentIntel = allItems.slice(0, 20);

        // 渲染
        feedContainer.innerHTML = recentIntel.map(item => {
            const date = new Date(item.pubDate);
            const timeStr = date.toLocaleTimeString('en-US', { hour12: false });
            
            const content = item.content || item.description || "";
            const imgMatch = content.match(/<img[^>]+src="([^">]+)"/);
            const imgTag = imgMatch ? `<div class="intel-img-box"><img src="${imgMatch[1]}" class="intel-img" loading="lazy"></div>` : '';
            let cleanDesc = (item.description || "").replace(/<[^>]+>/g, '').substring(0, 100) + '...';

            return `
                <div class="intel-card">
                    <div class="intel-header">
                        <span class="source-tag">[${item.source}]</span>
                        <span class="time-tag">${timeStr}</span>
                    </div>
                    <div class="intel-body">
                        <a href="${item.link}" target="_blank" class="intel-title">${item.title}</a>
                        <div class="intel-meta">${imgTag}<div class="intel-desc">${cleanDesc}</div></div>
                    </div>
                </div>
            `;
        }).join('');
        
        // 1.5秒后隐藏启动日志
        setTimeout(() => {
            logContainer.style.display = 'none';
        }, 1500);
    }

    fetchIntel();
});
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

.intel-card {
    background: #1e293b;
    border-left: 3px solid #475569;
    margin-bottom: 15px;
    padding: 12px;
    transition: all 0.2s;
}
.intel-card:hover {
    border-left-color: #fdba74;
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