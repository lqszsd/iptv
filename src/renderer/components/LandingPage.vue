<template>
  <div id="wrapper">
    <main>
      <el-container>
        <el-header style="padding:0px;">
          <el-button>IPTV</el-button>
        </el-header>

        <el-container>
          <el-aside width="200px" style="margin:20px 0; max-height:420px">
            <el-menu
              default-active="1"
              menu-trigger="click"
              class="el-menu-vertical-demo"
              @open="handleOpen"
              @close="handleClose"
            >
              <el-submenu index="1">
                <template slot="title">
                  <i class="el-icon-menu"></i>
                  <span>中国大陆</span>
                </template>
                <el-menu-item-group>
                  <el-menu-item
                    v-for="item in options"
                    @click="selectTv(item.value)"
                    :key="item.id"
                    :index="item.id"
                    style="padding: 0 20px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;"
                  >{{item.value}}</el-menu-item>
                </el-menu-item-group>
              </el-submenu>
            </el-menu>
          </el-aside>
          <el-main>
            <div id="dplayer"></div>
          </el-main>
        </el-container>
      </el-container>
    </main>
  </div>
</template>

<script>
import SystemInformation from "./LandingPage/SystemInformation";
import fs from "fs";
import parser from "iptv-playlist-parser";
import DPlayer from "dplayer";
import path from "path";
const playlist = fs.readFileSync(path.join(__static, "zho.m3u"), {
  encoding: "utf-8"
});
const result = parser.parse(playlist);

//console.log(result)
export default {
  name: "landing-page",
  components: { SystemInformation },
  data() {
    return {
      options: [],
      value: "",
      url: ""
    };
  },
  created: function() {
    console.log("wozhixingl", result);
    let arr = [];
    for (let i = 0; i < result.items.length; i++) {
      arr.push({
        value: result.items[i]["name"],
        lable: result.items[i]["url"]
      });
    }
    arr = Array.from(new Set(arr));
    console.log("arr", arr);
    this.options = arr;
  },
  methods: {
    open(link) {
      this.$electron.shell.openExternal(link);
    },
    /*change(t) {
      console.log(t);
      this.url = (this.options.find(option => option.value === t) || {}).lable;
      const dp = new DPlayer({
        container: document.getElementById("dplayer"),
        video: {
          url: this.url,
          type: "hls"
        },
        pluginOptions: {
          hls: {
            // hls config
          }
        }
      });
      console.log(dp.plugins.hls); // Hls 实例
    },*/
    selectTv(t) {
      this.url = (this.options.find(option => option.value === t) || {}).lable;
      const dp = new DPlayer({
        container: document.getElementById("dplayer"),
        video: {
          url: this.url,
          type: "hls"
        },
        pluginOptions: {
          hls: {
            // hls config
          }
        }
      });
    },
    handleOpen(key, keyPath) {
      console.log(key, keyPath);
    },
    handleClose(key, keyPath) {
      console.log(key, keyPath);
    }
  }
};
</script>

<style>
@import url("https://fonts.googleapis.com/css?family=Source+Sans+Pro");

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: "Source Sans Pro", sans-serif;
}

#wrapper {
  background: radial-gradient(
    ellipse at top left,
    rgba(255, 255, 255, 1) 40%,
    rgba(229, 229, 229, 0.9) 100%
  );
  height: 100vh;
  padding: 20px 20px;
  width: 100vw;
}

#logo {
  height: auto;
  margin-bottom: 20px;
  width: 420px;
}

main {
  display: flex;
  justify-content: space-between;
}

main > div {
  flex-basis: 50%;
}

.left-side {
  display: flex;
  flex-direction: column;
}

.welcome {
  color: #555;
  font-size: 23px;
  margin-bottom: 10px;
}

.title {
  color: #2c3e50;
  font-size: 20px;
  font-weight: bold;
  margin-bottom: 6px;
}

.title.alt {
  font-size: 18px;
  margin-bottom: 10px;
}

.doc p {
  color: black;
  margin-bottom: 10px;
}

.doc button {
  font-size: 0.8em;
  cursor: pointer;
  outline: none;
  padding: 0.75em 2em;
  border-radius: 2em;
  display: inline-block;
  color: #fff;
  background-color: #4fc08d;
  transition: all 0.15s ease;
  box-sizing: border-box;
  border: 1px solid #4fc08d;
}

.doc button.alt {
  color: #42b983;
  background-color: transparent;
}
</style>

<style>
#dplayer {
  background: #2c3e50;
  width: 720px;
  height: 420px;
}
</style>