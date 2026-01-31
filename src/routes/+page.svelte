<script lang="ts">
    /*根路由*/

    import func from "../common/func.svelte.js";
    import config from "../config";
    import {browser} from "$app/environment";

    //
    function go_home(){
        // 重新定向 /到 /home 页面
        func.redirect_pathname({
            url_pathname: func.url_path(config.sys.base_route+config.sys.home_route),
            url_params: func.get_params(),
        });
    }

    //
    let href = func.get_href();
    let host = config.sys.base_route+"/";
    if (browser){
        host = "//"+window.location.host+config.sys.base_route+"/";
    }

    // 兼容老View框架的路由
    if (href.indexOf("route=home") != -1){
        let array = href.split("route=home");
        let url = "";
        if (array.length >= 2){
            url = host + "home?" + array[1];
        }else{
            url = host + "home";
        }
        func.open_url(url);
    }
    else if (href.indexOf("route=search") != -1){
        let array = href.split("route=search");
        let url = "";
        if (array.length >= 2){
            url = host + "search?" + array[1];
        }else{
            url = host + "search";
        }
        func.open_url(url);
    }
    else if (href.indexOf("route=info") != -1){
        let array = href.split("route=info");
        let url = "";
        if (array.length >= 2){
            url = host + "info?" + array[1];
        }else{
            url = host + "info"
        }
        func.open_url(url);
    }
    //
    else{
        go_home();
    }


</script>