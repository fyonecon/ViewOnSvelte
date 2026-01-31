<script>

    import func from "../common/func.svelte.js";
    import {afterNavigate} from "$app/navigation";
    import {browser_ok, runtime_ok} from "../common/middleware.svelte.js";

    const tab_items = [ // tab数据
        {
            icon: '1',
            title: "PureHome",
            url: "./home",
        },
        {
            icon: '1',
            title: "Bookmark",
            url: "./bookmark",
        },
    ];
    let tab_width = $state(205);

    // 本页面函数：Svelte的HTML组件onXXX=中正确调用：={()=>def.xxx()}
    const def = {
        open_url: function (href=""){
            func.open_url(href);
        },
        calc_tab: function (){ // 计算tab的实际宽度
            let len = tab_items.length;
            tab_width = (85 + 10) * len + 10;
        },
    };


    // 刷新页面数据
    afterNavigate(() => {
        if (!runtime_ok() || !browser_ok()){return;} // 系统基础条件检测
        //
        def.calc_tab();
    });

</script>

<div class="select-none">

    <svg style="display: none">
        <filter id="glass-distortion" x="0%" y="0%" width="100%" height="100%" filterUnits="objectBoundingBox">
            <feturbulence type="fractalNoise" baseFrequency="0.001 0.005" numOctaves="1" seed="17" result="turbulence"></feturbulence>

            <fecomponenttransfer in="turbulence" result="mapped">
                <fefuncr type="gamma" amplitude="1" exponent="10" offset="0.5"></fefuncr>
                <fefuncg type="gamma" amplitude="0" exponent="1" offset="0"></fefuncg>
                <fefuncb type="gamma" amplitude="0" exponent="1" offset="0.5"></fefuncb>
            </fecomponenttransfer>

            <fegaussianblur in="turbulence" stdDeviation="3" result="softMap"></fegaussianblur>

            <fespecularlighting in="softMap" surfaceScale="5" specularConstant="1" specularExponent="100" lighting-color="white" result="specLight">
                <fepointlight x="-200" y="-200" z="300"></fepointlight>
            </fespecularlighting>

            <fecomposite in="specLight" operator="arithmetic" k1="0" k2="1" k3="1" k4="0" result="litImage"></fecomposite>

            <fedisplacementmap in="SourceGraphic" in2="softMap" scale="200" xChannelSelector="R" yChannelSelector="G"></fedisplacementmap>
        </filter>
    </svg>

    <div class="div-tab-box pywebview-drag-region can-drag" style="width: {tab_width}px;">
        <div class="liquidGlass-wrapper div-tab-menu">
            <div class="liquidGlass-effect"></div>
            <div class="liquidGlass-tint"></div>
            <div class="liquidGlass-shine"></div>
            <div class="liquidGlass-text div-tab-li">
                {#each tab_items as item}
                    <button class="div-tab-item click select-none" onclick={()=>def.open_url(item.url)} >
                        <div class="font-title">{item.icon}</div>
                        <div class="font-text">{item.title}</div>
                    </button>
                {/each}
                <div class="clear"></div>
            </div>
        </div>
    </div>

</div>

<style>
    .liquidGlass-wrapper {
        position: relative;
        display: flex;
        overflow: hidden;
        cursor: pointer;
        transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 2.2);
    }
    .liquidGlass-effect {
        position: absolute;
        z-index: 0;
        inset: 0;
        backdrop-filter: blur(3px);
        filter: url(#glass-distortion);
        overflow: hidden;
        isolation: isolate;
    }
    .liquidGlass-tint {
        z-index: 1;
        position: absolute;
        inset: 0;
        background: rgba(255, 255, 255, 0.2);
        border-radius: 10px;
    }
    .liquidGlass-shine {
        position: absolute;
        inset: 0;
        z-index: 2;
        overflow: hidden;
        border-radius: 10px;
    }
    .liquidGlass-text {
        z-index: 3;
    }
    .div-tab-menu{
        padding: 5px 5px;
        border-radius: 10px;
    }

    .div-tab-box {
        position: fixed;
        bottom: 5px;
        left: 0;
        right: 0;
        margin: 0 auto;
        width: calc(202px);
        z-index: 1;
    }
    .div-tab-item{
        width: 85px;
        margin: 0 5px;
        padding: 5px 5px;
        overflow: hidden;
        text-align: center;
        display: inline-block;
        border: none;
        outline: none;
        float: left;
        border-radius: 10px;
        transition: all 0.1s ease-in;
    }

    /*.div-tab-item-active{*/
    /*    !*background-color: rgba(69,141,247, 1) !important;*!*/
    /*    !*color: rgba(220,220,220, 1);*!*/
    /*    !*box-shadow: inset -2px -2px 2px rgba(0, 0, 0, 0.1);*!*/
    /*    backdrop-filter: blur(2px);*/
    /*    !*color: rgba(69,141,247, 1);*!*/
    /*    color: orange;*/
    /*}*/

</style>