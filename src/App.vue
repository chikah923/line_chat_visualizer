<template>
  <div id="app">
    <div style="display:flex;justify-content:flex-start">

      <div style="width:50%">

        <!-- ▼LINE風ここから -->
        <div class="line__container" id="target">
        <!-- タイトル -->
          <div class="line__title">
            <span style="float:left;"><</span>
            <span style="margin-left:40px;">{{ name }}</span>
            <span style="float:right;">
            <img src="./assets/call_head.png">
            <img src="./assets/menu.png" style="margin-left:17px;margin-right:6px;">
            </span>
          </div>

         <!-- ▼会話エリア scrollを外すと高さ固定解除 -->
          <div class="line__contents">

            <div v-for="chat in chatStr" v-bind:key="chat.str_no">
              <!-- 相手の吹き出し -->
              <div class="line__left" v-if="chat.category == 'editedYourText'">
  
                <figure>
                  <img v-show="uploadedImage" :src="uploadedImage" />
                </figure>
                <div class="line__left-text">
                  <div class="name"> {{ name }}</div>
                  <div class="text" style="float:left;">{{ chat.textContent }}</div>
                </div>
              </div>
          
              <!-- 自分の吹き出し -->
              <div class="line__right" v-if="chat.category == 'editedMyText'">
                <div class="text">{{ chat.textContent }}</div>
                <!-- <span class="date">既読</span> -->
              </div>

              <!-- 相手からの電話 -->
              <div class="line__left" v-if="chat.category == 'yourCall'">
                <figure>
                  <img v-show="uploadedImage" :src="uploadedImage" />
                </figure>
                <div class="line__left-text">
                  <div class="name"> {{ name }}</div>
                  <div class="text call">
                    <div>
                      <img src="./assets/call.png" style="height:30px;width:30px;">
                    </div>
                    <div>{{ chat.callTime }}</div>
                  </div>
                </div>
              </div>

                <!-- 自分からの電話 -->
              <div class="line__right" v-if="chat.category == 'myCall'">
                <div class="text call">
                  <div>
                    <img src="./assets/call.png" style="height:30px;width:30px;">
                  </div>
                  <div>{{ chat.callTime }}</div>
                </div>
                <!-- <span class="date">既読</span> -->
              </div>

            </div>
          </div>
          <!--　▲会話エリア ここまで -->
        </div>
        <!--　▲LINE風 ここまで -->
      </div>
      <div class="flex_input">
        <div>
          <button style="margin-right:10px;" @click="saveCanvas1()">line画像キャプチャ作成</button>

          <button @click="saveCanvas()">line画像ダウンロード</button>
        </div>

        <br>
        <br>
    
        <div>
          <span>○会話相手の名前： </span>
          <input v-model="name" placeholder="name">
        </div>
    
        <br>
        <br>
    
        <div>
        <span>○会話相手のプロフィール画像：</span><br>
          <input type="file" v-on:change="onFileChange">
          <img class="profile_display" v-show="uploadedImage" :src="uploadedImage" />
        </div>
    
        <br>
        <br>
    
        <div>
          <span>○会話内容入力： </span><br>
          <textarea v-model="chatContent" placeholder="会話内容の入力" rows="4" cols="50"></textarea>
        </div>

        <br>
        <div id="canvas_display"></div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'app',
  data() {
    return {
      name: '',
      uploadedImage: '',
      chatStr: [],
      chatContent: '',
    }
  },
  methods: {
    onFileChange(e) {
      let files = e.target.files || e.dataTransfer.files;
      this.createImage(files[0]);
    },
    createImage(file) {
      let reader = new FileReader();
      reader.onload = (e) => {
        this.uploadedImage = e.target.result;
      };
      reader.readAsDataURL(file);
    },
    saveCanvas1() {
      var scaleBy = 3;
      var w = document.querySelector('#target').clientWidth +100;
      var h = document.querySelector('#target').clientHeight + 300;
      var div = document.querySelector('#target');
      var canvas = document.createElement('canvas');
      canvas.width = w * scaleBy;
      canvas.height = h * scaleBy;
      canvas.style.width = w + 'px';
      canvas.style.height = h + 'px';
      var context = canvas.getContext('2d');
      context.scale(scaleBy, scaleBy);
      html2canvas(div, {
        canvas:canvas,
        onrendered: function(canvas) {
          document.body.appendChild(canvas);
          canvasData = canvas;
        }
      }); 
    },
    saveCanvas() {
      var scaleBy = 3;
      var w = document.querySelector('#target').clientWidth + 200;
      var h = document.querySelector('#target').clientHeight + 200;
      var div = document.querySelector('#target');
      var canvas = document.createElement('canvas');
      canvas.width = w * scaleBy;
      canvas.height = h * scaleBy;
      canvas.style.width = w + 'px';
      canvas.style.height = h + 'px';
      var context = canvas.getContext('2d');
      context.scale(scaleBy, scaleBy);
      
      html2canvas(div, 
      {
        canvas:canvas,
        onrendered: function (canvas) {
          var a = document.createElement('a');
          a.href = canvas.toDataURL("image/png");
          a.download = 'chat.png';
          a.click();
        }
      });
    },    
  },
  watch: {
    chatContent() {
      const str = this.chatContent
      const chatStr = []
      //ここでの分け方を変える
      //まず区切り文字で分ける
      const s_str = str.split('chat')
      console.log(s_str)

      //ここでchatStr配列を作成する。
      for(let chat=1; chat < s_str.length; chat++) {
        //相手からの会話
        if(s_str[chat].startsWith('1:')) {
          const yourText = {
            str_no: chat,
            category: 'editedYourText',
            textContent: s_str[chat].replace('1:', '')
          }
          chatStr.push(yourText)
        }
        //自分からの会話
        if(s_str[chat].startsWith('2:')) {
          const myText = {
            str_no: chat,
            category: 'editedMyText',
            textContent: s_str[chat].replace('2:', '')
          }
          chatStr.push(myText)
        }
        //相手からの電話
        if(s_str[chat].startsWith('3:')) {
          if (s_str[chat].indexOf('missed') != -1) {
            var callTime = '応答なし'
          } else {
            var callTime = s_str[chat].replace('3:', '')
          }
          var yourCall = {
            str_no: chat,
            category: 'yourCall',
            callTime: callTime
          }
          chatStr.push(yourCall)
          continue
        }
        //自分からの電話
        if(s_str[chat].startsWith('4:')) {
          if (s_str[chat].indexOf('missed') != -1) {
            var callTime = '応答なし'
          } else {
            var callTime = s_str[chat].replace('4:', '')
          }
          var myCall = {
            str_no: chat,
            category: 'myCall',
            callTime: callTime
          }
          chatStr.push(myCall)
          continue
        }
      }
      this.chatStr = chatStr
    }
  }
}
</script>

