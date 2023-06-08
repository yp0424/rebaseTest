<template>
  <!-- 填空题 -->
  <div ref="questionBox" class="fillBlank">
    <qTitle :question="question"></qTitle>
    <div v-show="errorTips" class="tips">{{ this.errorTips }}</div>
    <div v-show="smsCodeErrorTips" class="tips">{{ this.smsCodeErrorTips }}</div>
    <div class="q-options">
      <div class="q-option-list">
        <!--                手机/固话-->
        <mbTelSelect
          @focus="onFocus(question)"
          @blur="onBlur(question)"
          :id="question.cid + '__'"
          :option="question"
          :question="question"
          :fail="question.fail"
          v-if="question.content_type && question.content_type == 'mobile_tel'"
          :value.sync="optionBlank[question.cid]"
          :mtSelect.sync="optionBlank[question.cid + '_type']"
        ></mbTelSelect>

        <!--                日期组件-->
        <pickerDate
          @focus="onFocus(question)"
          @blur="onBlur(question)"
          :id="question.cid + '__'"
          :option="question"
          :question="question"
          :fail="question.fail"
          v-else-if="question.content_type && question.content_type == 'date'"
          :value.sync="optionBlank[question.cid]"
        ></pickerDate>

        <!--                普通input-->
        <template v-else>
          <base-text-area
            :classOut="[{ 'q-blank-flex': question.unit }]"
            :classInner="['q-blank', 'q-option-input', { fail: question.fail }]"
            :unitIsShow="question.unit"
            ref="textarea"
            :id="question.cid + '__'"
            @blur="onBlur(question)"
            @focus="onFocus(question)"
            :placeholder="placeholder"
            :value.sync="optionBlank[question.cid]"
            :type="question.content_type"
            :option="question"
          ></base-text-area>
          <p style="margin:0.34rem 0 0.24rem">请签名</p>
          <div :id="question.cid+'canvas'" style="border:1px solid #e1e4eb"></div>
          <button style="margin-top:0.1rem;padding:0.05rem" @click="submitSign">提交签名</button>
          <button style="margin:0.1rem 0 0 0.05rem;padding:0.05rem" @click="clearSign">清空签名</button>
        </template>

        <div
          v-if="smsCodeShow && !isModifyAnswer"
          class="q-blank q-option-input q-sms-code"
          :class="{ fail: smsFail }"
        >
          <input
            v-model="smsCode"
            ref="smsCode"
            :placeholder="$t('placeholder.check_code')"
            rows="1"
          />
          <div
            @click="getSmsVerifyCode"
            class="q-sms-code__button"
            :class="{
              sending: sending
            }"
          >
            <span v-if="!sending">{{ $t("validate.send_check_code") }}</span>
            <span v-else>{{ $t("validate.re_send", number, { time: number }) }}</span>
          </div>
        </div>
      </div>
    </div>
    <qUpload v-if="question.attachData && question.attachData.openAttach" :question="question"></qUpload>
  </div>
