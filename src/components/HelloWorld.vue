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
import Toolbar from "./NodesBar.vue";
import { Graph } from "@antv/x6";
import spanCss from "../common/style";

// 删除元素
import removeBtn from "../common/removeBtn";

import { ToolTypeEnum } from "../common/enums";

function changeTag(tagType, value = "") {
  const node = document.createElement(tagType);
  if (tagType == "span") {
    node.style.cssText = spanCss;
    node.textContent = value;
  } else {
    // node.type = "text";
    // node.setAttribute("value", value);
    node.innerText = value;
    node.style.cssText = spanCss;
    node.focus();
  }
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
      currentCell: null
    };
  },
  watch: {},
  mounted() {
    // this.graph = graph()
    this.graph = new Graph({
      container: document.getElementById("container"),
      keyboard: {
        enabled: true,
        global: true,
      },
      autoResize: true,
      resizing: {
        preserveAspectRatio: true,
        enabled: true,
      },
      // 点选/框选，默认禁用。
      // selecting: {
      //   enabled: true,
      //   showNodeSelectionBox: true,
      //   // https://x6.antv.vision/zh/docs/tutorial/basic/selection
      //   rubberband: true, // 启用框选
      //   modifiers: "ctrl", // 组合键
      // },
      // 对齐线
      snapline: true,
      // 画布平移
      // https://x6.antv.vision/zh/docs/tutorial/basic/graph/#%E7%94%BB%E5%B8%83%E5%B9%B3%E7%A7%BB
      panning: {
        enabled: true,
        eventTypes: ["leftMouseDown", "rightMouseDown", "mouseWheel"],
      },
      // 网格
      grid: {
        size: 10, // 网格大小 10px
        visible: true, // 渲染网格背景
      },
      // 剪切板 Clipboard
      // https://x6.antv.vision/zh/docs/tutorial/basic/clipboard
      clipboard: true,
      // 撤销/重做 Redo/Undo
      // https://x6.antv.vision/zh/docs/tutorial/basic/history/#gatsby-focus-wrapper
      history: {
        enabled: true,
        beforeAddCommand(event, args) {
          if (args.key) {
            // 禁止删除按钮添加到 Undo 队列中
            return args.key !== "tools";
          }
        },
      },
      // 定制节点和边的交互行为
      // https://x6.antv.vision/zh/docs/tutorial/basic/interacting/#%E5%AE%9A%E5%88%B6%E4%BA%A4%E4%BA%92%E8%A1%8C%E4%B8%BA
      interacting: function (cellView) {
        if (cellView.cell.getData()?.disableMove) {
          return { nodeMovable: false };
        }
        return true;
      },
      // 配置全局的连线规则
      // https://x6.antv.vision/zh/docs/api/graph/interaction
      connecting: {
        // 自动吸附
        snap: true,
        // 不允许连接到画布空白位置的点
        allowBlank: false,
        // 不允许创建循环连线
        allowLoop: false,
        // 不允许在相同的起始节点和终止之间创建多条边
        allowMulti: false,
        // 高亮显示所有可用的连接桩或节点
        // https://x6.antv.vision/zh/docs/tutorial/basic/interacting/#highlight
        highlight: true,
        // 当连接到节点时，通过 sourceAnchor 来指定源节点的锚点。
        sourceAnchor: {
          name: "center",
        },
        // 当连接到节点时，通过 targetAnchor 来指定目标节点的锚点。
        targetAnchor: "center",
        // 指定连接点，默认值为 boundary。
        connectionPoint: "anchor",
        // 连接器将起点、路由返回的点、终点加工为 元素的 d 属性，决定了边渲染到画布后的样式，默认值为 normal。
        connector: {
          name: "rounded",
          args: {
            radius: 20,
          },
        },
        // 路由将边的路径点 vertices 做进一步转换处理，并在必要时添加额外的点，然后返回处理后的点，默认值为 normal。
        router: "metro",
        // https://x6.antv.vision/zh/docs/tutorial/basic/interacting/#validatemagnet
        // 判断是否新增边
        validateMagnet({ magnet }) {
          const portGroup = magnet.getAttribute("port-group");
          return portGroup !== "in";
        },
        // 连接的过程中创建新的边
        createEdge() {
          return this.createEdge({
            zIndex: -1,
            attrs: {
              line: {
                // strokeDasharray: "5 5",
                stroke: "#7c68fc",
                strokeWidth: 2,
                targetMarker: {
                  name: "block",
                  args: {
                    size: "6",
                  },
                },
              },
            },
          });
        },
        getHTMLComponent(node) {
          debugger
          const data = node.getData();
          if (data.flag) {
            return document.createElement("div");
          }
          return document.createElement("p");
        },
        // https://x6.antv.vision/zh/docs/tutorial/basic/interacting/#validateconnection
        // 在移动边的时候判断连接是否有效，如果返回 false，当鼠标放开的时候，不会连接到当前元素，否则会连接到当前元素。
        validateConnection({
          targetView,
          sourceMagnet,
          targetMagnet,
          sourceCell,
        }) {
          if (!sourceMagnet || !targetMagnet) {
            return false;
          }
          if (sourceCell.getData()?.disableMove) return false;
          // 判断目标链接桩是否可连接
          const portId = targetMagnet.getAttribute("port");
          const node = targetView.cell;
          const port = node.getPort(portId);
          if (!port) {
            return false;
          }
          return true;
        },
      },
      // 滚轮缩放
      mousewheel: {
        enabled: true,
        modifiers: ["ctrl", "meta"],
      },
    });

    Graph.registerNodeTool(ToolTypeEnum.REMOVE_BUTTON, removeBtn, true);
    Graph.registerEdgeTool(ToolTypeEnum.REMOVE_BUTTON, removeBtn, true);
    this.graph.on("cell:selected", ({ cell, e }) => {
    });

    if (localStorage.getItem("cell")) {
      console.log(JSON.parse(localStorage.getItem("cell")));
      this.graph.fromJSON(JSON.parse(localStorage.getItem("cell")));
    }


    this.graph.on("cell:unselected", ({ cell, node, e }) => {
      // Event.stopPropagation
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
        if (childrenTag.nodeName !== "SPAN") {
          const input = html.querySelector("textarea");
          html.querySelector("textarea").outerHTML = changeTag(
            "span",
            input.value
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
      if(!this.currentCell) return;
      const cell = this.currentCell
      cell.removeTools();
      changePortsVisible(cell, false);
      const html = cell.html instanceof Function ? cell.html() : cell.html;
      cell.setProp("html", () => {
        const childrenTag = html.children[1];
        if (childrenTag.nodeName !== "SPAN") {
          const input = html.querySelector("textarea");
          html.querySelector("textarea").outerHTML = changeTag(
            "span",
            input.value
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
            name: "ellipse",
            rx: 1, // 椭圆箭头的 x 半径
            ry: 1, // 椭圆箭头的 y 半径
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
      this.currentCell = cell;
      Event.stopPropagation;
      const html = cell.html instanceof Function ? cell.html() : cell.html;
      cell.setProp("html", (cells) => {
        const childrenTag = html.children[1];
        if (childrenTag.nodeName !== "TEXTAREA") {
          const span = html.querySelector("span");
          html.querySelector("span").outerHTML = changeTag(
            "textarea",
            span.innerText
          ).outerHTML;
        }
        return html;
      });

      let removeBtnCfg;
      if (cell.isEdge()) {
        cell.attr("line", { stroke: "skyblue", strokeWidth: 3 });
        removeBtnCfg = { distance: "30%" };
      }

      if (cell.isNode()) {
        const cellView = this.graph.findView(cell);
        cellView.addClass(`${cell.shape}-selected`);
        // 删除图标偏移量 { x: 0, y: 0, offset: { x: -5, y: -5 } }
        // https://x6.antv.vision/zh/docs/api/registry/node-tool#button-remove
        removeBtnCfg = { x: -28, y: -5 };
      }
      // Channel.dispatchEvent(CustomEventTypeEnum.CELL_CLICK, SelectStateEnum.SELECTED)

      // 多单选选中时，移除删除
      const cells = this.graph.getSelectedCells();
      if (cells.length > 1) {
        cells.forEach((currentCell) => {
          currentCell.removeTools();
        });
      } else {
        // x6默认提供 button-remove, icon比较小, 交互体验不友好
        // cell.addTools({
        //   name: 'button-remove', // 工具名称
        //   args: removeBtnCfg // 工具对应的参数
        // });
        cell.addTools({
          name: "custom-remove-button", // 工具名称
          args: removeBtnCfg, // 工具对应的参数
        });
      }
    });

    this.graph.on("node:mouseenter", ({ node, e }) => {
      Event.stopPropagation;
      const data = node.getData();
      if (data && data.disableMove) {
        changePortsVisible(node, false);
      } else changePortsVisible(node, true);
    });

    this.graph.on("node:mouseleave", ({ cell, node, e }) => {
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
      console.log(this.graph, this.graph.getNodes())
      localStorage.setItem("cell", JSON.stringify(this.graph.getNodes()));
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
