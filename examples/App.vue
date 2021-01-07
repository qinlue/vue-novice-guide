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
      11111
    </div>
    <div>
      <button @click="aaa">点击</button>
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
        // {
        //   targetNode: null,
        //   nodeId: 2,
        //   tips: {
        //     content: '<div>我是tipsContent</div>'
        //   },
        //   decorateContent: `<div><div>我是decorateContent</div>
        //     <div> 我是decorateContent</div>
        //   <div>我是decorateContent</div>
        //   <div>我是decorateContent</div>
        //   <div>我是decorateContent</div>
        //   <div>我是decorateContent</div>
        //   <div>我是decorateContent</div>
        //             <div>我是decorateContent</div>

        //   <div>我是decorateContent</div>

        //   <div>我是decorateContent</div>

        //   <div>我是decorateContent</div>

        //   <div>我是decorateContent</div></div >`

        // },
        {
          targetNode: null,
          nodeId: 3,
          tips: {
            content: '<div>我是tipsContent</div>'
          },
          decorateContent: `<div><div>我是decorateContent</div>
            < div > 我是decorateContent</>
          <div>我是decorateContent</div>
          <div>我是decorateContent</div>
          <div>我是decorateContent</div>
          <div>我是decorateContent</div>
          <div>我是decorateContent</div>
          <div>我是decorateContent</div></div >`

        }
      ]
    }
  },
  methods: {
    aaa () {
      const dom = document.getElementById('aaa')
      setTimeout(() => {
        this.allGuideNodes = this.allGuideNodes.map((item) => {
          item.targetNode = dom
          return item
        })
        this.show = true
      })
      // const ref = this.$refs.FinFinNoviceGuide
      // // ref.beginGuide()

      // console.log(ref, 'ref')
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
#app {
  height: 100%;
  width: 100%;
}
</style>
