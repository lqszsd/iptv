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
              <el-submenu index="2">
                <template slot="title">
                  <i class="el-icon-menu"></i>
                  <span>虎牙直播</span>
                </template>
                <el-menu-item-group>
                  <el-menu-item
                    v-for="item in data.gameList"
                    @click="selectHuya(item.profileRoom)"
                    :key="item.id"
                    :index="item.id"
                    style="padding: 0 20px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;"
                  >{{item.introduction}}</el-menu-item>
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
import request from "request";
import cheerio from "cheerio";
// const playlist = fs.readFileSync(path.join(__static, "zho.m3u"), {
//   encoding: "utf-8"
// });
// const result = parser.parse(playlist);
// console.log(result);
export default {
  name: "landing-page",
  components: { SystemInformation },
  data() {
    return {
      options: [],
      huya: [],
      value: "",
      url: "",
      data: {},
      page: 1,
      m: {},
      videourl: "" //
    };
  },
  watch: {
    videourl: (url, oldurl) => {
      console.log(url, oldurl);
      let dp = new DPlayer({
        container: document.getElementById("dplayer"),
        video: {
          url,
          type: "hls"
        },
        autoplay: true, //自动播放
        live: true, //直播视频形式
        pluginOptions: {
          hls: {
            // hls config
          }
        }
      });
    }
  },
  created: function() {
    this.init();
  },
  methods: {
    init: function() {
      this.getTvData();
      this.getHyData();
    },
    open(link) {
      this.$electron.shell.openExternal(link);
    },
    handleOpen(key, keyPath) {
      // console.log(key, keyPath);
      if (key == "1" && this.options.length == "0") {
        return this.$message("正在解析TV频道请稍后");
      }
    },
    handleClose(key, keyPath) {
      // console.log(key, keyPath);
    },
    // 获取全部电视
    getTvData() {
      // https://iptv-org.github.io/iptv/languages/zho.m3u
      this.requestFun({
        url: "https://iptv-org.github.io/iptv/languages/zho.m3u",
        timeout: 70000
      })
        .then(res => {
          let result = parser.parse(res);
          let arr = [];
          for (let i = 0; i < result.items.length; i++) {
            arr.push({
              value: result.items[i]["name"],
              lable: result.items[i]["url"]
            });
          }
          this.options = arr;
        })
        .catch(err => {
          console.log(err);
          this.$message("请求出错，请检查网络！");
        });
    },
    //选择TV台
    selectTv(t) {
      this.videourl = (
        this.options.find(option => option.value === t) || {}
      ).lable;
    },
    //获取虎牙全部直播
    getHyData() {
      this.requestFun({
        url:
          "https://m.huya.com/cache.php?m=Live&do=ajaxGetProfileLive&page=1&pageSize=120"
      })
        .then(res => {
          this.data = JSON.parse(res);
        })
        .catch(err => {
          console.log(err);
          this.$message("请求出错，请检查网络！");
        });
    },
    //点击更改虎牙播放视频链接
    selectHuya(t) {
      this.requestFun({
        url: "https://m.huya.com/" + t
      })
        .then(res => {
          var $ = cheerio.load(res);
          let data = $("#html5player-video").attr("src");
          data = data.replace("_2500", "");
          data = data.replace("hw.hls", "al.hls");
          this.videourl = "https://" + data;
        })
        .catch(err => {
          console.log(err);
          this.$message("请求出错，请检查网络！");
        });
    },
    requestFun(params) {
      let {
        url,
        method = "GET",
        headers = {
          "User-Agent":
            "Mozilla/5.0 (iPhone; CPU iPhone OS 13_2_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.0.3 Mobile/15E148 Safari/604.1"
        },
        timeout = 7000
      } = params;
      return new Promise((resolve, reject) => {
        request(
          {
            url,
            method,
            headers,
            timeout
          },
          function(error, response, body) {
            if (!error && response.statusCode == 200) {
              resolve(body);
            } else {
              reject(error);
            }
          }
        );
      });
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
