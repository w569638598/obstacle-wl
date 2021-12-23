<template>
  <div class="hello">
    <!-- 工具栏 -->
    <div class="toolbar-container">
      <Toolbar :graph="graph" />
    </div>
    <!-- 画布 -->
    <div id="container" :style="{ height: canvasHeight }"></div>
    <!-- <button class="exportData" @click="exportData">导出</button> -->
    <div class="help">
    <span class="helpBtn iconhelp iconfont"></span>
    <ul>
      <li><span class="label">复制</span><span class="describe">ctrl+c</span></li>
      <li><span class="label">粘贴</span><span class="describe">ctrl+v</span></li>
      <li><span class="label">框选</span><span class="describe">ctrl+按下鼠标左键拖动</span></li>
      <li><span class="label">删除</span><span class="describe">delete</span></li>
      <li><span class="label">批量粘贴</span><span class="describe">多选后按ctrl+v</span></li>
      <li><span class="label">批量删除</span><span class="describe">多选后按delete</span></li>
      <li><span class="label">点选</span><span class="describe">ctrl+鼠标左键单击</span></li>
    </ul>
    </div>
  </div>
</template>

<script>
import Toolbar from "./components/NodesBar.vue";
import spanCss from "./common/style";
import { nodes } from "./common/nodesBar";
import createDom from "./common/createDom";
import { initGraph } from "./common/graph";
import { createImageNode } from "./common/transform";
/**
 * @param {HTMLElement} tagType 需要替换的目标元素标签 --span
 * @param {String} value 设置的值
 * @param {cell} cell graph的元素
 * @returs {HTMLElement}
 */
