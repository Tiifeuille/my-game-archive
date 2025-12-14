---
title: "情报网 "
slug: "monitor"
description: "网络链路故障排查中..."
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
        <span class="status-dot" id="status-light"></span> 
        STATUS: <span id="status-text">CONNECTING...</span>
    </div>
    
    <div id="system-log">
        > SYSTEM BOOT SEQUENCE INITIATED...<br>
        > WAITING FOR DATA STREAM...<br>
    </div>

    <div id="tweet-feed" class="feed-container"></div>
</div>

<script>
document.addEventListener("DOMContentLoaded", function() {
    const feedContainer = document.getElementById('tweet-feed');
    const logContainer = document.getElementById('system-log');
    const statusText = document.getElementById('status-text');
    const statusLight = document.getElementById('status-light');

    // 备用方案：只用这一个最稳的源测试
    const testUrl = 'https://api.rss2json.com/v1/api.json?rss_url=https%3A%2F%2Fgematsu.com%2Ffeed';

    function log(msg, isError = false) {
        const color = isError ? '#ef4444' : '#64748b';
        logContainer.innerHTML += `<span style="color:${color}">> ${msg}</span><br>`;
    }

    log("正在尝试建立连接...");

    // 设置一个 8 秒的超时计时器
    const timeoutTimer = setTimeout(() => {
        if(feedContainer.innerHTML === "") {
            handleError("连接超时 (TIMEOUT)。服务器 8 秒内无响应。");
        }
    }, 8000);

    fetch(testUrl)
        .then(res => {
            if (!res.ok) throw new Error(`HTTP Error: ${res.status}`);
            return res.json();
        })
        .then(data => {
            clearTimeout(timeoutTimer); // 清除超时报警

            if(data.status === 'ok') {
                log("连接成功！开始渲染数据...", false);
                statusText.innerText = "ONLINE";
                statusText.style.color = "#4ade80";
                statusLight.style.backgroundColor = "#4ade80";
                statusLight.style.boxShadow = "0 0 8px #4ade80";
                
                // 渲染测试数据
                const items = data.items.slice(0, 5); // 只取前5条测试
                feedContainer.innerHTML = items.map(item => `
                    <div class="intel-card">
                        <div class="intel-title"><a href="${item.link}" target="_blank" style="color:#e2e8f0">${item.title}</a></div>
                        <div class="intel-desc" style="color:#94a3b8; font-size:0.8rem">${item.pubDate}</div>
                    </div>
                `).join('');
                
                log("渲染完成。");
            } else {
                handleError("API 响应错误: " + data.message);
            }
        })
        .catch(err => {
            clearTimeout(timeoutTimer);
            handleError(err.message);
        });

    function handleError(msg) {
        statusText.innerText = "OFFLINE";
        statusText.style.color = "#ef4444";
        statusLight.style.backgroundColor = "#ef4444";
        statusLight.style.animation = "none";
        
        log("CRITICAL ERROR DETECTED:", true);
        log(msg, true);
        
        // 给出针对性建议
        let advice = "未知错误";
        if (msg.includes("Failed to fetch")) {
            advice = "请求被拦截。请检查【广告拦截插件】或【网络代理】。";
        } else if (msg.includes("TIMEOUT")) {
            advice = "网络连接超时。请检查网络是否通畅。";
        } else if (msg.includes("429")) {
            advice = "API 请求次数过多，请一小时后再试。";
        }

        feedContainer.innerHTML = `
            <div style="padding:20px; border:1px solid #ef4444; background:#450a0a; color:#fca5a5; margin-top:20px;">
                <strong>故障诊断报告:</strong><br>
                ${msg}<br><br>
                <strong>建议操作:</strong><br>
                ${advice}
            </div>
        `;
    }
});
</script>

<style>
#monitor-terminal {
    background: #0b1120;
    border: 1px solid #334155;
    padding: 20px;
    font-family: 'Consolas', monospace;
    min-height: 400px;
}
.monitor-header {
    border-bottom: 2px solid #334155;
    padding-bottom: 10px;
    margin-bottom: 15px;
    color: #e2e8f0;
    font-weight: bold;
}
.status-dot {
    display: inline-block; width: 10px; height: 10px;
    background: #fbbf24; border-radius: 50%; /* 默认黄色 */
    margin-right: 8px;
}
.intel-card {
    background: #1e293b;
    border-left: 3px solid #64748b;
    margin-bottom: 10px;
    padding: 10px;
}
</style>