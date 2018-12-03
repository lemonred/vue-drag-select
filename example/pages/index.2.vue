<template>
  <div class="app">
    <h1>Vue Drag Select Example</h1>
    <button @click="changeClear()">清除选区</button>
    <no-ssr>
      <div>
      <!-- <drag-select-container selectorClass="item" :clearSection="clearSection" @backClear="backClear">
        <template slot-scope="{ bigSelected }">
          <div
            v-for="item in 50"
            :key="item"
            :class="getClasses(item, bigSelected, 0)"
            :data-item="item"
          >
            sItem
          </div>
          {{bigSelected}}
        </template>
      </drag-select-container> -->
1
      <!-- <drag-select-container selectorClass="item" :clearSection="clearSection" @backClear="backClear" :selectGroupType="1">
        <template slot-scope="{ seletedItem }">
          <div
            v-for="item in 50"
            :key="item"
            :class="getClasses(item, seletedItem, 1)"
            :data-item="item"
          >
            sItem{{item}}
          </div>
          {{seletedItem}}
        </template>
      </drag-select-container> -->
2
      <drag-select-container selectorClass="item" :clearSection="clearSection" @backClear="backClear"  :selectGroupType="2">
        <template slot-scope="{ seletedItem, hoveredItem }">
          <div
            v-for="item in 50"
            :key="item"
            :class="getClasses(item, seletedItem, 2, hoveredItem)"
            :data-item="item"
          >
            sItem 
          </div>
          {{seletedItem}}
        </template>
      </drag-select-container>
      </div>
     
    </no-ssr>
  </div>
</template>

<script>
  // import DragSelect from './../../dist/vue-drag-select.js'
  import DragSelect from '../../src/DragSelect.vue'
  import NoSSR from 'vue-no-ssr'
  console.log(DragSelect)
  export default {
    name: 'home',
    data() {
      return {
        clearSection: false
      }
    },
    components: {
      'drag-select-container': DragSelect,
      'no-ssr': NoSSR
    },

    methods: {
      backClear() {
        this.clearSection = false;  
      },
      changeClear() {
        this.clearSection = true;
      },
      /*
      set active class

      */
      getClasses (item, bigSelected, type, hoveredItems) {
        let returnItem;
        const isActive = !!(bigSelected.find((selectedItem) => {
          
          if(type==0 || !type || type==1) { /*简单数组*/
              return parseInt(selectedItem.dataset.item, 10) === item;
          }else if(type==2) {
            console.log(selectedItem)
          for(let i = 0; i < selectedItem.length; i++) {
            console.log(selectedItem[i].dataset.item,item)
            if(parseInt(selectedItem[i].dataset.item, 10) === item){
              returnItem = selectedItem[i];
              break;
            }
          }
          return returnItem;
          }

        }))

        const isHover = !!(hoveredItems.find((hItem) => {
          return parseInt(hItem.dataset.item, 10) === item;
        }))
        

        return {
          item: true,
          active: isActive,
          hovered: isHover
        }
      }
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
    color: #fff;
  }
</style>
