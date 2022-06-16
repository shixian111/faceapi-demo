<template>
  <div class="bigScreen">
    <div class="leftContainer">
      <div class="dataStatistic">
        <!-- <div class="watchStatistic">
              <div class="title">
                <p class="title-value" >人员进入刷脸统计</p>
              </div>
              <div class="content">
               <near-six-month width="100%" height="100%" :EnterPeople="EnterPeople"></near-six-month>
              </div>
            </div> -->
      </div>
      <div class="dataStatistic">
        <!-- <bar-chart width="100%" height="100%" :toolbox="false"></bar-chart> -->
      </div>
    </div>
    <div class="rightContainer">
      <div class="webrtc_face_recognition">
        <div class="option">
          <div class="cameraBtn">
            <span class="svg-container" @click="fnOpen">
              <el-tooltip
                class="item"
                effect="dark"
                content="打开"
                placement="bottom"
              >
                <svg-icon icon-class="runVideo" />
              </el-tooltip>
            </span>
            <span class="svg-container" @click="fnClose">
              <el-tooltip
                class="item"
                effect="dark"
                content="关闭"
                placement="bottom"
              >
                <svg-icon icon-class="endVideo" />
              </el-tooltip>
            </span>
          </div>
          <div class="chooseIt">
            <label>
              轮廓检测
              <input type="radio" v-model="detection" value="landmark" />
            </label>
            <label>
              表情检测
              <input type="radio" v-model="detection" value="expression" />
            </label>
            <label>
              年龄性别检测
              <input type="radio" v-model="detection" value="age_gender" />
            </label>
          </div>
        </div>
        <div class="see">
          <video
            id="myVideo"
            poster="https://dummyimage.com/1280x720"
            muted
            loop
            playsinline
            @loadedmetadata="fnRun"
          ></video>
          <canvas id="myCanvas" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import * as faceapi from "face-api.js";
