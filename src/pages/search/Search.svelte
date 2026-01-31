<script lang="ts">
    import { resolve } from '$app/paths';
    import func from "../../common/func.svelte.js";
    import {afterNavigate} from "$app/navigation";
    import {onMount} from "svelte";
    import {browser_ok, runtime_ok} from "../../common/middleware.svelte";
    import config from "../../config";
    import search_engines_dict from "../../common/search_engines";
    import {browser} from "$app/environment";


    // 本页面参数
    let route = $state(func.get_route());


    // 本页面函数：Svelte的HTML组件onXXX=中正确调用：={()=>def.xxx()}
    const def = {
        check_white_word: function(word = ""){
            let that = this;
            let back_state = false;
            // 白名单跳转
            if (word === "@home"){
                back_state = true;
                that.open_url("./home?cache="+func.js_rand(10000, 99999));
            }
            else if (word === "@info"){
                back_state = true;
                that.open_url("./info");
            }
            else if (word === "@404"){
                back_state = true;
                that.open_url("./_404");
            }
            // 搜索引擎
            else if (word === "@bing"){
                back_state = true;
            }
            //
            else if (word === "@bing"){
                back_state = true;
            }
            else if (word === "@baidu"){
                back_state = true;
            }
            else if (word === "@sogou"){
                back_state = true;
            }
            else if (word === "@google"){
                back_state = true;
            }
            else if (word === "@yahoo"){
                back_state = true;
            }
            else if (word === "@yandex"){
                back_state = true;
            }

            //
            return back_state;
        },
        open_url: function(href=""){
            if (browser){
                window.location.replace(href);
            }else{
                func.open_url(href, "_self");
            }
        },
        check_param: function(){
            let that = this;
            //
            let word = func.search_href_param("", "word").trim();
            let engine = func.search_href_param("", "engine").trim();
            let url_timeout = func.search_href_param("", "url_timeout").trim();
            //
            if (url_timeout){
                func.loading_show();
                if (func.url_timeout_decode("search", url_timeout)){
                    // 是url链接就直接打开
                    if (func.is_url(word)){
                        that.open_url(word);
                        return
                    }
                    // word白名单级校验
                    if (!that.check_white_word(word)){ // 正常打开关键词
                        if (!search_engines_dict[engine]){engine = "bing";}
                        let href = search_engines_dict[engine].url+encodeURIComponent(word);
                        that.open_url(href);
                        return;
                    }
                }else{ // 过期
                    func.open_url_404("./", func.get_translate("url_timeout"), func.get_href());
                }
            }
        },
    };


    // 页面函数执行的入口，实时更新数据
    function page_start(){
        console.log("page_start()=", route);
        // 开始
        def.check_param();
    }


    // 检测$state()值变化
    $effect(() => {
        //
    });


    // 刷新页面数据
    afterNavigate(() => {
        if (!runtime_ok() || !browser_ok()){return;} // 系统基础条件检测
        //
        page_start();
    });


    // 页面装载完成后，只运行一次
    onMount(() => {
        //
    });


</script>

<div class="hide">
    <h3>{route}</h3>
    <ul class="ul-group font-text">
        <li class="li-group">
            <div class="li-group-title break">
                test
            </div>
            <div class="li-group-content select-text">
                123
            </div>
        </li>

    </ul>
</div>