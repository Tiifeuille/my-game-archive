---
# 菜单定义 (确保 Home 菜单不消失)
menu:
    main:
        name: Home
        weight: 1
        params:
            icon: home
        # 如果需要，取消注释这行：
        # url: / 
--- 

{{< rawhtml >}}
<div id="fixed-bgm-player" class="main-page-bgm-player">
    <h3 class="bgm-title">ARCHIVE BGM: Wangan MT5 - Driving Into The Abyss</h3>
    <iframe 
        width="100%" 
        height="100" 
        src="https://www.youtube-nocookie.com/embed/eR2G21p3hws?controls=1&loop=1&playlist=eR2G21p3hws" 
        frameborder="0" 
        allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" 
        allowfullscreen
        title="主页背景音乐播放器">
    </iframe>
    <p class="bgm-note">（请点击上方播放，开启听觉档案层）</p>
</div>
{{< /rawhtml >}}