<style>
/* 正方形を用意 */
.line__container .line__left figure img {
  border-radius: 50%;
  width: 50px;
  height: 50px;
}
textarea {
  resize: both;
}

.profile_display {
  border-radius: 50%;
  width: 50px;
  height: 50px;
}


.flex_input {
  margin-left: 6%;
}
.line__container {
  padding:0;
  background: #7494c0;
  overflow: hidden;
  font-size: 80%;
  width: 70%;
}
/* タイトル部分 */
.line__container .line__title {
  background: #273246;
  padding: 10px;
  text-align: center;
  font-weight: bold;
  font-size: 150%;
  color: #ffffff;
  line-height: 30px;
}

.line__container .line__title img {
  height:20px;
  width:20px;
}

/* 会話部分 */
.line__container .line__contents {
  padding: 10px;
  overflow: hidden;
  line-height: 135%;
}

.line__container .scroll {
  height: 500px;
  overflow-y: scroll;
}


/* 相手の会話 */
.line__container .line__left {
    width: 100%;
    position: relative;
    display: block;
    margin-bottom: 60px;
    max-width: 80%;
    clear: both;
}

/* アイコン画像 */
.line__container .line__left figure {
    width: 50px;
    position: absolute;
    top: 15px;
    left: 0;
    padding: 0;
    margin: 0;

}

.line__container .line__left .line__left-text {
  margin-left: 70px;
}

.line__container .line__left .line__left-text .name {
  text-align: left;
  font-size: 80%;
  color: #ffffff;
}

/* コメントエリア */
.line__container .line__left .text {
  white-space:pre-wrap;
  /* word-wrap:break-word; */
  text-align: left;
  margin: 0;
  position: relative;
  padding: 10px;
  border-radius: 20px;
  background-color: #ffffff; 
}

/* 電話 */
.line__container .line__left .call {
  display:flex;
  flex-flow:column;
  justify-content:space-between;
  text-align: center;
  font-size: 11px;
  width: 50px;
  padding-left:15px;
  padding-right: 15px;
}

/* 吹き出し */
.line__container .line__left .text::after {
  content: '';
  position: absolute;
  /* display: block; */
  width: 0;
  height: 0;
  left: -10px;
  top: 10px;
  border-right: 20px solid #ffffff;
  border-top: 10px solid transparent;
  border-bottom: 10px solid transparent;
}

/* 自分の会話 */
.line__container .line__right {
    position: relative;
    display: block;
    margin: 10px 15px 5px 0;
    max-width: 80%;
    float: right;
    margin-right: 15px;
    clear: both;
}

/* コメントエリア */
.line__container .line__right .text {
  white-space:pre-wrap;
  /* word-wrap:break-word; */
  text-align: left;
  padding: 10px;
  border-radius: 20px;
  background-color: #8de055;
  margin: 0;
  margin-left: 60px;
}

/* 電話 */
.line__container .line__right .call {
  display:flex;
  flex-flow:column;
  text-align: center;
  width: 50px;
  padding-left:19px;
  padding-right: 15px;
  font-size: 11px;
}

/* 吹き出し */
.line__container .line__right .text::after {
  content: '';
  position: absolute;
  display: block;
  width: 0;
  height: 0;
  right: -10px;
  top: 10px;
  border-left: 20px solid #8de055;
  border-top: 10px solid transparent;
  border-bottom: 10px solid transparent;
}

/* 既読エリア */
.line__container .line__right .date {
  content: '';
  position: absolute;
  display: block;
  width: 100px;
  text-align: right;
  left: -30px;
  bottom: 0px;
  font-size: 80%;
  color: #ffffff;
}

button {
  display: inline-block;
  padding: 0.5em 1em;
  text-decoration: none;
  background: #668ad8;/*ボタン色*/
  color: #FFF;
  border-bottom: solid 4px #627295;
  border-radius: 3px;
}
button:active {
  /*ボタンを押したとき*/
  -webkit-transform: translateY(4px);
  transform: translateY(4px);/*下に動く*/
  border-bottom: none;/*線を消す*/
}

#app {
  font-family: "Hiragino Kaku Gothic ProN", "ヒラギノ角ゴ ProN W3", "メイリオ", Meiryo, "ＭＳ Ｐゴシック", "MS PGothic", sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: left;
  color: #2c3e50;
  padding-top: 60px;
}

</style>