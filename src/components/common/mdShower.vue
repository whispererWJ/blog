<!--
 * @Author: whisperer
 * @Date: 2019-08-02 15:24:36
 * @LastEditors: whisperer
 * @Description: 
 -->
<template>
  <section v-html="cleanHtml"></section>
</template>

<script>
import marked from "marked";
import createDOMPurify from "dompurify";
import { JSDOM } from "jsdom";
import "whatwg-fetch";

export default {
  name: "markdown-shower",
  data: ()=>({
    cleanHtml: ""
  }),
  props: {
    url: {
      type: String,
      required: true
    },
    onloading:{
      type: Function,
      required: true
    },
    onShow:{
      type: Function,
      required: true
    }
  },
  mounted() {
    this.getMd(); 
  },
  watch: {
    url: function (val, oldVal) {
        if(val&&val!==oldVal){
            this.getMd(); 
        }
    }
  },
  methods: {
    //净化得到的html文本,防止xss等攻击
    cleanMd(dirty) {
      const win = (new JSDOM("")).defaultView,
        DOMPurify = createDOMPurify(win);
      return DOMPurify.sanitize(dirty);
    },
    //将markdown文本转化成html文本
    transMdIntoHtml(str) {
      return marked(str);
    },
    //获取markdown文本
    getMd() {
      const location = window.location,
            baseUrl =  `${location.origin}/${location.pathname.replace('/','')}/`;
      this.$props.onloading&&this.$props.onloading();
      window
        .fetch(`${baseUrl}${this.$props.url}`)
        .then((response) => response.text())
        .then((mdStr) => this.transMdIntoHtml(mdStr))
        .then((dirty) => this.cleanMd(dirty))
        .then((clean)=>{ 
          this.cleanHtml = clean;
          this.$props.onShow&&this.$props.onShow();
        });
    }
  }
};
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
