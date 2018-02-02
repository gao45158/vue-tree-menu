<template>
  <div id="app" ref="main" @click="hideRn" @contextmenu.prevent="hideRn">
    <div class="chapter" v-for="(chapter, index) in navList">
      <div class="tab">第{{ chapterNum[index] }}章
        <button @click="addText(index)">添加内容</button>
        <button>批量上传</button>
      </div>
      <div class="item" v-for="(item, indexItem) in chapter" @mouseover="itemShow(index, indexItem)" @mouseout="itemhide">
        <span class="kc-name" @contextmenu.stop.prevent="showMenu($event)"> 
          {{ (index + 1) + '.' + (indexItem + 1) + "： " + item.videoName }} 
        </span> 
        <a class="file"> {{ item.fileName }} </a> 
        <a v-show="itemIn[0] === index && itemIn[1] === indexItem" class="cz-btn">修改</a>
        <a v-show="itemIn[0] === index && itemIn[1] === indexItem" class="cz-btn" @click="deleteItem(index, indexItem)">删除</a>
      </div>
    </div>
    <transition name="rnshow">
      <div class="add" v-show="addShow">
        <!-- <form action=""> -->
          <input type="text" v-model="addList.valueText">
          <input type="file" @change="getFile($event)">
          <button @click="addBtn">提交</button>
        <!-- </form> -->
      </div>
    </transition>
    <transition name="rnshow">
      <div class="right-nav" @contextmenu.stop.prevent @click.stop v-show="rnShow" ref="rnav">
        <a @click="hideRn">添加课程</a>
        <a @click="hideRn">添加课程</a>
        <a @click="hideRn">添加课程</a>
      </div>
    </transition>
    <transition name="rnshow">
      <div class="mixin" v-show="addShow"></div>
    </transition>
  </div>
</template>

<script>
import Axios from "axios";

export default {
  name: "App",
  data() {
    return {
      navList: {},
      addList: {
        valueText: "",
        fileType: "",
        fileName: ""
      },
      tjINdex: -1,
      addShow: false,
      chapterNum: { 0: "一", 1: "二", 2: "三", 3: "四", 4: "五", 5: "六", 6: "七", 7: "八", 8: "九", 9: "十", 10: "十一", 11: "十二"},
      typeList: {
        imgType: ["png", "jpg", "jpeg", "bmp", "gif", "bmp", "Tif", "psd", { typeName: "[图形文件]" }],
        textType: ["doc", "docx", "xls", "xlsx", { typeName: "[文本文件]" }],
        animationType: ["swf", { typeName: "[动画文件]" }],
        audioType: ["wav", "mp3", { typeName: "[音频文件]" }],
        videoType: ["avi", "mpg", "mpeg", "mov", "rm", "rmvb", "mp4", { typeName: "[视频文件]" }],
        compressType: ["rar", "zip", { typeName: "[压缩文件]" }],
        htmlType: ["htm", "html", { typeName: "[网页文件]" }]
      },
      itemIn: [-1, -1],
      rnShow: false
    };
  },
  created() {
    Axios.get("http://localhost:3000/video").then(responses => {
      let listData = responses.data;
      this.navList = listData;
    });
  },
  methods: {
    itemShow(index, indexItem) {
      this.itemIn = [index, indexItem];
    },
    itemhide() {
      this.itemIn = [-1, -1];
    },
    hideRn() {
      this.rnShow = false;
    },
    showMenu(e) {
      var pos = this.getMousePos(e);
      this.rnShow = true;
      this.$refs.rnav.style.left = pos.x - this.$refs.main.offsetLeft - 15  + "px";
      this.$refs.rnav.style.top = pos.y - 15 + "px";
    },
    addText(index) {
      this.tjINdex = index;
      this.addShow = true;
    },
    getFile(e) {
      this.addList.fileType = e.target.files[0].name;
      let i = this.addList.fileType.lastIndexOf(".");
      if (i > -1) {
        this.addList.fileType = this.addList.fileType.substring(i+1);
      }
    },
    addBtn() {
      // e.preventDefault();

      if (this.addList.valueText.trim() == "") return;
      
      // let formData = new FormData();
      // formData.append("name", this.addList.valueText);
      // formData.append("file", this.addList.filePath);
      // Axios.post("http://localhost:3000", formData).then(function(res) {

        this.navList[this.tjINdex].push({
          videoName: this.addList.valueText,
          fileType: this.addList.fileType,
          fileName: ""
        });
        
        var typeCount = this.attrCount(this.typeList);    
        var file = this.navList[this.tjINdex][this.navList[this.tjINdex].length-1];

        for (var items in this.typeList) {
          this.typeList[items].forEach(item => {
            if (item == file.fileType) {
              file.fileName = this.typeList[items][this.typeList[items].length-1].typeName
            }
          });
        }

      // });

      this.addList.valueText = "";
      this.addShow = false;
    },
    deleteItem(index, indexItem) {
      this.navList[index].splice(indexItem, 1);
    },
    attrCount(obj) {
      var count = 0;
      for(var i in obj) {
        if(obj.hasOwnProperty(i)) {
          count++;
        }
      }
      return count;
    },
    getMousePos(e) {
      var e = event || window.event;
      var scrollX = document.documentElement.scrollLeft || document.body.scrollLeft;
      var scrollY = document.documentElement.scrollTop || document.body.scrollTop;
      var x = e.pageX || e.clientX + scrollX;
      var y = e.pageY || e.clientY + scrollY;
      return { 'x': x, 'y': y };
    }
  }
};
</script>

