<template>
  <div class="node-bar-container">
    <ul class="nodes-bar">
      <li v-for="(node, index) in nodes" :key="index" :title="node.label" :class="[index > 11 ? 'fullWidth': '']">
        <img
          v-if="node.shape == 'image'"
          :src="node.url"
          alt=""
          @mousedown="startDrag(node, $event)"
        />
        <p>{{node.label}}</p>
      </li>
    </ul>
  </div>
</template>
<script>
import { nodes } from "../common/nodesBar";
import { getImageNode } from "../common/transform";
import { Addon } from "@antv/x6";
export default {
  props: ["graph"],
  data() {
    return {
      nodes,
    };
  },
  watch: {
    graph() {
      this.initDnd();
    },
  },
  methods: {
    initDnd() {
      this.dnd = new Addon.Dnd({
        target: this.graph,
        validateNode() {
          return true;
        },
      });
    },
    startDrag(currentTarget, e) {
      const {
        actionType,
        shape,
        label,
        urlStr,
        ports,
        group,
        yAxis,
        typeImgNo,
        width,
        height
      } = currentTarget;
      let json = getImageNode({
        shape,
        tooltip: label,
        size: { width: 100, height: 50 },
        actionType,
        urlStr,
        ports,
        group,
        yAxis,
        typeImgNo,
        width,
        height
      });

      const node = this.graph.createNode(json);
      if (!this.freeze) this.dnd.start(node, e);
    },
  },
  created() {},
};
</script>
<style lang="less" scoped>
.nodes-bar{
  p{
    margin: 0;
  }
}
.node-bar-container{
  height: 100%;
  overflow: auto;
  background: #f8f8f8;
}
ul,li{
  margin: 0;
  padding: 0;
  list-style: none;
}
ul{
  height: 100%;
  .fullWidth{
    width: 90%;
    margin: auto;
    padding: 10px 0;
    clear: both;

  }
  li:nth-child(3),li:nth-child(4),li:nth-child(11),li:nth-child(12){
    border-bottom: solid 1px #ccc;
  }
  li:nth-child(12){
    margin-bottom: 20px;
  }
  li{
    width: 50%;
    float: left;
    box-sizing: border-box;
    padding: 10px;
  }
  img{
      width: 61%;
    }
}

</style>
