<template>
  <div 
    class="vue-drag-select" 
    @mousedown="onMouseDown">
    <slot 
      :itemsStatus="Items" 
      :groupArr="groupArr"/>
    <div 
      v-if="mouseDown" 
      class="vue-drag-select-box"
      :style="selectionBoxStyling"></div>
    
    <div 
      class="vue-drag-selected-group-warp" 
      v-for="(item) in groupArr" 
      :style="groupBoxStyling[item]"
      v-if="selectGroupType==2 && item!=0"
      @click.stop="moveSelectGroup(item)"
      :key="'g'+item">
      <i 
        class="el-icon-remove" 
        @click.stop="moveSelectGroup(item)"></i>
    </div>
    <!--清除选区-->
    <button 
      class="el-button el-button--default el-button--mini"
      @click="changeClear()" 
      v-if="hasClearSection">{{ clearSectionText }}</button>
  </div>
</template>

<script>
  export default {
    name: 'VueDragSelect',
    props: {
      selectorClass: {
        type: String,
        required: true
      },
      hasClearSection: { /*清除选区标志，如果为true则清除当前的选择*/
        type: Boolean,
        required: false,
        default: false
      },
      clearSectionText: {
        type: String,
        required: false,
        default: '清除选区'
      },
      selectGroupType: { 
        /*
        2:是多个组，返回选区类型 [{selected: '', hovered: '', index: '', groupIndex:0}],且选区被浮层遮住，有单独的清除按钮
        1: 返回数据为一个组，返回选区[{selected: '', hovered: ''}],每次拖拽的选区可以叠加
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
        hoveredItem: [],/*正在选择中的元素，其实就是hoveredItem*/
        hoveredIndex: [],
        Items: [],
        groupIndex: 0,
        groupArr: [],
        groupBoxStyling: {},
        selectedGroupBox: {}, //被选中的群组box
        notSelect: false, /*不是目标元素*/
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
      },

      // groupBoxStyling() {
      //  const { left, top, width, height } = this.selectedGroupBox

      //   // Return the styles to be applied
      //   return {
      //     left: `${left}px`,
      //     top: `${top}px`,
      //     width: `${width}px`,
      //     height: `${height}px`
      //   } 
      // },
    },
    watch: {
      hoveredItem (val) {
        this.$emit('change', val)
      },
      selectedGroupBox(val) {
        if(!val){
          return {}
        }
       const { left, top, width, height,groupId } = val

        // Return the styles to be applied
        this.groupBoxStyling[groupId] = {
          left: `${left+2}px`,
          top: `${top+2}px`,
          width: `${width-4}px`,
          height: `${height-4}px`
        } 
      }
    },
    methods: {
      getMaxOfArray(numArray) {
        return Math.max.apply(null, numArray);
      },
      getMinOfArray(numArray) {
        return Math.min.apply(null, numArray);
      },      
      /*删除被选中的群组*/
      moveSelectGroup(groupId)  {
        for(let i = 0 ; i < this.Items.length; i++) {
          if(this.Items[i].groupIndex==groupId) {
            this.$set(this.Items[i], 'active', false)
            // this.$delete(this.Items[i], 'groupIndex')

            this.$set(this.Items[i], 'groupIndex', 0)

            if(this.groupArr.indexOf(groupId)>-1) {
              this.groupArr.splice(this.groupArr.indexOf(groupId), 1);
            }
          }
        }
      },
      getselectedGroupBox () {

        let left = 0, top = 0, width = 0, height = 0, lefts = [], tops= [], rights=[], bottoms=[];
        let groupId = this.groupArr[this.groupArr.length-1];
        if(!groupId) {
          return
        }
        this.hoveredItem.forEach((item, index)=>{
          tops.push(item.offsetTop)
          lefts.push(item.offsetLeft)
          rights.push(item.offsetLeft + item.clientWidth)
          bottoms.push(item.offsetTop + item.clientHeight)
        })
        top =this.getMinOfArray(tops);
        left =this.getMinOfArray(lefts);
        width =this.getMaxOfArray(rights)-left;
        height =this.getMaxOfArray(bottoms)-top;

        this.selectedGroupBox = {
          left,
          top,
          width,
          height,
          groupId
        }
      },
      /*清除选区*/
      changeClear() {
        this.hoveredItem = [];
        this.hoveredIndex = [];
        this.Items = [];
        this.groupIndex = 0;
        this.groupArr = [];
        this.$emit('backClear')
      },
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
        this.hoveredItem = [];
        this.hoveredIndex = [];
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
          this.notSelect = false;
        if(event.target.className == 'vue-drag-selected-group-warp' || event.target.parentNode.className == 'vue-drag-selected-group-warp'){
          this.notSelect = true;
          this.hoveredItem = [];
          this.hoveredIndex = [];
          return
        }
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
            this.hoveredItem = [];this.hoveredIndex = [];
            Array.from(children).filter((item, index) => {
              item.dataset.dragIndex =index;
              if(!this.Items[index]) {
                this.$set(this.Items, index, {});
              }
              this.$set(this.Items[index], 'hovered', false)
              this.$set(this.Items[index], 'index', index)
              let itemSeleted = this.isItemSelected(item.$el || item), returnIndex;
              if(itemSeleted) { /*hovered*/
              
                returnIndex = index;
                
                this.$set(this.Items[index], 'hovered',true)
                this.hoveredItem.push(item.$el || item);
                this.hoveredIndex.push(index);
              }
            });
           
          }
        }
      },
      onMouseUp (event) {
        // Clean up event listeners
        if(event.target.className == 'vue-drag-selected-group-warp' || event.target.parentNode.className == 'vue-drag-selected-group-warp'){
          this.notSelect = true;
          this.hoveredItem = [];
          this.hoveredIndex = [];
          
        }
        if(this.notSelect === false){
          this.dealSelectData()
        }

        window.removeEventListener('mousemove', this.onMouseMove)
        window.removeEventListener('mouseup', this.onMouseUp)

        // Reset state
        this.mouseDown = false
        this.startPoint = null
        this.endPoint = null
      },
      dealSelectData () {
        this.groupIndex += 1;
        if(this.selectGroupType == 0) {
          this.Items.forEach((item, index)=> {
            // item.index = item.dataset.index;
            item.hovered = false;
            item.active = false;
            if(this.hoveredIndex.indexOf(index)>-1){
              item.active = true;
            }
          })
        }else if(this.selectGroupType == 1) {
          this.Items.forEach((item, index)=> {
            // item.index = item.dataset.index;
            item.hovered = false;
            // item.active = false;
            if(this.hoveredIndex.indexOf(index)>-1){
              item.active = true;
            }
          })
        }else if(this.selectGroupType == 2) {
          this.Items.forEach((item, index)=> {
            // item.index = item.dataset.index;
            item.hovered = false;
            // item.active = false;
            if(this.hoveredIndex.indexOf(index)>-1) {
              if(item.active) {
                item.groupIndex = 0;
                
              }else {
                item.groupIndex = this.groupIndex;
                if(this.groupArr.indexOf(this.groupIndex)==-1){
                  this.groupArr.push(this.groupIndex);
                }
              }
              
              item.active = !item.active;

            }
          })
          /*获取刚选中的区域固定区域*/
          this.getselectedGroupBox();
        }
        
      },
      isItemSelected (el) {

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
  .vue-drag-select{
    position: relative;
    user-select: none;
  }
  .vue-drag-selected-group-warp{
    background:  rgb(55, 108, 215, 0.5);
    /* background: rgba(175, 223, 238, 0.2); */
    border: 2px solid #333;
    z-index: 98;
    position: absolute;

  }
  .vue-drag-selected-group-warp .el-icon-remove{
    font-size: 30px;
    color: #fff;
  }
  .vue-drag-select-box {
    position: absolute;
    background: rgba(0, 162, 255, .4);
    z-index: 99;
  }
</style>
