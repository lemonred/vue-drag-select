<template>
  <div class="vue-drag-select" @mousedown="onMouseDown">
    <slot :seletedItem="lastSelectedData" :hoveredItem="hoveredItem"/>
    <div v-if="mouseDown" class="vue-drag-select-box"
      :style="selectionBoxStyling"></div>
  </div>
</template>

<script>
  export default {
    name: 'vue-drag-select',
    props: {
      selectorClass: {
        type: String,
        required: true
      },
      clearSection: { /*清除选区标志，如果为true则清除当前的选择*/
        type: Boolean,
        required: false
      },
      selectGroupType: { 
        /*
        2:是多个组，返回选区类型[[item, item],[item, item]],[item, item]]]
        1: 返回数据为一个组，返回选区[item, item, item],每次拖拽的选区可以叠加
        0: 返回数据为一个组，并且每次选择会把上次选择的清空
        */
        type: Number,
        required: false,
        default: 0,
      },
    },
    data () {
      return {
        mouseDown: false,
        startPoint: null,
        endPoint: null,
        // hoveredItem: [], /*正在选择中的元素，其实就是hoveredItem*/
        
        hoveredItem: [],/*正在选择中的元素，其实就是hoveredItem*/
        lastSelectedData: [], /*上次mouseup的被选中的数据, hover选中的数据又不一样*/
        bigSelected: [], /*上次mouseup的被选中的数据平铺结构*/
      }
    },
    computed: {
      selectionBox () {
        // Only set styling when necessary
        if (!this.mouseDown || !this.startPoint || !this.endPoint) return {}

        const clientRect = this.$el.getBoundingClientRect()
        const scroll = this.getScroll()

        // Calculate position and dimensions of the selection box
        const left = Math.min(this.startPoint.x, this.endPoint.x) - clientRect.left - scroll.x
        const top = Math.min(this.startPoint.y, this.endPoint.y) - clientRect.top - scroll.y
        const width = Math.abs(this.startPoint.x - this.endPoint.x)
        const height = Math.abs(this.startPoint.y - this.endPoint.y)

        // Return the styles to be applied
        return {
          left,
          top,
          width,
          height
        }
      },
      selectionBoxStyling () {
        // Only set styling when necessary
        if (!this.mouseDown || !this.startPoint || !this.endPoint) return {}

        const { left, top, width, height } = this.selectionBox

        // Return the styles to be applied
        return {
          left: `${left}px`,
          top: `${top}px`,
          width: `${width}px`,
          height: `${height}px`
        }
      }
    },
    watch: {
      hoveredItem (val) {
        this.$emit('change', val)
      },
      lastSelectedData (val) {
        this.$emit('change', val)
      },
      clearSection (val) {
        if(val) {
          this.hoveredItem = [];
          this.bigSelected = [];
          this.lastSelectedData = [];
          this.$emit('backClear')
        }
      }
    },
    methods: {
      getScroll () {
        // If we're on the server, default to 0,0
        if (typeof document === 'undefined') {
          return {
            x: 0,
            y: 0
          }
        }

        return {
          x: this.$el.scrollLeft || document.body.scrollLeft || document.documentElement.scrollLeft,
          y: this.$el.scrollTop || document.body.scrollTop || document.documentElement.scrollTop
        }
      },
      onMouseDown (event) {
        // Ignore right clicks
        if (event.button === 2) return

        // Register begin point
        this.mouseDown = true
        this.startPoint = {
          x: event.pageX,
          y: event.pageY
        }

        // Start listening for mouse move and up events
        window.addEventListener('mousemove', this.onMouseMove)
        window.addEventListener('mouseup', this.onMouseUp)
      },
      onMouseMove (event) {
        // Update the end point position
        if (this.mouseDown) {
          this.endPoint = {
            x: event.pageX,
            y: event.pageY
          }

          const children = this.$children.length
            ? this.$children
            : this.$el.children

          if (children) {
            this.hoveredItem = Array.from(children).filter((item) => {
              return this.isItemSelected(item.$el || item)
            });
            // this.hoveredItem = pushArrs;

          }
        }
      },
      onMouseUp (event) {
        // Clean up event listeners
        this.dealSelectData()
        window.removeEventListener('mousemove', this.onMouseMove)
        window.removeEventListener('mouseup', this.onMouseUp)
        // this.bigSelected.push(this.hoveredItem);
        // Reset state
        this.mouseDown = false
        this.startPoint = null
        this.endPoint = null
      },
      /*
      当前选中的数据处理与返回
      */
      dealSelectData () {
        let pushItems = this.hoveredItem;
        if(this.selectGroupType == 0) { //一次性的
          this.bigSelected = pushItems;
        }else if(this.selectGroupType == 1) { /*可累加，需要考虑反选，选中的数据是一个数组，1和2比较是否已经选中要跟上次mouseup的数据比*/
            console.log(this.bigSelected);
            pushItems.forEach((item)=>{
              console.log('alreay have ',this.bigSelected.indexOf(item));
              if(this.bigSelected.indexOf(item)>-1) {
                this.bigSelected.splice(this.bigSelected.indexOf(item), 1);
              }else{
                this.bigSelected.push(item);  
              }
            })
            
        }else if(this.selectGroupType == 2) { /*可累加，需要考虑反选，选中的数据是多个数组，按照选择顺利用数组存起来*/
            this.bigSelected.forEach((item)=>{

              pushItems.forEach((bItem, index)=>{
                if(item.indexOf(bItem)>-1){
                  item.splice(item.indexOf(bItem), 1);
                  pushItems.splice(index, 1);
                }
              })
              // if(this.bigSelected.indexOf(item)>-1){
              //   this.bigSelected.splice(this.bigSelected.indexOf(item), 1);
              // }
            })
            this.bigSelected.push(pushItems);
        }
        // this.hoveredItem = [];
        this.lastSelectedData = this.bigSelected;
      },
      isItemSelected (el) {
        // console.log('el.classList:',el.classList)
        // console.log('contains:',el.classList.contains)
        if (el.classList.contains(this.selectorClass)) {
          const boxA = this.selectionBox
          const boxB = {
            top: el.offsetTop,
            left: el.offsetLeft,
            width: el.clientWidth,
            height: el.clientHeight
          }

          return !!(
            boxA.left <= boxB.left + boxB.width &&
            boxA.left + boxA.width >= boxB.left &&
            boxA.top <= boxB.top + boxB.height &&
            boxA.top + boxA.height >= boxB.top
          )
        }

        return false
      }
    },
    mounted () {
      this.$children.forEach((child) => {
        child.$on('click', (event) => {
          const included = this.hoveredItem.find((item) => {
            return child.$el === item.$el
          })

          if (included) {
            this.hoveredItem = this.hoveredItem.filter((item) => {
              return child.$el !== item.$el
            })
          } else {
            this.hoveredItem.push(child)
          }
        })
      })
    },
    beforeDestroy () {
      // Remove event listeners
      window.removeEventListener('mousemove', this.onMouseMove)
      window.removeEventListener('mouseup', this.onMouseUp)

      this.$children.forEach((child) => {
        child.$off('click')
      })
    }
  }
</script>

<style>
  .vue-drag-select {
    position: relative;
    user-select: none;
  }

  .vue-drag-select-box {
    position: absolute;
    background: rgba(0, 162, 255, .4);
    z-index: 99;
  }
</style>
