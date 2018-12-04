<template>
  <div class="app">
    <h1>Vue Drag Select Example</h1>
    
    <no-ssr>
      <div>
      <drag-select-container selectorClass="item" :hasClearSection=true  :selectGroupType="2">
        <template slot-scope="{ itemsStatus }">
          <div
            v-for="item in 50"
            :key="item"
            :class="getClasses(item, itemsStatus, 2)"
            :data-item="item"
          >
            sItem 
          </div>
          {{itemsStatus}}
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
      getClasses (item, itemStatus, type) {
        let returnItem;
        console.log(item.$el);
        let hovered = false, active = false;
        if(itemStatus && itemStatus[item-1]){
          hovered = itemStatus[item-1].hovered;
          active =  itemStatus[item-1].active;
        }
        return {
          item: true,
          active: active,
          hovered: hovered
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
    color: #000;
  }
</style>
