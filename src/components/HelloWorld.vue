<template>
  <div class="hello">
    <div class="input-section">
      <input v-model="inputValue" type="text" placeholder="请输入内容..." class="input-box" />
      <button @click="fetchData" class="confirm-button">确认</button>
    </div>
    <pre class="result-box"><code ref="codeBlock" class="language-json"></code></pre>
    <div ref="flowchart" class="flowchart"></div>
  </div>
</template>

<script>
import axios from 'axios';
import Prism from 'prismjs';
import 'prismjs/themes/prism-okaidia.css';
import d3 from 'd3';
export default {
  name: 'HelloWorld',
  data() {
    return {
      inputValue: '',
      responseData: ''
    };
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
          path: '/api/parse'
        });
        this.responseData = JSON.stringify(response.data, null, 2);
        this.$nextTick(() => {
          this.highlightCode();
        });
      } catch (error) {
        this.responseData = '请求失败，请检查网络或接口！';
      }
    },
  },
  updated() {
    this.highlightCode();
  }
}
</script>

<style scoped>
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
  color: #f8f8f2;
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
