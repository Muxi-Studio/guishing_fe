<template>
    <div id="header" v-hide="onclick||showTips||!headerFixed">
        <div :class="$style.top">
            <div :class="$style.icon" v-show="specialPage" v-on:click="showSpecial">
                <svg viewBox="0 0 200 200" :class="$style.img">
                    <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#classify"></use>
                </svg>
            </div>
            <div :class="$style.logo" :style="logoStyle">
                <div :class="$style.logoImg" v-on:click="backToRoot"></div>
            </div>
            <div :class="$style.profile" v-on:click="showProfile" v-show="!specialPage">
                <svg viewBox="0 0 200 200" :class="$style.img">
                    <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#user"></use>
                </svg>
            </div>
            <div :class="$style.search" v-on:click="showSearch">
                <svg viewBox="0 0 200 200" :class="$style.img">
                    <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#find"></use>
                </svg>
            </div>
        </div>
        <div :class="$style.mask" v-show="onclick" v-on:click="showSearch" :style = "styleWidth">
            <div :class="$style.searchBox" v-on:click="Search">
                <input type="text" :class="$style.input" v-model="content">
                <div :class="$style.button" v-on:click="postContent">搜索</div>
            </div>
            <div :class="$style.title_s">热门搜索</div>
            <div :class="$style.tagList" v-on:click="Search">
                <div :class="$style.tag" v-for="tag in tagList" v-on:click="getTag(tag)">#{{tag}}#</div>
            </div>
        </div>
        <div v-show="showTips" :class="$style.suggestMask" :style = "styleWidth">
            <div :class="$style.returnCard">
                <div :class="$style.returnContent">您还没有登录哦~</div>
                <div :class="$style.returnButton" v-on:click="admin">我要登录</div>
                <div :class="$style.returnButton" v-on:click="quit">取消</div>
            </div>
        </div>
    </div>
</template>
<script>
import Hidden from '../directives/hidden'
import {
    bus
} from '../bus.js'
import Cookie from '../common/cookie.js'
import FETCH from '../common/fetch.js'

export default {
    data() {
        return {
            onclick: false,
            content: "",
            tagList: [],
            showTips: false,
            specialPage: false,
            headerFixed: true,
            phone:false
        }
    },
    directives: {
        hide: Hidden
    },
    mounted() {
        if(window.screen.availWidth > 500)
            this.phone = false
        if (window.location.pathname == '/search') {
            this.onclick = true
            FETCH.FetchData("/api/v1.0/hottag/", "GET").then(value => {
                this.tagList = value.hot_tag
            })
        }
        else if (window.location.pathname == "/special") {
            this.specialPage = true
        }
        bus.$on('headerFix', this.fixHead)
    },
    computed: {
        logoStyle: function () {
            return {
                marginLeft: this.specialPage ? '25%' : '12.5px'
            }
        },
        styleWidth:function(){
            return{
                width:this.phone? '':'400px'
            }
        }
    },
    methods: {
        showSpecial() {
            bus.$emit('showSpecialPage')
            this.fixHead()
        },
        fixHead() {
            if (this.headerFixed)
                this.headerFixed = false
            else {
                this.headerFixed = true
            }
        },
        backToRoot() {
            window.location = "/"
        },
        getTag(e) {
            this.content = e
        },
        Search(e) {
            e.stopPropagation()
        },
        quit() {
            this.showTips = false
        },
        admin() {
            this.quit()
            Cookie.setCookie('url', window.location.href)
            
            window.location = "https://user.muxixyz.com?landing=gs.muxixyz.com/landing"
            // window.location = "https://user.muxixyz.com?landing=localhost:3000/landing"
        },
        showSearch(e) {
            if (window.location.pathname !== '/search') {
                window.location = '/search'
                this.onclick = true
            } else {
                console.log("hah")
            }
            FETCH.FetchData("/api/v1.0/hottag/", "GET").then(value => {
                this.tagList = value.hot_tag
            })
            if (this.onclick) {
                if (window.location.pathname == '/search')
                    window.history.back()
                this.onclick = false
                this.content = ""
            } else {
                this.onclick = true
            }
        },
        showProfile() {
            if (!Cookie.getCookie("token")) {
                console.log("Cookie.getCookie(token)", Cookie.getCookie("token"))
                this.showTips = true
            } else {
                window.location = "/profile/" + Cookie.getCookie("uid")
            }
        },
        postContent() {
            FETCH.FetchData('/api/v1.0/feed/', "POST", {
                content: this.content
            }).then(json => {
                this.onclick = false
                this.content = ""
                bus.$emit('search', json)
            })
        }
    }
}
</script>
<style lang ="sass" module>
@import '../scss/color.scss';
@import '../scss/zindex.scss';

.top {
    z-index: $Zindex3;
    position: relative;  
    background-color: $white;
    height: 54px;
     width:100%; 
    composes: space from 'sass-loader!../scss/utility.scss';
}

.suggestMask {
    position: fixed;
    top: 0;
    bottom: 0;
    width: 100%;
    z-index: $Zindex3;
    background-color: rgba(51, 51, 51, 0.85);
}

.returnCard {
    position: absolute;
    top: 50%;
    left: 50%;
    width: 217px;
    height: 87.5px;
    background-color: $white;
    color: $black;
    padding: 25px 17.5px 15px;
    border-radius: 2px;
    transform: translate(-50%, -50%);
    box-sizing: border-box;
    font-size: 14px;
}

.returnButton {
    float: right;
    margin-left: 25px;
    color: $orange;
    cursor: pointer;
    margin-top: 16px;
}

.common {
    line-height: 54px;
    height: 54px;
}

.logo {
    margin-left: 12.5px;
    font-family: "Adobe 黑体 Std";
    color: $orange;
    font-size: 25px;
    composes: common;
    composes: horizon from 'sass-loader!../scss/utility.scss'; 
}

.icon{
    composes: common;
    composes: horizon from 'sass-loader!../scss/utility.scss';
    margin-left:4%;
}

.search {
    float: right; 
    margin-right: 26px;
    composes: common;
    vertical-align:middle;
}

.profile {
     float: right; 
    margin-right: 17.5px;
    composes: common;
    vertical-align:middle;
}

.img {
    vertical-align: middle;
    width: 19px;
}

.logoImg {
    vertical-align: middle;
    height: 32px;
    width: 95px;
    margin-top: 11px;
    background-size: 100%;
    background-image: url('../img/hdgslogo.png');
}

.mask {
    position: fixed;
    top: 54px;
    z-index: $Zindex2;
    height: 100%;
    width: 100%;
    background-color: rgba(229, 233, 233, 0.85);
}

.searchBox {
    margin: 16px 15px;
    composes: space from 'sass-loader!../scss/utility.scss';
    border: $orange 1px solid;
}

.input {
    padding-left: 10px;
    padding-right: 68px;
    height: 31px;
    width: 80%;
    border: none;
    box-sizing: border-box;
    composes: horizon from 'sass-loader!../scss/utility.scss';
}

.button {
    width: 20%;
    float: right;
    box-sizing: border-box;
    font-size: 16px;
    background-color: $orange_button;
    color: $white;
    padding: 4.5px 16px;
    composes: horizon from 'sass-loader!../scss/utility.scss';
}

.title_s {
    margin: 20px;
}

.tagList {
    margin-left: 20px;
    margin-right: 20px;
    display: justify;
}

.tag {
    font-size: 16px;
    background-color: $orange_button;
    padding: 5.5px 12.5px;
    color: $white;
    margin: 10px;
    display: inline-block;
    vertical-align: middle;
}
</style>
