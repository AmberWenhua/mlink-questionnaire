<template>
<!-- 满意度调查 -->
  <div class="satisfaction">
    <Layout :showContent="!toResult" type="satisfy">
      <img :src="$t('satisfaction.头部banner')" alt="" slot="banner">
      <q-one slot="middle" v-if="active === 0" @selectQ1="selectQ1" />
      <q-two
        slot="middle" 
        v-if="active === 1"
        @selectQ2="selectQ2"
        @selectSub="selectSub"
        @changeOther="changeOther"
      />
      <com-status
        slot="other"
        :status="status"
        :msg="msg"
      />
      <div slot="bottom">
        <van-button
          v-if="!toResult"
          :disabled="disabled"
          round
          color="#6f38d4"
          block
          @click="next"
          v-prevent-re-click
        >
          {{statusText}}
        </van-button>
        <van-button
          v-else
          round
          color="#6f38d4"
          block
          :disabled="disabled"
          @click="statusClick"
        >
          {{statusText}}
        </van-button>
      </div>
    </Layout>
  </div>
</template>

<script>
import { Button } from 'vant';
import QOne from './question/Q-one';
import QTwo from './question/Q-two';
import Layout from '@c/layout.vue';
import ComStatus from '@c/status.vue';
import index from '@/mixin/index.js';
import { isApp, callMylinkApp } from "@common/handle.js";

export default {
  name: "Home",
  components: {
    vanButton: Button,
    QOne,
    QTwo,
    Layout,
    ComStatus
  },
  mixins: [index],
  data() {
    return {
      active: 0,
      statusText: this.$t('下一步'),
      disabled: true,
      toResult: false,
      submitParam: {
        content: "", // 多选内容
        other: "", // 其他内容
        score: 0, // 满意度
        type: "" // 类别：界面、体验、功能
      },
      status: '',
      msg: '',
      isRepeat: false,
    };
  },
  created() {},
  mounted() {
    window.setWebViewFlag = () => {};
  },
  methods: {
    // 题目一
    selectQ1(val) {
      this.submitParam.score = val;
      this.disabled = false;
    },
    // 题目二
    selectQ2(val) {
      this.submitParam.type = val;
    },
    // 多选
    selectSub(val){
      if(val.length) {
        let temp = val.filter(item => item !== 'input');
        this.submitParam.content = JSON.stringify(temp);
        // 如果选项中没有勾选其他，则数据赋值为空
        if(val.includes('input') && !this.submitParam.other) {
          this.disabled = true;
        } else {
          this.disabled = false;
        }
      } else {
        this.disabled = true;
      }
    },
    changeOther(val) {
      if(val) {
        this.disabled = false;
      } else {
        this.disabled = true;
      }
      this.submitParam.other = val;
    },
    // 下一步
    next() {
      if(this.active < 1) {
        this.active++;
        this.disabled = true;
        this.statusText = this.$t('提交');
      }else {
        if(isApp()) {
          this.submit();
        } else {
          callMylinkApp(window.location.href);
        }
      }
    },
    // 提交结果页按钮点击
    statusClick() {
      if(this.status === 'fail') {
        if(this.isRepeat) {
          this.toMyPoints();
          return;
        }
        // 失败时，返回答题页
        this.toResult = false;
        this.active = 1;
        this.statusText = this.$t('提交');
      } else {
        this.toMyPoints();
      }
    },
    // 提交
    submit() {
      console.log('提交内容', this.submitParam);
      this.$post("/survey/satisfy/add", this.submitParam).then(res => {
        if(+res.code === 0) {
          console.log(res);
          this.status  = 'success';
          this.statusText = this.$t('我的积分');
          this.msg = `500 ${this.$t('发送积分')}`;
        } else {
          this.status  = 'fail';
          this.msg = this.languageType === 'sc' ? res.msg :
          this.languageType === 'tc' ? res.msgTw : res.msgEn;
          if(+res.code === 1081) {
            this.isRepeat = true;
            this.statusText = this.$t('我的积分');
          } else {
            this.statusText = this.$t('重新填写');
          }
        }
        this.toResult = true;
      })
    },

  },
};
</script>

<style lang="scss" scoped>
.satisfaction {
  .bottom {
    position: fixed;
    bottom: 0;
    right: 0;
    left: 0;
    padding: 30px * $scale 50px * $scale;
    background: #fff;
  }
}
</style>