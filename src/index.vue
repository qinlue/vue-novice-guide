<template>
</template>
<script>
import { LS } from '../common/tools'
export default {
    name: 'FinFinNoviceGuide',
    data () {
        return {
            guideIndex: 0,
            noviceGuideNodes: [],
            hasGuideNodes: JSON.parse(LS.getItem(`_${this.uniqueName}`)) || [],
            needGuideNodes: [],
            axis: { // 当前节点坐标
                Y: 0,
                X: 0
            },
            currentNodeMode: this.model
        }
    },
    props: {
        model: {
            type: String,
            default: 'shade',
        },
        allGuideNodes: {
            type: Array,
            default: () => []
        },
        zIndex: {
            type: Number,
            default: 99
        },
        uniqueName: {
            type: String,
            default: ''
        },
        preventScroll: {
            type: Boolean,
            default: true
        },
        scrollNode: {
            type: String,
            default: ''
        }
    },
    computed: {
        targetNodeIndex() {
            const { zIndex, guideIndex } = this
            return zIndex + guideIndex + 1
        }
    },
    watch: {
        guideIndex(value) {
            if (value < this.needGuideNodes.length) {
                this.beginGuide()
            } else {
                this.endGuide()
            }
        }
    },
    created () {
        this.filterGuideNodes()
        if (this.needGuideNodes.length) {
            this.generateNodes()
            this.beginGuide()
        }
    },
    mounted () {
    },
    methods: {
        beginGuide() {
            this.findTargetNode()
            this.getAllStyle()
            this.setGuideNodeStyle()
            this.calculateTargetNodeArea()
            this.combineWrapNode()
            this.updateHasGuideNodes()
        },
        filterGuideNodes() {
           this.needGuideNodes = this.allGuideNodes.filter(item => item.targetNode && !this.hasGuideNodes.includes(item.nodeId))
           this.needGuideNodes.forEach(item => {
               if (!item.priority) {
                   item.priority = 99
               }
           })
           this.needGuideNodes.sort((a, b) => a.priority - b.priority)
           console.warn('this.needGuideNodes', this.allGuideNodes, this.needGuideNodes)
        },
        findTargetNode() {
            const { targetNode = null, currentMode = 'shade' } = this.needGuideNodes[this.guideIndex]
            this.targetNode = targetNode
            this.currentNodeMode = currentMode
            this.guideNodes = this.needGuideNodes[this.guideIndex]
        },
        generateNodes() {
            const len = this.needGuideNodes.length
            this.fragment = document.createDocumentFragment()
            this.maskNode = document.createElement('div')
            this.maskNode.className = 'guide-mask'
            this.maskNode.style.zIndex = this.zIndex
            this.wrapNode = document.createElement('div')
            this.wrapNode.className = 'guide-wrap'
            this.wrapNode.style.position = 'absolute'
            this.wrapNode.style.zIndex = this.zIndex + len + 1
            this.wrapNode.addEventListener('click', () => {
                this.onHandleNextStep()
            })
            this.shadeNode = document.createElement('div')
            this.shadeNode.className = 'guide-shade'
            this.shadeNode.style.zIndex = this.zIndex + len
            this.shadeNode.addEventListener('click', () => {
                this.onHandleNextStep()
            })
            this.tipsNode = document.createElement('div')
            this.decorateNode = document.createElement('div')
            this.decorateNode.className = 'guide-decorate'
            this.nextStepNode = document.createElement('div')
            this.nextStepNode.className = 'guide-next-step'
            this.cloneTargetNode = document.createElement('div')
            this.nextStepNode.addEventListener('click', () => {
                this.onHandleNextStep()
            })
        },
        getAllStyle() {
            const { targetNode } = this.guideNodes
            const position = window.getComputedStyle(targetNode, null).getPropertyValue('position')
            const currentNodePos = targetNode.getBoundingClientRect()
            let { left, top, width, height } = currentNodePos
            const zIndex = window.getComputedStyle(targetNode, null).getPropertyValue('zIndex')
            const { offsetLeft, offsetTop } = targetNode
            const { scrollLeft, scrollTop } = document.documentElement
            const { clientWidth, clientHeight } = document.body
            const right = clientWidth - width - left
            this.targetNodeStyle = {
                left: left + scrollLeft,
                top: top + scrollTop,
                right,
                width,
                height,
                position,
                zIndex,
                offsetLeft,
                offsetTop,
                clientWidth,
                clientHeight
            }
            console.log('======', this.targetNodeStyle)
        },
        calculateTargetNodeArea() {
            const { clientWidth, clientHeight } = document.body
            const { left, top, width, height, right } = this.targetNodeStyle
            if (parseInt(height / 2 + top) < parseInt(clientHeight / 2)) {
                this.axis.Y = 'top'
            } else {
                this.axis.Y = 'bottom'
            }
            // if (parseInt(width / 2 + left) < parseInt(clientWidth / 2)) {
            //     this.axis.X = 'left'
            // } else if (parseInt(width / 2 + left) > parseInt(clientWidth / 2)) {
            //     this.axis.X = 'right'
            // } else {
            //     this.axis.X = 'center'
            // }
            if (parseInt(left) === parseInt(right)) {
                this.axis.X = 'center'
            } else if (parseInt(left) > parseInt(right)) {
                this.axis.X = 'right'
            } else {
                this.axis.X = 'left'
            }
            console.log('axis', this.axis, clientWidth, width)
            this.calculateScroll()
            this.onPreventScroll()
        },
        calculateScroll() {
            const { top, height, clientHeight, position } = this.targetNodeStyle
            this.scrollElement = document.scrollingElement || document.documentElement
            if (position === 'fixed') {
                this.scrollToPosY(0)
            } else if (top + height > clientHeight) {
                const scrollY = top + height / 2 - clientHeight / 2
                if (this.scrollNode !== '') {
                    this.scrollElement = document.querySelector(`${this.scrollNode}`)
                }
                this.scrollToPosY(scrollY)
            }
        },

        combineWrapNode() {
            const { X, Y } = this.axis
            const { left, top, width, height, offsetTop, position } = this.targetNodeStyle
            const { tips: { className = 'guide-tips', content = '' } = {}, nextStepText = '下一步', decorateContent = '' } = this.guideNodes
            this.tipsNode.className = className
            this.tipsNode.innerHTML = content
            this.nextStepNode.innerHTML = nextStepText
            this.decorateNode.innerHTML = decorateContent
            console.log('xxxxxx', X)
            if (X !== 'center') {
                // this.wrapNode.style.minWidth = '200px'
                this.wrapNode.style.width = width + 'px'
                this.wrapNode.style.left = left + 'px'
            } else {
                this.wrapNode.style.left = 'auto'
                this.wrapNode.style.width = '100%'
            }
            if (Y === 'top') {
                this.wrapNode.style.bottom = ''
                this.wrapNode.style.top = top + height + 'px'
                this.wrapNode.appendChild(this.decorateNode)
                this.wrapNode.appendChild(this.tipsNode)
                this.wrapNode.appendChild(this.nextStepNode)
            } else {
                this.wrapNode.style.top = ''
                if (position === 'fixed') {
                    this.wrapNode.style.bottom = document.body.clientHeight - offsetTop + 'px'
                } else {
                    this.wrapNode.style.bottom = document.body.clientHeight - top + 'px'
                }
                if (!this.wrapNode.className.includes('wrap-y')) {
                    this.wrapNode.className += ' wrap-y'
                }
                this.wrapNode.appendChild(this.tipsNode)
                this.wrapNode.appendChild(this.nextStepNode)
                this.wrapNode.appendChild(this.decorateNode)
            }
            this.fragment.appendChild(this.maskNode)
            this.fragment.appendChild(this.shadeNode)
            if (this.currentNodeMode === 'clone') {
                this.fragment.appendChild(this.cloneTargetNode)
            }
            this.fragment.appendChild(this.wrapNode)
            document.body.appendChild(this.fragment)
        },
        updateHasGuideNodes() {
            const { nodeId } = this.guideNodes
            this.hasGuideNodes.push(nodeId)
            LS.setItem(`_${this.uniqueName}`, JSON.stringify(this.hasGuideNodes))
        },
        resetPreGuideNode() {
            const preGuideIndex = this.guideIndex - 1
            const { targetNode: preNode } = this.needGuideNodes[preGuideIndex]
            const { position, zIndex } = this.targetNodeStyle
            preNode.style.zIndex = zIndex
            preNode.style.position = position
        },
        onPreventScroll() {
            if (this.preventScroll) {
                document.body.style.height = '100vh'
                document.body.style.overflow = 'hidden'
            }
        },
        onHandleNextStep() {
            this.clearTipsAndNextNode()
            this.guideIndex += 1
            this.resetPreGuideNode()
        },
        clearTipsAndNextNode() {
            this.wrapNode.innerHTML = ''
            this.wrapNode.style.top = ''
            console.log('this.currentNodeMode', this.currentNodeMode)
            if (this.currentNodeMode === 'clone') {
                this.cloneTargetNode.innerHTML = ''
                this.cloneTargetNode.style.display = 'none'
            }
        },
        setGuideNodeStyle() {
            if (this.currentNodeMode === 'shade') {
                this.targetNode.style.zIndex = this.targetNodeIndex
                const { position } = this.targetNodeStyle
                if (position === 'static') {
                    this.targetNode.style.position = 'relative'
                }
            } else if (this.currentNodeMode === 'clone') {
                const { left, top, height, right, position, offsetLeft } = this.targetNodeStyle
                this.cloneTargetNode = this.targetNode.cloneNode(true)
                this.cloneTargetNode.style.position = 'absolute'
                this.cloneTargetNode.style.zIndex = this.targetNodeIndex
                this.cloneTargetNode.style.left = left + 'px'
                this.cloneTargetNode.style.top = top + 'px'
                if (position === 'fixed') {
                    this.cloneTargetNode.style.left = offsetLeft + 'px'
                    this.cloneTargetNode.style.top = 'auto'
                }
                this.cloneTargetNode.style.right = right + 'px'
                this.cloneTargetNode.style.height = height + 'px'
            }
        },
        endGuide() {
            this.maskNode.style.display = 'none'
            this.wrapNode.style.display = 'none'
            this.shadeNode.style.display = 'none'
            document.body.style.height = ''
            document.body.style.overflow = ''
            this.scrollToPosY(0)
        },
        scrollToPosY(posY) {
            if (this.scrollElement.scrollTo) {
                this.scrollElement.scrollTo({
                    top: posY,
                    behavior: 'smooth'
                })
            } else {
                this.scrollElement.scrollTop = posY
            }
        }
    }
};
</script>
<style scoped lang='less'>
.guide-mask {
  background: rgba(0,0,0,.7);
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
}
.guide-shade {
  background: transparent;
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
}
.guide-wrap {
  display: flex;
  justify-content: center;
  color: #fff;
  flex-direction: column;
  align-items: center;
  text-align: center;
  font-size: 14px;
  line-height: 20px;
  & > .guide-decorate {
    margin-top: 5px;
    width: 6px;
    height: 40px;
    background: url('https://p1.meituan.net/scarlett/7cf8c04fc3bea1573854f41f5657277c911.png') no-repeat center top;
    background-size: 6px;
  }
  & > .guide-tips {
    margin: 12px auto 18px;
    font-size: 14px;
    font-weight: bold;
  }
}
.wrap-y {
  & > .guide-decorate {
    width: 6px;
    height: 40px;
    background: url('https://p0.meituan.net/scarlett/b2dbd2e17aa909ab9ca57fbff08ee855901.png') no-repeat center top;
    background-size: 6px;
    margin-bottom: 5px;
  }
  & > .guide-tips {
    margin: 0 auto 12px;
  }
  & > .guide-next-step {
    margin-bottom: 12px;
  }
}

.guide-next-step {
  border-radius: 9px;
  height: 30px;
  line-height: 30px;
  padding: 0 17px;
  display: inline-block;
  border: solid 1px #fff;
  text-align: center;
  cursor: pointer;
  font-weight: bold;
}
</style>