</template>
<script>
export default {
  data: function data() {
    return {
      optionBlank: {},
      errorTips: "",
      smsCodeShow: false,
      smsFail: false,
      smsCodeErrorTips: "",
      smsCode: "",
      sending: false,
      number: 60,
      sendingTimer: null,
      smsCodeVerify: false,
      isModifyAnswer: logic.isModify,
      canvasDiv: null,
      offsetTop: 0,
      offsetLeft: 0,
      mouseX: 0,
      mouseY: 0,
      clickX: [],
      clickY: [],
      clickDrag: [],
      paint: false,
      point: {}
    };
  },
  props: ["question", "pNode"],
  inject: ["errorTipsApi"],
  created: function created() {
    console.log("blank", this.question);
    this.initData();
  },
  mounted: function mounted() {
    this.initCanvas();
  },
  destroyed(){
      this.canvas.removeEventListener("mousedown", this.mouseDown);
      this.canvas.removeEventListener("mousemove", this.mouseMove);
      this.canvas.removeEventListener("mouseup", this.mouseUp);
      this.canvas.removeEventListener("mouseleave", this.mouseUp);
      //移动端监听touch
      this.canvas.removeEventListener("touchstart", this.touchStart);
      this.canvas.removeEventListener("touchmove",this.touchMove);
      this.canvas.removeEventListener("touchend",this.touchEnd)
  },
  computed: {
    placeholder: function() {
      if (this.question && this.question.placeholder_text) {
        return this.question.placeholder_text;
      }
      if (this.question.content_type === "mobile") {
        return this.$t("placeholder.mobile");
      }
      return this.$t("placeholder.input");
    },
    surveyId: function surveyId() {
      return this.$route.params.surveyId;
    }
  },
  watch: {
    errorTipsApi: {
      deep: true,
      handler: function(n, o) {
        if (n.value && this.question && n.value.gid == this.question.gid) {
          this.errorTips = n.value.msg;
        }
      }
    }
  },
  methods: {
    initData: function initData() {
      var self = this;
      console.log(
        "self.question111111*********************************",
        self.question,
        self.question.content_type
      );
      self.answer = null;
      if (
        self.question.open_validate &&
        self.question.content_type === "mobile"
      ) {
        self.smsCodeShow = true;
      }
    },
    onFocus: function onFocus(question) {
      this.question.fail = false;
      this.pNode
        ? this.pNode.answerChanged(this)
        : this.$parent.answerChanged(this);
    },
    onBlur: function onBlur(question) {
      this.pNode
        ? this.pNode.answerChanged(this)
        : this.$parent.answerChanged(this);

      //失去焦点检验格式是否正确
      // this.validate();
    },
    clearCompAnswer: function clearCompAnswer() {
      Object.assign(this.$data, this.$options.data());
      this.initData();
    },
    getAnswer: function getAnswer() {
      var open = this.optionBlank[this.question.cid],
        ans = "";
      if (open) {
        ans = [this.trims(open)];
      }
      return ans;
    },
    setAnswer: function setAnswer(answer) {
      console.log(answer);
      this.optionBlank[this.question.cid] = answer[0];
      this.pNode
        ? this.pNode.answerChanged(this)
        : this.$parent.answerChanged(this);
      this.$forceUpdate();
    },
    getAnswerStr: function getAnswerStr(oid) {
      var answer = this.getAnswer() || {};
      if (!oid) {
        var result = [];
        for (var key in answer) {
          result.push(answer[key]);
        }
        return result.join(",");
      } else {
        return answer[oid + "_open"] || "";
      }
    },
    getSmsVerifyCode: function getSmsVerifyCode() {
      if (this.sending) {
        return false;
      }
      // 获取验证码前验证手机号
      var answer_option = this.optionBlank[this.question.cid];
      if (
        !answer_option ||
        !answer_option[0] ||
        !this.trims(answer_option[0])
      ) {
        this.errorTips = this.$t("validate.answer");
        this.question.fail = true;
        return false;
      }
      //开放类型校验
      var validate = [],
        dataType = "";
      var type = this.question.content_type;
      if (type == "date") dataType = this.question.words_number_range;
      if (type) {
        validate = this.$validates.text_type(
          this.trims(answer_option),
          type,
          dataType
        );
        if (!validate || !validate[0]) {
          this.errorTips = validate[1];
          this.question.fail = true;
          return false;
        }
      }
      //部分校验大小
      if (type == "int" || type == "decimal" || type == "number") {
        validate = this.$validates.maxmin_check(
          this.trims(answer_option),
          rule.setValue
        );
        if (!validate || !validate[0]) {
          this.errorTips = validate[1];
          this.question.fail = true;
          return validate[0];
        }
      }
      var params = {
        seq: this.question.seq || this.question.gid,
        phone: this.getAnswer()[0]
      };
      var self = this;
      this.$getSmsVerifyCode(this.surveyId, params).then(function(res) {
        if (res.code === 0) {
          self.sending = true;
          self.number = 60;
          self.sendingTimer = setInterval(function() {
            self.number--;
            if (self.number === 0) {
              self.sending = false;
              clearInterval(self.sendingTimer);
            }
          }, 1000);
          self.vcodeID = res.data.vcodeID;
        } else if (res.code === 202) {
          self.errorTips = self.$t("api.202");
          self.question.fail = true;
          return false;
        }
      });
    },
    verifySmsCode: function verifySmsCode() {
      this.smsFail = false;
      this.question.fail = false;
      if (!this.smsCode || !this.trims(this.smsCode)) {
        this.smsCodeVerify = false;
        this.smsFail = true;
        this.smsCodeErrorTips = this.$t("placeholder.check_code");
        return false;
      }
      if (!/^\d{6}$/.test(this.trims(this.smsCode))) {
        this.smsCodeVerify = false;
        this.smsFail = true;
        this.smsCodeErrorTips = this.$t("validate.format_check_code");
        return false;
      }
      var surveyId = this.surveyId;
      var params = {
        seq: this.question.seq || this.question.gid,
        phone: this.getAnswer()[0],
        vcodeID: this.vcodeID,
        vcode: this.smsCode
      };

      var self = this;
      self.smsCodeErrorTips = "";
      var validate = false;
      if (this.smsCodeVerify) {
        return true;
      }
      this.$verifySmsCode(surveyId, params).then(function(res) {
        var code = res.code;
        self.smsCodeErrorTips = self.$t("loading.validate");
        self.smsCodeVerify = false;
        if (code === 0) {
          self.smsFail = false;
          self.smsCodeVerify = true;
          self.smsCodeErrorTips = "";
          self.$parent.bus.$emit("parentValidate");
        } else if (code === 202) {
          self.smsFail = true;
          self.smsCodeErrorTips = self.$t("api.202");
        } else if (code === 1100) {
          self.smsFail = true;
          self.smsCodeErrorTips = self.$t("api.1100");
        } else if (code === 4) {
          self.smsFail = true;
          self.smsCodeErrorTips = self.$t("api.4");
        } else {
          self.smsFail = true;
          self.smsCodeErrorTips = self.$t("api.network");
        }
      });
      return validate;
    },
    validate: function(showError, screen) {
      var config = { showError: showError, screen: screen };
      var validate_instance = new this.$QuestionAnswerValidate(
        this.question,
        config
      );
      this.$once("hook:beforeDestroy", function() {
        validate_instance.destroy();
        validate_instance = null;
      });
      var result = validate_instance.validate({
        open_text: this.optionBlank[this.question.cid],
        mobileOrTel: this.optionBlank[this.question.cid + "_type"],
        smsCodeShow: this.smsCodeShow,
        verifySmsCode: this.verifySmsCode
      });
      if (!result.status) {
        this.errorTips = result.errorTips;
      }
      return result.status;
    },
    customValidate: function customValidate(self) {
      //自定义答案校验代码，self是局部题目vue对象，可以进行参数设置
      // const s = `Qx.${self.question.cid}.select()`;
      // self.$refs.qComponent.testTips = "test!";
      return true;
    },
    customNext: function customNext() {
      //自定义下一题代码
      //返回qcid组成的list, 跳转到该题
      //也可返回'1','2','3'。'1'：成功完成,'2'：被筛选，'3'：配额满
      //暂未使用此功能2019.9.2
    },
    sizeF: function sizeF() {
      var s = 1;

      if (this.question.inputbox_height == "middle") {
        s = 3;
      }

      if (this.question.inputbox_height == "big") {
        s = 5;
      }

      return s;
    },
    initCanvas() {
      //初始化Canvas
      this.canvasDiv = document.getElementById(this.question.cid + "canvas");
      //获取当前元素距离屏幕边缘的距离
      setTimeout(()=>{
        //生产Canva画板
         this.canvas = document.createElement("canvas");
         let canvasWidth = this.canvasDiv.clientWidth;
         this.canvas.setAttribute("width", canvasWidth);
         this.canvas.setAttribute("height", 400);
         this.canvasDiv.appendChild(this.canvas);
         this.context = this.canvas.getContext("2d");
         this.point.notFirst = false;
         //添加监听事件
         this.canvas.addEventListener("mousedown", this.mouseDown);
         this.canvas.addEventListener("mousemove", this.mouseMove);
         this.canvas.addEventListener("mouseup", this.mouseUp);
         this.canvas.addEventListener("mouseleave", this.mouseUp);
         //移动端监听touch
         this.canvas.addEventListener("touchstart", this.touchStart);
         this.canvas.addEventListener("touchmove",this.touchMove);
         this.canvas.addEventListener("touchend",this.touchEnd);
         this.offsetLeft = this.canvasDiv.getBoundingClientRect().left;
         this.offsetTop = this.canvasDiv.getBoundingClientRect().top;
      });
      
    },
    //鼠标点击事件
    mouseDown(e) {
      this.mouseX = e.pageX - this.offsetLeft;
      this.mouseY = e.pageY - this.offsetTop;
      this.paint = true;
      this.addClick(this.mouseX, this.mouseY);
      this.redraw();
    },
    mouseMove(e) {
      if (this.paint) {
        let x = e.pageX - this.offsetLeft;
        let y = e.pageY - this.offsetTop;
        this.addClick(x, y, true);
        this.redraw();
      }
    },
    mouseUp(e) {
      this.paint = false;
    },
    touchStart(e){
      e.preventDefault();
      this.mouseX = e.touches[0].pageX - this.offsetLeft;
      this.mouseY = e.touches[0].pageY - this.offsetTop;
      this.paint = true;
      this.addClick(this.mouseX, this.mouseY);
      this.redraw();
    },
    touchMove(e){
      e.preventDefault();
      if (this.paint) {
        let x = e.touches[0].pageX - this.offsetLeft;
        let y = e.touches[0].pageY - this.offsetTop;
        this.addClick(x, y, true);
        this.redraw();
      }
    },
    touchEnd(){
      this.paint = false;
    },
    addClick(x, y, dragging) {
      this.clickX.push(x);
      this.clickY.push(y);
      this.clickDrag.push(dragging);
    },
    //绘制Canvas
    redraw() {
      this.context.strokeStyle = "#000";
      this.context.lineJoin = "round";
      this.context.lineWidth = 2;
      while (this.clickX.length > 0) {
        this.point.bx = this.point.x;
        this.point.by = this.point.y;
        this.point.x = this.clickX.pop();
        this.point.y = this.clickY.pop();
        this.point.drag = this.clickDrag.pop();
        this.context.beginPath();
        if (this.point.drag && this.point.notFirst) {
          this.context.moveTo(this.point.bx, this.point.by);
        } else {
          this.point.notFirst = true;
          this.context.moveTo(this.point.x - 1, this.point.y);
        }
        this.context.lineTo(this.point.x, this.point.y);
        this.context.closePath();
        this.context.stroke();
      }
    },
    submitSign(){
      var blank = document.createElement('canvas');//系统获取一个空canvas对象
      blank.width = this.canvas.width;
      blank.height = this.canvas.height;
      let isNull=this.canvas.toDataURL()==blank.toDataURL(); //判断canvas是否为空
      if(!isNull){
        this.optionBlank[this.question.cid]=this.canvas.toDataURL("image/png");
        this.$forceUpdate();
      }
    },
    clearSign(){
      this.optionBlank[this.question.cid]="";
      this.canvas.width=this.canvas.width;
      this.$forceUpdate();
    }
  }
};
</script>
<style lang="less" scoped>
.q-sms-code {
  position: relative;
  margin-top: 0.16rem;
  .q-sms-code__button {
    position: absolute;
    right: 0.32rem;
    color: #2672ff;
    cursor: pointer;
    top: 50%;
    transform: translateY(-50%);
    -ms-transform: translateY(-50%);
    -moz-transform: translateY(-50%);
    -webkit-transform: translateY(-50%);
    &.sending {
      color: #c0c4cc;
    }
  }
}
@media (max-width: 680px) {
  .q-sms-code {
    margin: 0.16rem 0 0 0;
  }
}
</style>
