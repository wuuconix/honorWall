<template>
    <div class="loginout">
        <a href="https://github.com/wuuconix/honorWall" target="_blank" title="Github仓库">
            <img src="https://tvax3.sinaimg.cn/large/007YVyKcly1h27w6nxmvqj305k05kaa3.jpg">
        </a>
        <span title="自信地没有用任何UI库，手写布局！"> 武丑兄荣誉墙 </span>
        <button v-if="!online" @click="authenticate = true" class="login" title="登录后可见武丑兄的荣誉整证书!">Login</button>
        <button v-if="online" @click="logout" class="logout">Logout</button>
    </div>
    <div class="imgs_wrapper">
        <img v-for="img in imgs" :src="img['url']" @click="handle_click(img['url'])">
    </div>
    <transition name="fade">
        <div class="shadow" v-show="preview" @click="preview=false">
            <img :src="previewSrc" class="preview">
        </div>
    </transition>
    <transition name="fade">
        <div class="shadow auth" v-show="authenticate" @click="handle_click2" ref="shadow2">
            <input type="text" placeholder="user" v-model="user">
            <input type="password" placeholder="pass" v-model="pass">
            <button @click="login">Login</button>
            <p>{{ response }}</p>
        </div>
    </transition>
</template>

<script>

export default {
    data() {
        return {
            imgs: [],
            preview: false,
            previewSrc: "",
            authenticate: false,
            user: "",
            pass: "",
            online: false,
            response: ""
        }
    },
    methods: {
        get_imgs() {
            fetch("https://wuuconix.link/api/honor/imgs", {
                headers: {authorization: localStorage.getItem("token")}
            }).then(res => res.json()).then(res => {
                if (res.group == "guest") {
                    this.online = false
                } else if (res.group == "admin") {
                    this.online = true
                }
                this.imgs = res.imgs
            })
        },
        handle_click(src) {
            this.previewSrc = src
            this.preview = true
        },
        handle_click2(e) {
            if (e.target == this.$refs.shadow2) {
                this.response = ""
                this.authenticate = false
            }
        },
        login() {
            fetch("https://wuuconix.link/api/honor/login", {
                method: "POST",
                body: new URLSearchParams({user: this.user, pass: this.pass})
            }).then(res => res.json()).then(res => {
                if (res.success) {
                    localStorage.setItem("token", res.token)
                    this.authenticate = false
                    this.get_imgs()
                    this.response = ""
                } else {
                    this.response = "账号或密码错误"
                }
            })
        },
        logout() {
            localStorage.removeItem("token")
            this.get_imgs()
        }
    },
    mounted() {
        document.addEventListener("keypress", (e) => {
            if (e.key == "Enter") {
                if (this.authenticate) {
                    this.login()
                } else {
                    this.authenticate = true
                }
            }
        })
        this.get_imgs()
    },
}

</script>

<style>
* {
    padding: 0;
    margin: 0;
}

div.imgs_wrapper {
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
    width: 100%;
    height: 100%;
}

div.imgs_wrapper img {
    flex: 30%;
    height: 40vh;
    object-fit: contain;
    padding: 30px;
    filter: drop-shadow(10px 10px 10px rgba(0,0,0,.5));
}

div.shadow {
    position: fixed;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    width: 100vw;
    height: 100vh;
    background-color: rgba(0,0,0,.5);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 100;
}

div.auth {
    flex-direction: column;
}

div.shadow img {
    width: 100%;
    height: 100%;
    object-fit: contain;
}

.fade-enter-active, .fade-leave-active {
	transition: opacity .25s
}

.fade-enter, .fade-leave-to /* .fade-leave-active in <2.1.8 */ {
	opacity: 0
}

input {
    width: 30%;
    height: 32px;
    line-height: 32px;
    font-size: 16px;
    font-family: Microsoft YaHei;
    border-radius: 5px;
    margin: 10px;
    border: 3px solid #337ecc;
    outline: none;
}

div.btn_wrapper {
    display: flex;
    justify-content: center;
    align-items: center;
}

button {
    width: 64px;
    height: 32px;
    color: white;
    margin: 10px;
    background-color: #337ecc;
    border: 0px;
    border-radius: 5px;
    outline: none;
}

div.loginout {
    position: sticky;
    top: 0;
    width: 100%;
    height: 10%;
    background-color: #337ecc;
    z-index: 99;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0 5%;
    box-sizing:border-box
}

div.loginout button.login {
    background-color:  #95d475;
}

div.loginout button.logout {
    background-color:  #f89898
}

div.loginout img {
    height: 40px;
    border-radius: 20px;
}

button:hover {
    cursor: pointer;
}

span, p {
    color: white;
}
</style>
