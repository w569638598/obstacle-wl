<template>
  <div class="hello">
    <!-- 工具栏 -->
    <div class="toolbar-container">
      <Toolbar :graph="graph" />
    </div>
    <!-- 画布 -->
    <div id="container"></div>
    <button class="exportData" @click="exportData">导出</button>
  </div>
</template>

<script>
import Toolbar from "./components/NodesBar.vue";
import spanCss from "./common/style";
import { nodes } from "./common/nodesBar";
import createDom from "./common/createDom";
import { initGraph } from "./common/graph";

/**
 * @param {HTMLElement} tagType 需要替换的目标元素标签 --span
 * @param {String} value 设置的值
 * @param {cell} cell graph的元素
 * @returs {HTMLElement}
 */
function changeTag(tagType, value = "", cell) {
  const node = document.createElement(tagType);
  if (tagType == "pre") {
    node.style.cssText = spanCss;
    node.innerHTML = value;
  } else {
    node.innerHTML = value;
    node.style.cssText = spanCss;
  }
  node.setAttribute("id", "currentTextNode")
  cell.setProp("data", {
    typeImgNo: cell.store.data.data.typeImgNo,
    textContent: value,
  });
  return node;
}
export default {
  name: "HelloWorld",
  props: {
    msg: String,
  },
  data() {
    return {
      graph: null,
      currentCell: null,
    };
  },
  watch: {},
  mounted() {
    this.graph = initGraph();
    this.graph.on("edge:click", ({ cell, e }) => {
      if (this.currentCell) {
        this.currentCell.attr("line", { stroke: "#7c68fc", strokeWidth: 2 });
        this.currentCell.removeTools();
      }

      let removeBtnCfg;
      if (cell.isEdge()) {
        cell.attr("line", { stroke: "red", strokeWidth: 3 });
        removeBtnCfg = { distance: "30%" };
      }
      cell.addTools({
        name: "custom-remove-button", // 工具名称
        args: removeBtnCfg, // 工具对应的参数
      });
      this.currentCell = cell;
    });

    if (localStorage.getItem("cell")) {
      let cells = JSON.parse(localStorage.getItem("cell")).cells;

      cells.forEach((e) => {
        if (e.shape === "html") {
          nodes.forEach((item) => {
            if (e.data.typeImgNo === item.typeImgNo) {
              e.html = () =>
                createDom({ tooltip: e.data.textContent, urlStr: item.urlStr });
            }
          });
        }
      });
      this.graph.fromJSON(cells);
      const nodesss = this.graph.getNodes();
      nodesss.forEach((e) => {
        changePortsVisible(e, false);
      });
    }

    this.graph.on("cell:unselected", ({ cell, node, e }) => {
      if (cell.isEdge()) {
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
      const cell = this.currentCell;
      if (cell.isEdge()) {
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
      this.currentCell = null
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
            name: "path"
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
    this.graph.on("node:mousedown", ({ cell, e }) => {
      if (this.currentCell) {
        if(this.currentCell.id == cell.id)return;
        this.currentCell.removeTools();
        const html =
          this.currentCell.html instanceof Function
            ? this.currentCell.html()
            : this.currentCell.html;
        this.currentCell.setProp("html", () => {
          const childrenTag = html.children[1];
          if (childrenTag.nodeName !== "PRE") {
            const input = html.querySelector("textarea");
            html.querySelector("textarea").outerHTML = changeTag(
              "pre",
              input.value,
              this.currentCell
            ).outerHTML;
          }
          return html;
        });
      }

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
      document.getElementById('currentTextNode').focus()
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
  },
  components: {
    Toolbar,
  },
  methods: {
    exportData() {
      localStorage.setItem("cell", JSON.stringify(this.graph.toJSON()));
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
}
</style>