function changeTag(tagType, value = "", cell) {
  const node = document.createElement(tagType);
  if (tagType == "pre") {
    node.style.cssText = cell.store.data.data.defaultSpanStyle || spanCss;
    node.innerHTML = value;
  } else {
    node.innerHTML = value;
    node.style.cssText = cell.store.data.data.defaultSpanStyle || spanCss;
  }
  node.setAttribute("id", "currentTextNode");
  cell.setProp("data", {
    typeImgNo: cell.store.data.data.typeImgNo,
    textContent: value,
    defaultSpanStyle: cell.store.data.data.defaultSpanStyle,
  });
  return node;
}
export default {
  name: "HelloWorld",
  props: {
    height: String,
  },
  data() {
    return {
      graph: null,
      currentCell: null,
      history: null,
    };
  },
  computed: {
    canvasHeight() {
      if (this.height) return this.height;
      return window.innerHeight + "px";
    },
  },
  mounted() {
    this.graph = initGraph();
    this.graph.on("edge:click", ({ cell, e }) => {
      if (this.currentCell) {
        this.currentCell.attr("line", { stroke: "#7c68fc", strokeWidth: 2 });
        this.currentCell.removeTools();
      }
      let removeBtnCfg;
      if (cell.isEdge && cell.isEdge()) {
        cell.attr("line", { stroke: "red", strokeWidth: 3 });
        removeBtnCfg = { distance: "30%" };
      }
      cell.addTools({
        name: "custom-remove-button", // 工具名称
        args: removeBtnCfg, // 工具对应的参数
      });
      this.currentCell = cell;
    });
    this.graph.on("cell:unselected", ({ cell, node, e }) => {
      if (cell.isEdge && cell.isEdge()) {
        cell.attr("line", { stroke: "#7c68fc", strokeWidth: 2 });
      } else {
        const cellView = this.graph.findView(cell);
        cellView && cellView.removeClass(`${cell.shape}-selected`);
      }
      cell.removeTools();
      changePortsVisible(cell, false);
      Event.stopPropagation;
      Event.preventDefault;
      const html = cell.html instanceof Function ? cell.html() : cell.html;
      cell.setProp("html", () => {
        const childrenTag = html.children[1];
        if (childrenTag.nodeName !== "PRE") {
          const input = html.querySelector("textarea");
          html.querySelector("textarea").outerHTML = changeTag(
            "pre",
            input.value,
            cell
          ).outerHTML;
        }
        return html;
      });
    });
    this.graph.on("cell:removed", () => {
      const tooltipDom = document.getElementById("tooltip-container");
      if (tooltipDom) tooltipDom.style.display = "none";
    });
    this.graph.on("blank:click", () => {
      if (!this.currentCell) return;
      const cell = this.currentCell.id
        ? this.currentCell
        : this.currentCell.cell;
      if (cell.isEdge && cell.isEdge()) {
        cell.attr("line", { stroke: "#7c68fc", strokeWidth: 2 });
      }
      cell.removeTools();
      changePortsVisible(cell, false);
      const html = cell.html instanceof Function ? cell.html() : cell.html;
      cell.setProp("html", () => {
        const childrenTag = html.children[1];
        if (childrenTag.nodeName !== "PRE") {
          const input = html.querySelector("textarea");
          html.querySelector("textarea").outerHTML = changeTag(
            "pre",
            input.value,
            cell
          ).outerHTML;
        }
        return html;
      });
    });
    this.graph.on("edge:connected", (args) => {
      const edge = args.edge;
      const node = args.currentCell;
      const elem = args.currentMagnet;
      const portId = elem.getAttribute("port");
      // 触发 port 重新渲染
      node.setPortProp(portId, "connected", true);
      edge.zIndex = -1;
      // 更新连线样式
      edge.attr({
        line: {
          strokeDasharray: "",
          // targetMarker: 'classic'
          targetMarker: {
            name: "path",
          },
        },
      });
      edge.appendLabel({
        attrs: {
          label: {
            text: "",
          },
        },
      });
    });
    this.graph.on("node:click", ({ cell }) => {
      this.currentCell = cell;
    });
    this.graph.on("node:dblclick", ({ cell, e }) => {
      Event.stopPropagation;
      const html = cell.html instanceof Function ? cell.html() : cell.html;
      cell.setProp("html", () => {
        const childrenTag = html.children[1];
        if (childrenTag.nodeName !== "TEXTAREA") {
          const pre = html.querySelector("pre");
          html.querySelector("pre").outerHTML = changeTag(
            "textarea",
            pre.innerText,
            cell
          ).outerHTML;
        }
        return html;
      });
      let removeBtnCfg;
      if (cell.isNode()) {
        const cellView = this.graph.findView(cell);
        cellView.addClass(`${cell.shape}-selected`);
        removeBtnCfg = { x: -28, y: -5 };
      }
      cell.addTools({
        name: "custom-remove-button", // 工具名称
        args: removeBtnCfg, // 工具对应的参数
      });
      this.currentCell = cell;
      document.getElementById("currentTextNode").focus();
    });
    this.graph.on("node:mouseenter", ({ node }) => {
      Event.stopPropagation;
      const data = node.getData();
      if (data && data.disableMove) {
        changePortsVisible(node, false);
      } else changePortsVisible(node, true);
    });
    this.graph.on("node:mouseleave", ({ node }) => {
      Event.stopPropagation;
      const tooltipDom = document.getElementById("tooltip-container");
      changePortsVisible(node, false);
      if (tooltipDom) tooltipDom.style.display = "none";
    });
    this.history = this.graph.history;
    let copyState = false;
    let that = this;
    document.addEventListener("keydown", (e) => {
      if (e.key == "Delete") {
        var selectedNodeArr = this.graph.getSelectedCells();
        selectedNodeArr.forEach((e) => {
          e.remove();
        });
        return;
      }
      if (window.event.ctrlKey && e.key === "z") {
        const textarea = document.querySelectorAll(".nodeBox textarea");
        if (textarea.length) return;
        this.history.undo();
        return;
      }
      if (window.event.ctrlKey && window.event.shiftKey && e.key === "Z") {
        this.history.redo();
        return;
      }
      if (window.event.ctrlKey && e.key === "c") {
        copyState = true;
        return;
      }
      if (window.event.ctrlKey && e.key === "v" && copyState) {
        // if (!this.currentCell) return;
        const nodeArr = this.graph.getSelectedCells();
        for (let i = 0; i < nodeArr.length; i++) {
          const cell = nodeArr[i].store.data;
          let createCell;
          for (let i = 0; i < nodes.length; i++) {
            const item = nodes[i];
            if (!item.typeImgNo) break;
            if (cell.data.typeImgNo === item.typeImgNo) {
              createCell = JSON.parse(JSON.stringify(item));
              break;
            }
          }
          createCell.label = cell.data.textContent;
          createCell.tooltip = cell.data.textContent;
          createCell.x = cell.position.x + 10;
          createCell.y = cell.position.y + 10;
          createCell.width = cell.size.width;
          createCell.height = cell.size.height;
          // console.log(createCell)
          let json = createImageNode(createCell);
          this.graph.addNode(json);
        }
        return;
      }
    });
  },
  components: {
    Toolbar,
  },
  methods: {
    renderData(cells) {
      cells.forEach((e) => {
        if (e.shape === "html") {
          nodes.forEach((item) => {
            if (e.data.typeImgNo === item.typeImgNo) {
              e.html = () =>
                createDom({
                  tooltip: e.data.textContent,
                  urlStr: item.urlStr,
                  defaultSpanStyle: item.defaultSpanStyle,
                });
            }
          });
        }
      });
      this.graph.fromJSON(cells);
      const nodesss = this.graph.getNodes();
      nodesss.forEach((e) => {
        changePortsVisible(e, false);
      });
    },
    exportData(cb) {
      if (cb instanceof Function) cb(this.graph.toJSON(), this.graph);
      return this.graph.toJSON();
    },
    getGraph() {
      return this.graph;
    },
  },
};
const changePortsVisible = (node, visible) => {
  const ports = document.querySelectorAll(
    `g[data-cell-id="${node.id}"] .x6-port-body`
  );
  ports.forEach((port) => {
    port.style.visibility = visible ? "visible" : "hidden";
  });
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="less">
@import url('//at.alicdn.com/t/font_1471400_k4dlg7u3p1q.css');
ul,li{
  margin: 0;
  padding: 0;
  list-style: none;

}
.exportData {
  position: fixed;
  top: 10%;
  left: 50%;
}
.hello {
  display: flex;
  height: 100%;
  .toolbar-container {
    width: 200px;
  }
}
#container {
  flex: 1;
  height: 100% !important;
  min-height: 10px;
}
.help{
  float: left;
  position: fixed;
  right: 6px;
  font-size: 14px;
  .iconhelp{
    float: right;
    margin-right: 10px;
    margin-top: 10px;
    font-size: 22px;
    display: block;
    cursor: help;
    &:hover + ul{
      left: 0;
      opacity: 1;
    }
  }
  ul{
    background: white;
    padding: 10px 16px;
    border:solid 1px #eee;
    clear: both;
    position: relative;
    top: 10px;
    transition: ease-in-out .3s left, ease-in-out .4s opacity .1s;
    left: 100%;
    opacity: 0;
  }
  li{
    text-align: left;
    margin: 15px 0;
    line-height: 180%;
    .label{
      background: #efefef;
      padding: 5px;
      border: solid 1px #e0e0e0;
      margin-right: 6px;
    }
  }
}
</style>