<style>
  * {
    margin: 0;
    padding: 0;
  }
  #app {
    margin: auto;
    position: relative;
  }
  .right-nav {
    /* min-width: 120px; */
    background: #fff;
    position: absolute;
    left: 0;
    top: 0;
    border: 1px solid #f0f0f0;
    box-sizing: border-box;
    box-shadow: 1px 3px 10px #666;
    border-radius: 5px;
  }
  .right-nav a {
    display: block;
    line-height: 28px;
    padding: 0px 10px;
    margin: 3px;
    border-bottom: 1px dashed #f6f6f6;
    box-sizing: border-box;
    cursor: pointer;
  }
  .right-nav a:hover {
    background: #f60;
    color: #fff;
    border-bottom: 1px solid #f60;
  }
  .right-nav.rnshow-enter-active, .right-nav.rnshow-leave-active {
    transition: opacity 0.5s
  }
  .right-nav.rnshow-enter, .right-nav.rnshow-leave-active {
    opacity: 0 
  }   
  .mixin {
    width: 100%;
    height: 100%;
    position: fixed;
    left: 0; 
    top: 0;
    background: rgba(0, 0, 0, 0.8);
    z-index: 10;
  }   
  .chapter {
    width: 100%;
  }
  .tab {
    height: 40px;
    line-height: 40px;
    background: #ccc;
    margin: 10px;
    padding: 0 10px;
    font-size: 16px;
    color: #000;
    font-weight: 500;
    border-radius: 5px;
  }
  .item {
    height: 36px;
    line-height: 36px;
    margin: 5px 10px;
    padding: 0 15px;
    border-bottom: 1px solid #f5f5f5;
    box-sizing: border-box;
    font-size: 14px;
    color: #666;
    background: #f9f9f9;
  }
  .item:hover {
    background: #f2f2f2;
  }
  .kc-name {
    cursor: default;
  }
  .cz-btn {
    display: inline-block;
    margin: 0 5px;
    color: #069;
    cursor: pointer;
  }
  .cz-btn:hover {
    color: #f60;
  }
  button {
    background: #069;
    border: none;
    height: 28px;
    line-height: 28px;
    color: #fff;
    float: right;
    margin: 6px 0px 0px 6px;
    padding: 0px 10px;
    cursor: pointer;
    border-radius: 5px;
  }
  .file {
    color: #03c;
    cursor: pointer;
    font-size: 14px;
    font-weight: 900;
    margin: 0 15px;
  }
  .add {
    width: 500px;
    height: 300px;
    background: #fff;
    position: fixed;
    left: 50%;
    top: 50%;
    margin: -150px 0 0 -250px;
    z-index: 20;
  }
</style>