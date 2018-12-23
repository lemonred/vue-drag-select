<template>
  <div class="app">
    <h1>Vue Drag Select Example</h1>
    
    <no-ssr>
      <div>
          <drag-select-container 
              selector-class="item" 
              :has-clear-section=true 
              clear-section-text = '重新设置'
              :select-group-type="2">
              <template slot-scope="{ itemsStatus,groupArr }">
                <div 
                  v-for="(item, index) in boxItems" 
                  :key="index" 
                  :class="getClasses(item, itemsStatus, 2, groupArr)"
                  :data-index="item.index" 
                  :data-col="item.col" 
                  :data-row="item.row" 
                  :style="{width: 600/colSpan-2, height: 600/rowSpan-2}">
                  <span v-if="itemsStatus[item.index]">{{ itemsStatus[item.index].groupIndex }}</span>
                </div>
              </template>
            </drag-select-container>
      </div>
     
    </no-ssr>
  </div>
</template>

<script>

  import DragSelect from '../../src/DragSelect.vue'
  import NoSSR from 'vue-no-ssr'

  export default {
    name: 'home',
    data() {
      return {
        clearSection: false,
        rowSpan: 10, /* 行 */
        colSpan: 5, /* 列 */
      }
    },
    components: {
      'drag-select-container': DragSelect,
      'no-ssr': NoSSR
    },
    computed: {
      boxItems() { /*所有选区中的元素*/
        let arr = [], alen = this.rowSpan *  this.colSpan;
        for(let i = 0 ; i < alen; i++){
          arr.push({
            col: parseInt(i/this.colSpan), /*第几行*/
            row: i % this.colSpan, /*第几列*/
            index: i
          })
        }
        return arr
      }
    },
    methods: {


      /*
      set active class
      */
    getClasses (item, itemStatus, type, groupArr) {
      this.items = itemStatus;
      this.groupArr = groupArr;
      let returnItem;
      let hovered = false, active = false;
      if(itemStatus && itemStatus[item.index]){
        hovered = itemStatus[item.index].hovered;
        active =  itemStatus[item.index].active;
      }
      return {
        item: true,
        active: active,
        hovered: hovered
      }
    },
    }
  }
</script>

<style>
  /* Basic reset */
  *,
  *:before,
  *:after {
    box-sizing: inherit;
  }

  html {
    box-sizing: border-box;
    user-select: none;
  }

  html,
  body {
    margin: 0;
    padding: 0;
    min-height: 100vh;
  }

  body {
    font: 16px / 1.5 'Helvetica Neue', sans-serif;
    padding: 5%;
  }

  /* Custom styling */
  .item {
    display: inline-flex;
    min-width: 80px;
    height: 100px;
    margin-right: 10px;
    margin-bottom: 10px;
    background-color: #ddd;
    justify-content: center;
    align-items: center;
    text-transform: uppercase;
    letter-spacing: 3px;
    font-size: 10px;
  }

  .item.active, .item.active.hovered {
    background-color: rgb(0, 162, 255);
    color: #fff;
  }
  .item.hovered {
    background-color: yellow;
    color: #000;
  }
</style>
