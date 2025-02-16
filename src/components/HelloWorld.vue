<template>
  <div class="hello">
    <div class="input-section">
      <input v-model="inputValue" type="text" placeholder="请输入内容..." class="input-box" />
      <button @click="fetchData" class="confirm-button">确认</button>
    </div>
    <pre class="result-box"><code ref="codeBlock" class="language-json"></code></pre>
    <div class="graph">
      <div id="container" ref="container" class="container"></div>
      <!-- 悬浮提示框 -->
      <div v-if="tooltip.visible" class="tooltip" :style="{ top: tooltip.y + 'px', left: tooltip.x + 'px' }">
        <strong>服务名:</strong> {{ tooltip.data.serviceName }} <br>
        <strong>方法名:</strong> {{ tooltip.data.methodName }} <br>
        <strong>描述:</strong> {{ tooltip.data.methodDesc }} <br>
        <strong>路径:</strong> {{ tooltip.data.methodFullPath }}<br>
        <strong>源代码:</strong>
        <div class="source-code">
          {{ tooltip.data.sourceCode }}
        </div>

      </div>

    </div>
  </div>
</template>

<script>
import axios from 'axios';
import Prism from 'prismjs';
import 'prismjs/themes/prism-okaidia.css';
import { Graph } from '@antv/x6';
export default {
  name: 'HelloWorld',
  data() {
    return {
      inputValue: '',
      responseData: '',
      graph: {},
      tooltip: {
        visible: false,
        x: 0,
        y: 0,
        data: {
          methodName: '',
          methodDesc: '',
          methodFullPath: '',
          sourceCode: '',
          serviceName:''
        }
      }
    };
  },
  mounted() {
  },
  methods: {
    highlightCode() {
      if (this.$refs.codeBlock) {
        this.$refs.codeBlock.textContent = this.responseData;
        Prism.highlightElement(this.$refs.codeBlock);
      }
    },
    async fetchData() {
      try {
        const response = await axios.post(`http://localhost:8081/api/getSourceCode`, {
          path: '/api/getSourceCode1'
        });
        this.responseData = JSON.stringify(response.data, null, 2);
        this.$nextTick(() => {
          this.highlightCode();
        });
        
        this.drawGraph();
      } catch (error) {
        this.responseData = '请求失败，请检查网络或接口！';
      }
    },
    drawGraph() {
      this.graph = new Graph({
        container: this.$refs.container,
        width: 1800,
        height: 1000,
        grid: true, // 显示网格
      });

      const nodes = {}; // 存储所有节点
      let posX = 100; // 节点起始 X 坐标
      let posY = 50; // 节点起始 Y 坐标

      // 递归解析 JSON 并添加节点
      const parseTree = (node, parentId = null, depth = 0) => {
        const id = node.methodName;
        const x = posX + depth * 200; // X 轴位置
        const y = posY + Object.keys(nodes).length * 80; // Y 轴位置（动态调整）

        // 添加节点
        nodes[id] = this.graph.addNode({
          id,
          x,
          y,
          width: 140,
          height: 50,
          label: node.methodName,
          shape: "rect",
          data: node,
          attrs: {
            body: { stroke: "#5B8FF9", fill: "#C6E5FF" },
            label: { fill: "#000" },
          },
        });

        // 添加连线（如果有父节点）
        if (parentId) {
          this.graph.addEdge({
            source: nodes[parentId],
            target: nodes[id],
            label: "调用",
            attrs: { line: { stroke: "#5B8FF9", strokeWidth: 2 } },
          });
        }

        // 递归解析子节点
        node.calls.forEach((child) => {
          parseTree(child, id, depth + 1);
        });
      };

      // 开始解析 JSON
      parseTree(JSON.parse(this.responseData));
      // 监听鼠标悬浮事件，显示 tooltip
      this.graph.on('node:mouseenter', ({ node, e }) => {
        const nodeData = node.getData() || {}
        this.tooltip = {
          visible: true,
          x: e.clientX + 10,
          y: e.clientY + 10,
          data: {
            methodName: nodeData.methodName || '未知',
            methodDesc: nodeData.methodDesc || '暂无描述',
            methodFullPath: nodeData.methodFullPath || '未知路径',
            sourceCode: nodeData.sourceCode || '未知路径',
            serviceName: nodeData.serviceName || '未知路径',
          }
        }
      })

      // 鼠标移动时更新 tooltip 位置
      this.graph.on('node:mousemove', ({ e }) => {
        if (this.tooltip.visible) {
          this.tooltip.x = e.clientX
          this.tooltip.y = e.clientY
        }
      })

      // 鼠标移开时隐藏 tooltip
      this.graph.on('node:mouseleave', () => {
        //this.tooltip.visible = false
      })
      this.graph.on('blank:click', ({ e }) => {
        this.tooltip.visible = false
      });
      // this.graph.enablePanning(); // 允许平移
      // this.graph.enableSelection(); // 允许选中
      // this.graph.enableKeyboard(); // 允许键盘操作

    },
  },
  updated() {
    this.highlightCode();
  }
}
</script>

<style scoped>
.source-code {
  word-wrap: break-word;
  /* 让长单词也能换行 */
  word-break: break-word;
  /* 兼容更多浏览器 */
  white-space: pre-wrap;
  /* 允许正常换行 */
  /* overflow: scroll; */
  width: 100%;
  height: auto;
  background-color: rgb(24, 79, 79);
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);

}

.graph {
  position: relative;
  height: auto;
  /* overflow: auto; */
}

.hello {
  position: relative;
  overflow: visible;
}

.tooltip {
  position: absolute;
  background: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 8px 12px;
  border-radius: 6px;
  font-size: 20px;
  text-align: left;
  pointer-events: none;
  white-space: nowrap;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
  /* transition: opacity 0.2s; */
  opacity: 1;
  width: 800px;
  height: auto;
}

.container {
  width: 800px;
  height: 600px;
  border: 1px solid #ddd;
}

.input-section {
  display: flex;
  gap: 10px;
  justify-content: center;
  margin-bottom: 20px;
}

.input-box {
  flex: 1;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.confirm-button {
  padding: 8px 15px;
  border: none;
  background-color: #42b983;
  color: white;
  cursor: pointer;
  border-radius: 5px;
}

.result-box {
  width: 100%;
  height: 200px;
  border: 1px solid #ccc;
  padding: 10px;
  border-radius: 5px;
  resize: none;
  font-family: monospace;
}

.result-box {
  width: 100%;
  height: 600px;
  border: 1px solid #ccc;
  padding: 10px;
  border-radius: 5px;
  font-family: monospace;
  background-color: #1e1e1e;
  color: white;
  white-space: pre-wrap;
  overflow: auto;
  text-align: left;
}

.flowchart {
  margin-top: 20px;
  width: 100%;
  height: 400px;
  border: 1px solid #ccc;
}
</style>
