<template>
  <div>
      <v-chip class="ma-2" disabled label color="white" font-weight="700" text-color="black">
        <strong>Hierarchy:</strong> 
      </v-chip>
      <v-btn class="reset" @click="reset()" absolute large append color="#B0BEC5">Reset</v-btn>
    <vue-tree-list class="hypernym"
      ref="package"
      :model="clusterTree"
      @click="selectCluster"
      @change-name="onChangeName"
      @start-focus="onStartInput"
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
// import { VButton } from "vuetify/lib";
  import { VueTreeList, Tree, TreeNode } from 'vue-tree-list'
  export default {
    name: "Hypernym",
    components: {
      VueTreeList,
      // VButton
    },
    props: {
        clusterList: Object,
        hierarchy: Object,
        mentions: Array
    },
    data() {
      return {
        newTree: {},
        clusterTree: {},
        rendering: false,
        assignedMentions: []
      }
    },
    created() {
     Object.values(this.clusterList).forEach((value) => {
          value.addTreeNodeDisabled = true;
          value.addLeafNodeDisabled = true;
          value.editNodeDisabled =  false;
          value.delNodeDisabled = true;
          value.name = value.text;
        });
        this.clusterTree = new Tree(Object.values(this.clusterList));
          // this.clusterTree = new Tree(this.clusterList);
      
    },

    watch: {
      mentions: function(newVal, oldVal) {
        if (!this.arrayEquals(newVal, oldVal)) {
          this.assignedMentions = newVal;
        }
      },

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
        
        if (toBeDeleted.length > 0) {
          let clusterToBeDeleted = oldVal[toBeDeleted];
          let [start, end] = clusterToBeDeleted.id.split('-');
          
          let newCluster = this.getClusterMention(start, end);
          this.deleteExtraNode(oldVal[toBeDeleted].id, newCluster);
        }
        if (toBeAdded.length > 0) {
          this.addMissingNode(this.clusterList[toBeAdded]);          
        } 
        
      
      }
    },
    methods: { 
      onStartInput: function() {
        this.$emit('deactivateEvents')
      },
      onChangeName: function(params) {     
        if (params.newName.startsWith(params.oldName.trim())) {
          var node = this.dfs(params.id);
          node.name = params.newName;
          node.text = params.newName;
        }
        this.$emit('activateEvents')
      },

      reset: function() {
        let flatHierarchy = Array();

        for (var clusterId in this.clusterList) {
          var oldNode = this.dfs(clusterId);
          var newNode = {};
          for (var k in oldNode) {
            if (k !== 'children' && k !== 'parent') {
              newNode[k] = oldNode[k];
            }
          }
          flatHierarchy.push(newNode);
        }

        this.clusterTree = new Tree(flatHierarchy);
      },

      forceRerender: function() {
        this.$emit('forceRerender');
      },

      getClusterMention(start, end) {
        let mention = this.assignedMentions.find(x => x.start == start && x.end == end);
        return mention.clustId;
      },

      arrayEquals(a, b) {
        return Array.isArray(a) &&
          Array.isArray(b) &&
          a.length === b.length &&
          a.every((val, index) => val == b[index]);
      },


      dfs(nodeName) {
        let stack = [];
        let explored = new Set();
        stack.push(this.clusterTree);

        explored.add(this.clusterTree);

         // find with dfs the node to delete
        while (!stack.length == 0) {
          let t = stack.pop();
          explored.add(t);

          if (t.id == nodeName) {
            return t;
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

      deleteExtraNode(nodeName, clusterDest) {
        let deleted = this.dfs(nodeName);
        let dest = this.dfs(clusterDest);

        if (dest == undefined) {
          dest = this.dfs(deleted.pid);
        }
        

        // let newCluster = this.dfs(toBeDeleted).parent.id;
        let chidlren = [];
        if (deleted.children != null && deleted.children.length > 0) {
          deleted.children.forEach(x => chidlren.push(x.id));  
        }
        

        if (chidlren.includes(dest.id)) {
          dest = this.dfs(deleted.parent.id);
        }

        if (deleted.children) {
          deleted.children.forEach(x => dest.addChildren(x));
        }
        deleted.remove();

      },

      // onChangeName() {

      // },
      addMissingNode(nodeName) {
        if (nodeName) {
          nodeName.addTreeNodeDisabled = true;
          nodeName.addLeafNodeDisabled = true;
          nodeName.delNodeDisabled = true;
          nodeName.editNodeDisabled = false;
          nodeName.name = nodeName.text;
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
        var node = new TreeNode({ name: 'new node', isLeaf: false, editNodeDisabled: false })
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
          // console.log(this.clusterTree);
          return this.$emit('updateTree', this.newTree);
      },

      selectCluster(params) {
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
    .vtl-active {
      outline: 2px dashed pink;
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
    display: none;
  }

  
</style>