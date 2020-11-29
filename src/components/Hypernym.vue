<template>
  <div>
      <v-chip class="ma-2" disabled label color="white" font-weight="700" text-color="black">
        <strong>Hierarchy:</strong>
      </v-chip>
    <vue-tree-list
      @drop="updateTree()"
      :model="clusterTree"
      @click="selectCluster"
      default-tree-node-name="new node"
      default-leaf-node-name="new leaf"
      v-bind:default-expanded="true"
    >
      <template v-slot:leafNameDisplay="slotProps">
        <span>
          {{ slotProps.model.text }} <span class="muted">#{{ slotProps.model.id }}</span>
        </span>
      </template>
    </vue-tree-list>
  </div>
</template>


<script>
  import { VueTreeList, Tree, TreeNode } from 'vue-tree-list'
  import jsonData from '../data/clusters.json'
  export default {
    name: "Hypernym",
    components: {
      VueTreeList
    },
    props: {
        clusterList: Object
    },
    data() {
      return {
        newTree: {},
        clusterTree: {},
        data: new Tree(!this.json || this.json == "${data}"
        ? jsonData
        : JSON.parse(unescape(this.json).replace("\u00e2\u20ac\u2122", "'"))),
      }
    },
    created() {
        this.clusterTree = new Tree(Object.values(this.clusterList));
        // this.clusterTree = new Tree(this.clusterList);
    },
    watch: {
      clusterList: function(newVal, oldVal) {
        let oldClusters = Object.keys(oldVal);
        let newClusters = Object.keys(newVal);

        if (this.arrayEquals(oldClusters, newClusters)) {
          return;
        }
        

        let intersection = oldClusters.filter(x => newClusters.includes(x));
        let toBeDeleted = oldClusters.filter(x => !intersection.includes(x));
        let toBeAdded = newClusters.filter(x => !intersection.includes(x));


        // check whether to add or to delete a node from the hypernyms
        if (toBeAdded.length > 0) {
          this.addMissingNode(this.clusterList[toBeAdded]);          
        } else {
          this.deleteExtraNode(oldVal[toBeDeleted]);
        }
        
        
      }
    },
    methods: {
      arrayEquals(a, b) {
        return Array.isArray(a) &&
          Array.isArray(b) &&
          a.length === b.length &&
          a.every((val, index) => val == b[index]);
      },

      deleteExtraNode(nodeName) {
        let stack = [];
        let explored = new Set();
        stack.push(this.clusterTree);

        explored.add(this.clusterTree);

        // find with dfs the node to delete
        while (!stack.length == 0) {
          let t = stack.pop();
          explored.add(t);

          if (t.id == nodeName.id) {
            // if the node to delete has children, move the children to his parents
            if (t.children) {
              t.children.forEach(x => t.parent.addChildren(x));
            }
            t.remove();
            break;
          } 

          // add the children in the stack (for the dfs)
          if (t.children) {
            t.children.forEach(element => {
            stack.push(element);
            explored.add(element);
          });
          }
          
        }
      },


      addMissingNode(nodeName) {
        if (nodeName) {
          var node = new TreeNode(nodeName);
          this.clusterTree.addChildren(node);

          let start = parseInt(nodeName.id.split('-')[0]);
          for (var i = 0; i < this.clusterTree.children.length; i++) {
            let child = this.clusterTree.children[i];
            let childStart = parseInt(child.id.split('-')[0]);
            if (start < childStart) {
              break
            }
          }
          
          if (i < this.clusterTree.children.length) {
            this.clusterTree.children[this.clusterTree.children.length - 1].insertBefore(this.clusterTree.children[i]);
          }
          
        }
        
      },

      
      addNode() {
        var node = new TreeNode({ name: 'new node', isLeaf: false })
        if (!this.clusterTree.children) this.clusterTree.children = []
        this.clusterTree.addChildren(node)
      },

    
      getNewTree() {
        var vm = this
        function _dfs(oldNode) {
          var newNode = {}
          for (var k in oldNode) {
            if (k !== 'children' && k !== 'parent') {
              newNode[k] = oldNode[k]
            }
          }
          if (oldNode.children && oldNode.children.length > 0) {
            newNode.children = []
            for (var i = 0, len = oldNode.children.length; i < len; i++) {
              newNode.children.push(_dfs(oldNode.children[i]))
            }
          }
          return newNode
        }
        vm.newTree = _dfs(vm.clusterTree)
      },

      updateTree() {
          this.getNewTree();
          console.log(this.clusterTree);
          return this.$emit('updateTree', this.newTree);
      },

      selectCluster(params) {
        console.log(params);
        return this.$emit("candidateSelected", params.id);
      }
    }
  }
</script>


<style lang="less" rel="stylesheet/less">
  .vtl {
    .vtl-drag-disabled {
      background-color: #d0cfcf;
      &:hover {
        background-color: #d0cfcf;
      }
    }
    .vtl-disabled {
      background-color: #d0cfcf;
    }
  }
</style>

<style lang="less" rel="stylesheet/less" scoped>
  .icon {
    &:hover {
      cursor: pointer;
    }
  }
  .muted {
    color: gray;
    font-size: 80%;
  }
</style>