<script lang="ts">
	import './layout.css'; // 全局CSS
	import { onMount, onDestroy } from 'svelte';
	import { page } from '$app/state';
	import func from "../common/func.svelte.js";
	import { afterNavigate, beforeNavigate } from "$app/navigation";
    import Loading from "../parts/Loading.svelte";
    import Notice from "../parts/Notice.svelte";
    import Alert from "../parts/Alert.svelte";
    import config from "../config";
    import {browser_ok, runtime_ok} from "../common/middleware.svelte";


    /** @type {{children: import('svelte').Snippet}} */
    let { children } = $props();


    // 本页面参数
    let page_display = $state("hide");
    let theme_model = $state("");
    let lang_index = $state("");


    // 本页面函数
    const def = {
        watch_404_route: function() { // 重定向到自定义的404页面
            if (page.status === 404) {
                func.redirect_pathname({
                    url_pathname: func.url_path("/_404"),
                    url_params: "?error_url=" + encodeURIComponent(func.get_href()) + "&error_msg=404 Route"
                });
            }
        },
    };


    // 监控所有$state()值变化
    $effect(() => {
        // console.log("layout=effect=", page.route);
    });


	// 路由变化之前
	beforeNavigate(() => {
		//
	});


	// 路由变化之后
	afterNavigate(() => {
        // 系统基础条件检测
        if (!runtime_ok()){ // false
            func.alert_msg(func.get_translate("runtime_error_alert"), "long");
            page_display="hide";
            return
        }else{ // 附加条件检测
            if (!browser_ok()){ // false
                func.alert_msg(func.get_translate("runtime_cn_chat_alert"), "long");
                page_display="hide";
                return
            }else{ // ok
                page_display="show";
                //
                def.watch_404_route(); // 检测路由变化
            }
        }
	});


    // 页面装载完成后，只运行一次
    onMount(() => {
        if (!runtime_ok() || !browser_ok()){return;} // 系统基础条件检测
        //
        let theme_event = window.matchMedia('(prefers-color-scheme: dark)');
        theme_event.addEventListener('change', function (event){ // 监测主题变化
            //
        });
        //
    });


    //
    onDestroy(() => {
        //
    });


</script>

<div class="app {page_display} select-none" data-theme_model="{theme_model}" data-language_index="{lang_index}">
    <!-- 内容 -->
	<main class="main {page_display} ">{@render children()}</main>
</div>

<div class="alert select-none">
    <!--  全局交互组件  -->
    <Loading />
    <Notice />
    <Alert />
</div>