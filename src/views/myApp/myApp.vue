<template>
  <div class="myApp">
    <form v-if="!this.$store.state.username" v-show="!show">
      <div class="form-group">
        <label>用户名:</label>
        <input type="text" class="form-control" v-model="account">
      </div>
      <div class="form-group">
        <label for="pwd">密码:</label>
        <input type="password" class="form-control" v-model="password">
      </div>
      <button type="button" class="btn btn-primary" @click="sumbit()">登录</button>
      <button type="button" class="btn btn-primary" @click="show=true" style="float:right">注册</button>
    </form>

    <form v-else>
      <div class="me">
        <div class="title">个人信息修改</div>
        <div class="content">
          <div class="myInfo">
            <div class="myName">
              <span>我的名字：</span>
              <input type="text" v-model="name">
              <br>
            </div>

            <div class="myHead">
              <input
                ref="uploadImg"
                type="file"
                accept="image/png, image/gif, image/jpeg"
                @change="previewImg"
              >
              <img :src="imgsrc" alt="我的头像" @click.stop="activePreview">
              <p>点击图片更改头像</p>
            </div>
          </div>
          <div class="save">
            <button class="saveBtn" @click="changeInfo">提交</button>
          </div>
        </div>
      </div>
    </form>
    <form v-show="show">
      <div class="form-group">
        <label>用户名:</label>
        <input type="text" class="form-control" v-model="account">
      </div>
      <div class="form-group">
        <label>密码:</label>
        <input type="password" class="form-control" v-model="password">
      </div>
      <button type="button" class="btn btn-primary" @click="SignUp()">注册</button>
      <button type="button" class="btn btn-primary" @click="show=false" style="float:right">取消</button>
    </form>
    <div class="modal fade" id="myModal">
      <div class="modal-dialog">
        <div class="modal-content">
          <!-- 模态框主体 -->
          <div class="modal-body">输入信息有误</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import api from "@/fetch/api";
export default {
  name: "myApp",
  data() {
    return {
      account: "",
      password: "",
      show: false,
      imgsrc: "",
      name: ""
    };
  },
  created() {
    this.account = "";
    this.password = "";
    this.show = false;
  },
  methods: {
    changeInfo() {
            let r=confirm('确定保存吗？');
            if(r){
                let file = this.$refs.uploadImg.files[0];
                let formData = new FormData();
                formData.append('file',file);
                formData.append('name',this.name);
                formData.append('signature',this.signature);
                formData.append('breif',this.breif);

                console.log(formData);
                api.changeMyInfo(formData).then(res=>{
                    switch(res.code){
                        case '200':
                        console.log(res);
                        alert('信息修改成功');
                        this.reload();
                        break;
                        case '400':
                        console.log('服务器开小差了，信息修改失败');
                        break;
                    }
                },err=>{
                    console.log('信息修改失败');
                });
            }
    },
    activePreview() {
      this.$refs.uploadImg.click();
    },

    previewImg() {
      let file = this.$refs.uploadImg.files[0], // 获取第一张图片
        imageType = /^image\//, //校验图片的正则表达式
        reader = "",
        that = this; //存储this

      // 文件是否为图片
      if (!imageType.test(file.type)) {
        alert("请选择图片！");
        return;
      }
      // 判断是否支持FileReader
      if (window.FileReader) {
        reader = new FileReader();
      } else {
        alert("您的浏览器不支持图片预览功能，如需该功能请升级您的浏览器！");
        return;
      }
      //将图片读取为URL（base64）方便前端展示
      reader.readAsDataURL(file);
      // 读取完成
      reader.onload = function(event) {
        // 图片路径设置为读取的图片
        that.imgsrc = event.target.result;
        // console.log(that.headImg);
      };
    },
    SignUp() {},
    sumbit() {
      //账号名和密码是否合规验证（待补充）
      api.logins(this.account, this.password).then(
        res => {
          console.log(res);
          switch (res.code) {
            case "200":
              this.$store.state.userid = res.data[0].id;
              this.$store.state.username = res.data[0].name;
              this.name = res.data[0].name;
              this.imgsrc = res.data[0].imgsrc;
              //如果一开始img不存在，那么就显示默认图片
              if (!this.imgsrc) {
                this.imgsrc = "/static/img/myImg.jpg";
              }
              //将服务器返回的token放入store中
              this.$store.commit("logins", res.token);
              break;
            case "401":
              $("#myModal").modal({ backdrop: false });
              setTimeout(() => {
                $("#myModal").modal("hide");
              }, 1000);

              this.err = true;
              break;
          }
        },
        err => {
          console.log(err);
        }
      );
    }
  }
};
</script>

<style lang="less" scoped>
.myApp {
  position: absolute;
  top: 20px;
  bottom: 20px;
  left: 20px;
  right: 20px;
  background: white;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  .iconfont {
    font-size: 80px;
  }
  h1 {
    font-size: 40px;
  }
}
</style>
<style scoped lang="less">
.me {
  box-sizing: border-box;
  .title {
    height: 80px;
    line-height: 80px;
    background: white;
    padding: 0 20px;
    font-size: 25px;
    box-sizing: border-box;
    border-left: 10px solid #353d47;
  }
  .content {
    padding: 30px;
    box-sizing: border-box;
    background: white;
    margin-top: 20px;
    input {
      margin: 0 10px;
      padding: 5px 7px;
    }
    .myInfo {
      display: flex;
      justify-content: center;
      align-items: center;
      .myName {
        flex: 2;
        input {
          margin: 20px 0;
          width: 50%;
        }
      }
      .myHead {
        flex: 1;
        padding-bottom: 20px;
        box-sizing: border-box;
        input {
          opacity: 0;
          height: 0;
        }
        img {
          width: 100px;
          height: 100px;
          border-radius: 50%;
          vertical-align: middle;
          cursor: pointer;
        }
        p {
          margin-top: 10px;
        }
      }
    }

    .myBrief {
      margin: 20px 0;
      textarea {
        width: 100%;
        height: 100px;
        margin: 10px 0;
        border-radius: 4px;
        padding: 3px 8px;
        box-sizing: border-box;
        font-size: 18px;
        resize: none;
      }
    }
    .save {
      text-align: right;
      .saveBtn {
        width: 70px;
        height: 30px;
        background: #5cb85c;
        border: 1px solid #4cae4c;
        color: white;
        border-radius: 4px;
        outline: none;
        cursor: pointer;
      }
    }
  }
}
</style>