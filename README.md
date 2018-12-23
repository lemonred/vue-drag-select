<p align="center">
  <img src="https://raw.githubusercontent.com/stephan281094/vue-drag-select/master/images/screenshot.png"
    alt="Screenshot">
</p>

# Vue Drag to Select Component
A Vue component for drag selecting elements. Inspired by [react-drag-select](https://github.com/pablofierro/react-drag-select).
support select and select inverse and group select 

<p align="center">
group select
  <img src="https://github.com/lemonred/vue-drag-select/master/images/select-muliti.png"
    alt="Screenshot">
</p>

## Demo
* Visit the demo [here](http://vue-drag-select-example.now.sh).

## Usage
* Add `vue-drag-select` to your project:

```bash
$ yarn add vue-drag-select # or npm i --save vue-drag-select
```

* Import the component and add it to your template:

```js
import DragSelect from 'vue-drag-select/src/DragSelect.vue'

export default {
  components: {
    'drag-select-container': DragSelect
  }
}
```

* Wrap the items that you want to be selectable in the `drag-select-container`
  with a `selectorClass` property and a scoped slot:

```vue
<template>
  <drag-select-container selectorClass="itemToBeSelected">
    <template slot-scope="{ selectedItems }">
      <!-- Your items here -->
    </template>
  </drag-select-container>
</template>
```
## update add multiselect  and select revert(20181204)
* props selectGroupType=1 multiselect and select revert
* props selectGroupType=2 multiselect and select revert and group tag
* Then write your own logic to make items look selected. For instance by applying a class.

props hasClearSection can clear what you select

```
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
```

## License
MIT