export default {
  name: "WebRTCFaceRecognition",
  data() {
    return {
      nets: "tinyFaceDetector", // 模型
      options: null, // 模型参数
      withBoxes: true, // 框or轮廓
      detectFace: "detectAllFaces", // 单or多人脸
      detection: "landmark",
      videoEl: null,
      canvasEl: null,
      timeout: 0,
      // 视频媒体参数配置
      constraints: {
        audio: false,
        video: {
          // ideal（应用最理想的）
          width: {
            min: 320,
            ideal: 1280,
            max: 1920,
          },
          height: {
            min: 240,
            ideal: 720,
            max: 1080,
          },
          // frameRate受限带宽传输时，低帧率可能更适宜
          frameRate: {
            min: 15,
            ideal: 30,
            max: 60,
          },
          // 显示模式前置后置
          facingMode: "environment",
        },
      },
      dk: false, //摄像头的提示信息
      gb: false, //摄像头的提示信息
    };
  },
  watch: {
    nets(val) {
      this.nets = val;
      this.fnInit();
    },
    detection(val) {
      this.detection = val;
      this.videoEl.pause();
      setTimeout(() => {
        this.videoEl.play();
        setTimeout(() => this.fnRun(), 300);
      }, 300);
    },
  },
  mounted() {
    this.$nextTick(() => {
      this.fnInit();
    });
  },
  methods: {
    // 初始化模型加载
    async fnInit() {
      await faceapi.nets[this.nets].loadFromUri("/static/models"); // 算法模型
      await faceapi.loadFaceLandmarkModel("/static/models"); // 轮廓模型
      await faceapi.loadFaceExpressionModel("/static/models"); // 表情模型
      await faceapi.loadAgeGenderModel("/static/models"); // 年龄模型
      // 根据算法模型参数识别调整结果
      switch (this.nets) {
        case "ssdMobilenetv1":
          this.options = new faceapi.SsdMobilenetv1Options({
            minConfidence: 0.5, // 0.1 ~ 0.9
          });
          break;
        case "tinyFaceDetector":
          this.options = new faceapi.TinyFaceDetectorOptions({
            inputSize: 512, // 160 224 320 416 512 608
            scoreThreshold: 0.5, // 0.1 ~ 0.9
          });
          break;
        case "mtcnn":
          this.options = new faceapi.MtcnnOptions({
            minFaceSize: 20, // 0.1 ~ 0.9
            scaleFactor: 0.709, // 0.1 ~ 0.9
          });
          break;
      }
      // 节点属性化
      this.videoEl = document.getElementById("myVideo");
      this.canvasEl = document.getElementById("myCanvas");
    },
    // 人脸面部勘探轮廓识别绘制
    async fnRunFaceLandmark() {
      console.log("RunFaceLandmark");
      if (this.videoEl.paused) return clearTimeout(this.timeout);
      // 识别绘制人脸信息
      const result = await faceapi[this.detectFace](
        this.videoEl,
        this.options
      ).withFaceLandmarks();
      if (result && !this.videoEl.paused) {
        const dims = faceapi.matchDimensions(this.canvasEl, this.videoEl, true);
        const resizeResult = faceapi.resizeResults(result, dims);
        this.withBoxes
          ? faceapi.draw.drawDetections(this.canvasEl, resizeResult)
          : faceapi.draw.drawFaceLandmarks(this.canvasEl, resizeResult);
      } else {
        this.canvasEl
          .getContext("2d")
          .clearRect(0, 0, this.canvasEl.width, this.canvasEl.height);
      }
      this.timeout = setTimeout(() => this.fnRunFaceLandmark());
    },
    // 人脸表情识别绘制
    async fnRunFaceExpression() {
      console.log("RunFaceExpression");
      if (this.videoEl.paused) return clearTimeout(this.timeout);
      // 识别绘制人脸信息
      const result = await faceapi[this.detectFace](this.videoEl, this.options)
        .withFaceLandmarks()
        .withFaceExpressions();
      if (result && !this.videoEl.paused) {
        const dims = faceapi.matchDimensions(this.canvasEl, this.videoEl, true);
        const resizeResult = faceapi.resizeResults(result, dims);
        this.withBoxes
          ? faceapi.draw.drawDetections(this.canvasEl, resizeResult)
          : faceapi.draw.drawFaceLandmarks(this.canvasEl, resizeResult);
        // faceapi.draw.drawFaceExpressions(this.canvasEl, resizeResult, 0.05);  //绘制默认的框
        if (Array.isArray(resizeResult)) {
          resizeResult.forEach((result) => {
            let obj = result.expressions;
            let maxKey = Object.keys(obj).sort(function (a, b) {
              return obj[b] - obj[a];
            })[0];
            var mostExpress = "";
            var mostExpressSugg = "";
            const expressEmo = [
              { angry: { label: "生气", suggest: "多笑一笑哦" } },
              { disgusted: { label: "不耐烦", suggest: "试着让自己平静下来" } },
              { fearful: { label: "害怕", suggest: "放轻松一点" } },
              {
                happy: { label: "开心", suggest: "每天都要保持这样的好心情哦" },
              },
              { neutral: { label: "平静", suggest: "心静如水是最理智的状态" } },
              { sad: { label: "伤心", suggest: "加油,人生没有过不去的坎" } },
              {
                surprised: { label: "惊讶", suggest: "遇到什么好玩的事情了?" },
              },
            ];
            // const expressSugg=[{'angry':''},{'disgusted':'试着让自己平静下来'},{'fearful':'放轻松一点'},{'happy':'每天都要保持这样的好心情哦'},{'neutral':'心静如水'},{'sad':'加油,人生没有过不去的坎'},{'surprised':'遇到什么好玩的事情了?'}]
            for (let item of expressEmo) {
              // console.log("ha",item,typeof maxKey,maxKey);
              // console.log("lll",item);
              // console.log("xixi",item[maxKey])
              if (item[maxKey]) {
                mostExpress = item[maxKey]["label"];
                mostExpressSugg = item[maxKey]["suggest"];
                break;
              }
            }
            new faceapi.draw.DrawTextField(
              [
                `你看起来有点${mostExpress}哦`,
                // `${mostExpressSugg}`
              ],
              result.detection.box.topLeft,
              {
                fontSize: 16,
              }
            ).draw(this.canvasEl);
          });
        }
      } else {
        this.canvasEl
          .getContext("2d")
          .clearRect(0, 0, this.canvasEl.width, this.canvasEl.height);
      }
      this.timeout = setTimeout(() => this.fnRunFaceExpression());
    },
    // 年龄性别识别绘制
    async fnRunFaceAgeAndGender() {
      console.log("RunFaceAgeAndGender");
      if (this.videoEl.paused) return clearTimeout(this.timeout);
      // 识别绘制人脸信息
      const result = await faceapi[this.detectFace](this.videoEl, this.options)
        .withFaceLandmarks()
        .withAgeAndGender();
      if (result && !this.videoEl.paused) {
        const dims = faceapi.matchDimensions(this.canvasEl, this.videoEl, true);
        const resizeResults = faceapi.resizeResults(result, dims);
        this.withBoxes
          ? faceapi.draw.drawDetections(this.canvasEl, resizeResults)
          : faceapi.draw.drawFaceLandmarks(this.canvasEl, resizeResults);
        if (Array.isArray(resizeResults)) {
          // console.log("hh",resizeResults);
          resizeResults.forEach((result) => {
            // console.log("ceshi",result);
            const { age, gender, genderProbability } = result;
            new faceapi.draw.DrawTextField(
              [
                `${Math.round(age, 0)} 岁`,
                `${gender == "female" ? "女" : "男"} (检测可信度:${Math.round(
                  genderProbability
                )})`,
              ],
              result.detection.box.topLeft,
              {
                fontSize: 16,
              }
            ).draw(this.canvasEl);
          });
        } else {
          const { age, gender, genderProbability } = resizeResults;
          new faceapi.draw.DrawTextField(
            [
              `${Math.round(age, 0)} 岁`,
              `${gender == "female" ? "女" : "男"} (检测可信度:${Math.round(
                genderProbability
              )})`,
            ],
            resizeResults.detection.box.bottomLeft
          ).draw(this.canvasEl);
        }
      } else {
        this.canvasEl
          .getContext("2d")
          .clearRect(0, 0, this.canvasEl.width, this.canvasEl.height);
      }
      this.timeout = setTimeout(() => this.fnRunFaceAgeAndGender());
    },
    // 执行检测识别类型
    fnRun() {
      if (this.detection === "landmark") {
        this.fnRunFaceLandmark();
        return;
      }
      if (this.detection === "expression") {
        this.fnRunFaceExpression();
        return;
      }
      if (this.detection === "age_gender") {
        this.fnRunFaceAgeAndGender();
        return;
      }
    },
    // 启动摄像头视频媒体
    fnOpen() {
      if (typeof window.stream === "object") return;
      clearTimeout(this.timeout);
      this.timeout = setTimeout(() => {
        clearTimeout(this.timeout);
        navigator.mediaDevices
          .getUserMedia(this.constraints)
          .then(this.fnSuccess)
          .catch(this.fnError);
      }, 300);
    },
    // 成功启动视频媒体流
    fnSuccess(stream) {
      window.stream = stream; // 使流对浏览器控制台可用
      this.videoEl.srcObject = stream;
      this.videoEl.play();
    },
    // 失败启动视频媒体流
    fnError(error) {
      console.log(error);
      alert("视频媒体流获取错误" + error);
    },
    // 结束摄像头视频媒体
    fnClose() {
      this.canvasEl
        .getContext("2d")
        .clearRect(0, 0, this.canvasEl.width, this.canvasEl.height);
      this.videoEl.pause();
      clearTimeout(this.timeout);
      if (typeof window.stream === "object") {
        window.stream.getTracks().forEach((track) => track.stop());
        window.stream = "";
        this.videoEl.srcObject = null;
      }
    },
  },
  beforeDestroy() {
    this.fnClose();
  },
};
</script>

<style lang="scss" scoped>
.bigScreen {
  width: 100%;
  height: 100%;
  overflow: hidden;
  .leftContainer {
    float: left;
    // width:40%;
    .dataStatistic {
    }
  }
  .rightContainer {
    .see {
      position: relative;
      width: 100%;
      height: 100%;
      video {
        width: 100%;
        height: 100%;
      }
    }
    .see canvas {
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      width: 100%;
      height: 100%;
    }
    .option {
      position: absolute;
      left: 30px;
      padding: 20px 20px;
      z-index: 999;
      .cameraBtn {
        display: flex;
        float: left;
        margin-left: 10px;
        .svg-container {
          display: flex;
          flex-direction: column;
          width: 70px;
          font-size: 20px;
          cursor: pointer;
          .tips {
            float: left;
            color: #fff;
            margin-left: -15px;
            font-size: 12px;
            margin-top: 5px;
          }
        }
      }
      .chooseIt {
        float: left;
        // padding: 3px;
        font-size: 12px;
        label {
          margin-left: 20px;
        }
      }
    }
  }
}
</style>>

