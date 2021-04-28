<template>
  <div>
    <vue-tree-list class="hypernym"
      ref="package"
      :model="tree"
      v-bind:default-expanded="true"

    >
      <template v-slot:leafNameDisplay="slotProps">
        <span>
          {{ slotProps.model.text }}
        </span>
      </template>
    </vue-tree-list>
  </div>
</template>


<script>
import { VueTreeList } from 'vue-tree-list'
export default {
    name: "DisplayTree",
    components: {
        VueTreeList
    },
    props: {
        tree: Object()
    },
}
</script>