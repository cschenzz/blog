## 表格table及操作示例
```js
<script setup>
import { ElButton, ElTable, ElTableColumn, ElImage, ElTag } from 'element-plus';
import 'element-plus/dist/index.css';

import axios from 'axios';
import { onMounted, ref } from 'vue';

// ----------------
const __list = ref([])

const __$_request_data = async () => {
  const response = await axios.get('http://localhost:9900/cc/public/test/t-00?t=2&p=2')
  console.log(1, '---', response.data)
  __list.value = response.data.data
}

const __handle_up_shelf = async (index, row) => {
  console.log(index, row)
  // ------------------
  const url = 'http://localhost:9900/cc/public/test/t-01'
  // 准备要发送的 JSON 数据
  const data = {
    key1: 'value1',
    key2: 'value2',
    id: row.num_iid
  };

  // 发送 POST 请求
  const response = await axios.post(url, data)
  console.log(1, '---', response.data)
}


const __handle_post = async (index, row) => {
  console.log(2, index, row)
  // ------------------
  const url = 'http://localhost:9900/cc/public/test/t-01'
  const data = {
    id: row.num_iid
  };

  const response = await axios.post(url, data)
  console.log(3, '---', response.data)
}

onMounted(__$_request_data)
</script>

<template>
  <el-table :data="__list" stripe style="width: 100%">
    <el-table-column prop="title" label="名称" width="500" />
    <el-table-column prop="price" label="价格" width="200" />
    <el-table-column prop="pid" label="商品id" />
    <!--====显示图片====-->
    <el-table-column label="商品图">
      <template width="90" #default="scope">
        <el-image style="width:80px;height:80px;border:none;"
          :src="scope.row.img + '?x-oss-process=image/resize,h_90,m_lfit'" fit="contain" />
      </template>
    </el-table-column>

    <!--=================-->
    <el-table-column label="商品" width="180" align="left">
        <template #default="scope">
            <!--=================-->
            <router-link :to="'/xx/detail/' + scope.row.pid" class="link-type">
                <span>{{ scope.row.title }}</span>
            </router-link>
            <el-tag v-if="scope.row.xFlag" type="danger" size="small" style="margin-left: 8px;">促</el-tag>
            <!--=================-->
        </template>
    </el-table-column>
    <!--=================-->



    <el-table-column label="操作">
      <template #default="scope">
        <el-button size="small" type="primary" @click="__handle_up_shelf(scope.$index, scope.row)">
          上架
        </el-button>
        <el-button size="small" type="info" @click="__handle_post(scope.$index, scope.row)">
          下架
        </el-button>
        <el-button size="small" type="danger" @click="__handle_del(scope.$index, scope.row)">
          删除
        </el-button>
      </template>
    </el-table-column>
  </el-table>
</template>
```

---------------------
- [Vue3中文站](https://cn.vuejs.org/)