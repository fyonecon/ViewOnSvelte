<script lang="ts">
    import { resolve } from '$app/paths';
    import func from "../../common/func.svelte.js";
    import {afterNavigate} from "$app/navigation";
    import {onMount} from "svelte";
    import config from "../../config";
    import search_engines_dict from "../../common/search_engines";
    import {browser_ok, runtime_ok} from "../../common/middleware.svelte";
    import {Dialog, Portal} from "@skeletonlabs/skeleton-svelte";
    import {browser} from "$app/environment";

    // 本页面参数
    const animation = 'transition transition-discrete opacity-0 translate-y-[100px] starting:data-[state=open]:opacity-0 starting:data-[state=open]:translate-y-[100px] data-[state=open]:opacity-100 data-[state=open]:translate-y-0';
    let route = $state(func.get_route());
    let input_value_search = $state("");
    const search_selected_key = config.app.app_class + "search_selected";
    const search_history_key = search_selected_key+"_history";
    const search_history_split = "#@#"+search_history_key+"#@#";
    const search_history_max_len = 30;

    let search_engines_array: object[] = $state([]);
    let search_history_array: string[] = $state([]);
    let del_input_history_dialog_is_open = $state(false);
    let input_ele: any; // input标签对象
    let open_url_loading_timer = $state(0);

    // 本页面函数：Svelte的HTML组件onXXX=中正确调用：={()=>def.xxx()}
    const def = {
        close_dialog: function(){
            del_input_history_dialog_is_open = false;
        },
        open_dialog: function(){
            del_input_history_dialog_is_open = true;
        },
        create_select: function(){
            //
            search_engines_array = [
                {
                    name: "Bing",
                    value: "bing",
                    url: "https://www.bing.com/?ensearch=1&q=",
                    selected: "selected",
                }
            ];
            //
            func.get_db_data(search_selected_key).then(value => {
                if (!value){value = "bing";}
                //
                let the_dict = search_engines_dict[value];
                if (the_dict){ // 读取到已设置的引擎
                    //
                    let the_name = the_dict.name;
                    let the_url = the_dict.url;
                    // 渲染列表
                    search_engines_array = []; // init
                    Object.keys(search_engines_dict).forEach(key => {
                        let info = search_engines_dict[key];
                        let selected = value===key?"selected":"";
                        search_engines_array.push({
                            name: info.name,
                            value: key,
                            url: info.url,
                            selected: selected,
                        });
                    });
                }else{
                    func.notice("DB Error");
                }
            });
        },
        update_select: function(event: any){
            let that = this;
            //
            let value = event.target.value;
            func.set_db_data(search_selected_key, value).then(v=>{
                func.console_log("update_select=", value, v);
                // that.create_select();
            });
        },
        filter_array: function(value=""){ // 数组去重
            return value;
        },
        show_history: function(value=""){ // 显示历史
            console.log("show=", value)
            search_history_array = [];
            if (value.length>0 && value.indexOf(search_history_split) !== -1){
                let array = value.split(search_history_split);
                for (let i=0; i<array.length; i++){
                    let the_value = array[i];
                    if (the_value.trim()){
                        search_history_array.push(the_value);
                    }
                }
            }else if (value.length>0 && value.indexOf(search_history_split) === -1){
                search_history_array.push(value);
            }
        },
        input_history: function(_value=""){ // 更新与显示
            let that = this;
            //
            return new Promise(resolve1 => {
                func.get_db_data(search_history_key).then(value => {
                    if (_value.trim().length>0){
                        if (value){
                            value = value + search_history_split + _value;
                        }else{
                            value = search_history_split + _value;
                        }
                        // 数组去重
                        let new_value = that.filter_array(value);
                        func.set_db_data(search_history_key, new_value).then(_v=>{
                            that.show_history(new_value);
                            resolve1(new_value);
                        });
                    }else{
                        that.show_history(value);
                    }
                });
            });
        },
        input_enter: function(event: any){
            let that = this;
            //
            if (event.key === 'Enter') {
                console.log('Enter pressed:', input_value_search);
                // 执行回车操作
                that.input_run_search();
            }
        },
        input_run_search: function(){
            let that = this;
            //
            let the_value = input_value_search.trim();
            if (the_value){
                clearTimeout(open_url_loading_timer);
                func.loading_show();
                //
                that.input_history(the_value).then(v=>{
                    that.input_auto_write("");
                    func.get_db_data(search_selected_key).then(value => {
                        open_url_loading_timer = setTimeout(function (){
                            func.loading_hide();
                        }, 1500);
                        //
                        if (!value) {value = "bing";}
                        //
                        let href = "./search?word="+encodeURIComponent(the_value)+"&engine="+value+"&url_timeout="+func.url_timeout_encode("search", 6*60*60)+"&ap=ipt";
                        if (browser){
                            window.open(href, "_blank");
                        }else{
                            func.open_url_with_default_browser(href);
                        }
                    });
                });
            }else{
                input_ele.focus();
                func.notice(func.get_translate("input_null"));
            }
        },
        input_auto_write: function(value=""){ // 自动填充input
            input_value_search = value;
        },
        input_clear_write: function(){ // 清空输入框
            let that = this;
            //
            that.input_auto_write("");
            input_ele.focus();
        },
        input_del_history: function(){
            func.loading_show();
            func.del_db_data(search_selected_key).then(state=>{
                del_input_history_dialog_is_open = false;
                func.loading_hide();
            });
        },
    };


    // 页面函数执行的入口，实时更新数据
    function page_start(){
        console.log("page_start()=", route);
        // 开始
        def.create_select();
        def.input_history("")
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

<div>
    <div class="search-div-input font-text">
        <label class="label">
            <select onchange={(event)=>def.update_select(event)}>
                {#each search_engines_array as option_dict}
                    <option value="{option_dict.value}" selected="{option_dict.selected}">{option_dict.name}</option>
                {/each}
            </select>
        </label>
        <label class="label">
            <input class=" input-style w-full border-radius font-text select-text" type="search" maxlength="1200" placeholder="{func.get_translate('input_placeholder_search')}"
                   bind:value={input_value_search}
                   onkeydown={(event)=>def.input_enter(event)}
                   onmouseenter={(e) => e.currentTarget.focus()}
                   bind:this={input_ele}
            />
        </label>
    </div>
    <div class="search-div-btn font-title">
        <button onclick={()=>def.open_dialog()}>删除历史</button>
        <button onclick={()=>def.input_clear_write()}>重新输入</button>
        <button onclick={()=>def.input_run_search()}>搜 索</button>
    </div>
    <div class="search-div-history font-text font-blue">
        {#each search_history_array as history_value, index}
            <button class="history-btn break break-ellipsis" onclick={()=>def.input_auto_write(history_value)} title="{history_value}">{"#"+(index+1)+" "+history_value + " "}</button>
        {/each}
    </div>
</div>


<!-- 删除已设置的本地文件夹 -->
<Dialog closeOnInteractOutside={false} closeOnEscape={false} open={del_input_history_dialog_is_open} onOpenChange={()=>{}}>
    <Portal>
        <Dialog.Backdrop class="fixed inset-0 z-50 bg-surface-50-950/80  select-none" />
        <Dialog.Positioner class="fixed inset-0 z-50 flex justify-center items-center font-text select-none">
            <Dialog.Content class="card bg-neutral-100 dark:bg-neutral-800 w-full max-w-xs p-4 space-y-4 shadow-xl {animation}  px-[10px] py-[10px] border-radius">
                <header class="flex justify-between items-center pywebview-drag-region can-drag">
                    <Dialog.Title class="font-title font-bold">⚠️</Dialog.Title>
                </header>
                <Dialog.Description class="font-text select-text">
                    {@html func.get_translate('remove_help_2')}
                </Dialog.Description>
                <footer class="flex justify-center gap-10 select-none  px-[10px] py-[10px]">
                    <button title="Cancel" class="btn btn-base preset-tonal font-title" onclick={()=>def.close_dialog()}>{func.get_translate("btn_cancel")}</button>
                    <button title="Update" type="button" class="btn btn-base preset-filled-primary-500 font-title" onclick={()=>def.input_del_history()}>{func.get_translate("remove")}</button>
                </footer>
            </Dialog.Content>
        </Dialog.Positioner>
    </Portal>
</Dialog>

<style>
    .search-div-input{
        width: 100%;
        height: 60px;
    }
    .search-div-btn{
        width: 100%;
        height: 40px;
    }
    .search-div-btn>button{
        width: calc(100%/3 - 4px);
        float: left;
    }
    .search-div-history{
        width: 100%;
        min-height: 60px;
        max-height: 300px;
    }
    .history-btn{
        margin: 3px 10px;
        max-width: 220px;
        overflow: hidden;
        float: left;
    }
</style>