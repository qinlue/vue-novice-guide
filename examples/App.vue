<template>
  <div id="app">
    <img alt="Vue logo"
         src="./assets/logo.png">
    <NoviceGuide v-if="show"
                 ref="NoviceGuide"
                 scrollNode="aaa"
                 model="shade"
                 :allGuideNodes="allGuideNodes"
                 uniqueName="test" />
    <div id="aaa">
      引导1
    </div>
    <div id="bbb">
      引导2
    </div>
    <div>
      <button @click="aaa">开始引导</button>
    </div>
  </div>
</template>

<script>
import NoviceGuide from '../src/index'
export default {
  name: 'App',
  data () {
    return {
      show: false,
      allGuideNodes: [
        {
          targetNode: null,
          nodeId: 2,
          priority: 1,
          tips: {
            content: '<div>我是tipsContent</div>'
          },
          decorateContent: `<div>
          <div>我是decorateContent</div>
            <div> 我是decorateContent</div>
          <div>我是decorateContent</div>
          <div>我是decorateContent</div>
          <div>我是decorateContent</div>
          <div>我是decorateContent</div>
          <div>我是decorateContent</div>
          <div>我是decorateContent</div>
          <div>我是decorateContent</div>
          <div>我是decorateContent</div>
          <div>我是decorateContent</div>
           <div>我是decorateContent</div>
            <div>我是decorateContent</div>
             <div>我是decorateContent</div>
            <div>我是decorateContent</div>
             <div>我是decorateContent</div>
             <div>我是decorateContent</div>
             <div>我是decorateContent</div>
             <div>我是decorateContent</div>
             <div>我是decorateContent</div>
             <div>我是decorateContent</div>
             <div>我是decorateContent</div>
            <div>我是decorateContent</div>
             <div>我是decorateContent</div>
             <div>我是decorateContent</div>
             <div>我是decorateContent</div>
             <div>我是decorateContent</div>
          <div>我是decorateContent</div></div>`
        },
        {
          targetNode: null,
          priority: 2,
          nodeId: 3,
          tips: {
            content: '<div>我是tipsContent</div>'
          },
          decorateContent: `<div>我是decorateContent</div>`

        }
      ]
    }
  },
  methods: {
    aaa () {
      const aaa = document.getElementById('aaa')
      const bbb = document.getElementById('bbb')
      this.allGuideNodes = this.allGuideNodes.map((item, index) => {
        if (index === 0) {
          item.targetNode = aaa
        } else {
          item.targetNode = bbb
        }
        return item
      })
      this.show = true
    }
  },
  mounted () {

  },
  components: {
    NoviceGuide
  }
}
</script>

<style>
#aaa {
  /* background: red; */
}
/* #app {
  height: 100%;
  width: 100%;
} */
</style>
