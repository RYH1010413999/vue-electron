
<template>
  <div @click="getFocus" class="home">
    <div class="home-title">
      <p>自助报到机</p>
    </div>
    <input
      type="text"
      placeholder="请扫码或手工输入患者编号"
      v-model="inputValue"
      ref="codeInput"
      class="home-input"
      @click="visible = true"
      @keyup.enter="submit"
    />

    <div class="home-report" @click="submit" v-loading="loading">报到</div>

    <div class="home-defeated">{{errorCode}}</div>

    <div class="home-keyboard">
      <div @click="inputDiv" :data-index="''" class="home-keyboard-number">
        <div :data-index="1">1</div>
        <div :data-index="2">2</div>
        <div :data-index="3">3</div>
        <div :data-index="0">0</div>
        <div :data-index="4">4</div>
        <div :data-index="5">5</div>
        <div :data-index="6">6</div>
        <p style="width:165px;"></p>
        <div :data-index="7">7</div>
        <div :data-index="8">8</div>
        <div :data-index="9">9</div>
        <div class="home-keyboard-number-del" :data-index="'删除'">删除</div>
      </div>
    </div>
    <div class="home-set" @dblclick="goUserSet"></div>
  </div>
</template>
<script>
import { start } from "repl";
import errorCodeFun from "../errorCode";

export default {
  name: "Home",
  data() {
    return {
      dialogVisible: false,
      inputValue: "",
      loading: false,
      errorCode: ""
    };
  },
  methods: {
    async submit() {
      // 防抖
      if (this.inputValue === "" && this.errorCode === "请输入报到码") {
        return;
      }
      if (this.inputValue === "") {
        this.errorCode = "请输入报到码";
        this.delayedTime();
        return;
      }
      this.loading = true;
      const data = { patient_pid: `${this.inputValue}` };
      try {
        const res = await this.$axios.patientReport2(data);
        if (res.error_code === "success") {
          this.$router.push(`/patientDetail/${res.patient_name}`);
        } else {
          this.errorCode = errorCodeFun.patientReport2Error(res.error_code);
        }
        this.loading = false;
        this.inputValue = "";
        this.delayedTime();
      } catch (error) {
        this.loading = false;
      }
    },

    delayedTime() {
      const timeObj = setTimeout(() => {
        this.errorCode = "";
        clearTimeout(timeObj);
      }, 5000);
    },

    goUserSet() {
      this.$router.push("/userSet");
    },

    getFocus() {
      this.$refs.codeInput.focus();
    },

    inputDiv(e) {
      if (e.target.nodeName.toLowerCase() === "div") {
        const index = e.target.dataset.index;
        if (index === "删除") {
          this.inputDivDel();
        } else {
          this.inputValue = this.inputValue + index;
        }
      }
    },

    inputDivDel() {
      const start_index = this.$refs.codeInput.selectionStart;
      const end_index = this.$refs.codeInput.selectionEnd;
      let index;
      if (start_index === 0 && end_index === 0) {
        return;
      }
      if (start_index === end_index) {
        this.inputValue = this.inputValue.replace(
          this.inputValue[start_index - 1],
          ""
        );
        index = start_index - 1;
      } else {
        this.inputValue =
          this.inputValue.substring(0, start_index) +
          this.inputValue.substring(end_index);
        index = start_index;
      }

      this.$nextTick(() => {
        this.$refs.codeInput.selectionStart = index;
        this.$refs.codeInput.selectionEnd = index;
      });
    }
  },
  mounted() {
    this.input = this.$refs.codeInput;
    this.$refs.codeInput.focus();
  }
};
</script>
<style lang="less" scoped>
.home {
  width: 90%;
  height: 100%;
  text-align: center;
  padding-top: 67px;
  &-title {
    font-size: 48px;
    color: #ffffff;
    font-size: 48px;
    font-family: SimHei;
    font-weight: 400;
  }
  &-input {
    width: 576px;
    height: 78px;
    box-shadow: 0px 1px 0px 0px rgba(255, 255, 255, 0.25),
      0px 0px 4px 0px rgba(255, 255, 255, 0.15);
    border-radius: 10px;
    font-size: 36px;
    text-align: center;
    background: rgba(255, 255, 255, 1);
    margin-top: 100px;
    margin-bottom: 60px;
    border: none;
  }
  &-report {
    width: 175px;
    height: 67px;
    margin: 0 auto;
    background: url(../assets/button.png);
    opacity: 0.8;
    text-align: center;
    line-height: 62px;
    font-size: 30px;
    font-family: Adobe Heiti Std;
    font-weight: normal;
    color: rgba(255, 255, 255, 1);
    margin-bottom: 40px;
    cursor: pointer;
  }
  &-defeated {
    font-size: 30px;
    color: #ff3535;
    font-family: SimHei;
    font-weight: 400;
    height: 34px;
  }
  &-keyboard {
    width: 100%;
    display: flex;
    justify-content: center;
    margin-top: 70px;
    &-number {
      display: flex;
      flex-wrap: wrap;
      width: 660px;
      & > div {
        width: 165px;
        height: 125px;
        background: url(../assets/keyboard.png);
        font-size: 48px;
        font-family: CenturyGothic;
        font-weight: bold;
        color: white;
        line-height: 115px;
      }
      & > &-del {
        height: 245px;
        line-height: 240px;
        background: url(../assets/keyboard_del.png);
        position: relative;
        bottom: 123px;
      }
    }
  }
  &-set {
    width: 50px;
    height: 50px;
    position: absolute;
    right: 0;
    top: 230px;
  }
}
</style>