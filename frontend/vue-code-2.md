## 表格table及操作示例
```js
<script setup>
import { ElButton, ElImage, ElLink, ElPagination, ElTable, ElTableColumn, ElTag, ElNotification } from 'element-plus'
import 'element-plus/dist/index.css'

import axios from 'axios'
import { onMounted, ref } from 'vue'

// ----------------
const currentPage = ref(1)
const pageSize = ref(20)


const __total = ref(0)
const __list = ref([])

const __$_request_data = async () => {
  //   let para = {
  //     pageNum: currentPage.value,
  //     pageSize: pageSize.value
  //   }

  // 使用let定义url在这里仅作为使用示范
  let request_url = 'http://localhost:9999/xxx/public/test/page?pageNum=' + currentPage.value + '&pageSize=' + pageSize.value

  const response = await axios.get(request_url)
  console.log(1, '---', response.data)
  // 仅需读取值的变量建议使用const定义
  const responseBody = response.data

  __list.value = responseBody.data.records
  __total.value = responseBody.data.total
}



const handleSizeChange = (value) => {
  console.log(`${value} items per page`)
  __$_request_data()
}
const handleCurrentChange = (value) => {
  console.log(`current page: ${value}`)
  __$_request_data()
}




const __handle_up_shelf = async (index, row) => {
  console.log(index, row)
  // ------------------
  const url = 'http://localhost:9999/xxx/public/test/t-map'
  // 准备要发送的 JSON 数据
  const data = {
    key1: 'value1',
    key2: 'value2',
    id: row.pid
  }

  // 发送 POST 请求
  const response = await axios.post(url, data)
  console.log(1, '---', response.data)
}


const __handle_post = async (index, row) => {
  console.log(2, index, row)
  // ------------------
  const url = 'http://localhost:9999/xxx/public/test/t-map'
  const data = {
    id: row.pid
  }

  const response = await axios.post(url, data)
  console.log(3, '---', response.data)
  const responseBody = response.data

  ElNotification({
    title: '结果',
    message: responseBody.msg,
    type: 'success',
  })
}



// 生命周期
onMounted(() => __$_request_data())

</script>

<template>
  <el-table :data="__list" stripe style="width: 100%">

    <!--====带链接列=====-->
    <el-table-column label="商品" width="900" align="left">
      <template #default="scope">
        <!--=================-->
        <router-link :to="'/xx/detail/' + scope.row.pid" class="link-type">
          <span>{{ scope.row.title }}</span>
        </router-link>
        <el-tag v-if="scope.row.xFlag" type="danger" size="small" style="margin-left: 8px;">新</el-tag>
        <!--=================-->
      </template>
    </el-table-column>
    <!--=================-->

    <!--====显示图片====-->
    <el-table-column label="封面图">
      <template width="90" #default="scope">
        <el-image style="width:80px;height:80px;border:none;"
          :src="scope.row.img + '?x-oss-process=image/resize,h_90,m_lfit'" fit="contain" />
      </template>
    </el-table-column>


    <!---other col--->
    <el-table-column prop="price" label="价格" width="200" />
    <el-table-column prop="pid" label="商品id" align="center" />


    <el-table-column label="操作" align="center">
      <template #default="scope">
        <el-link type="primary" @click="handleUpdate(scope.row)" style="margin-right: 8px;">修改 </el-link>
        <!---业务操作--->
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
    <!--=================-->

  </el-table>


  <!--page footer-->
  <el-pagination class="page" v-model:current-page="currentPage" v-model:page-size="pageSize"
    :page-sizes="[20, 50, 100]" background layout="total, sizes, prev, pager, next, jumper" :total="__total"
    @size-change="handleSizeChange" @current-change="handleCurrentChange" />
</template>

<style scoped>
.page {
  margin-top: 10px;
  margin-bottom: 10px;
  float: right;
}
</style>
```


## uni-app中grid显示图片
```js
<uni-section title="历史记录" type="line" padding>
    <uni-grid :column="2" :highlight="true">
        <uni-grid-item v-for="(item, index) in __myMsgList" :index="index" :key="index">
            <image style="width: 200px; height: 200px; background-color: #eeeeee;" mode="aspectFill"
                :src="item.file_url" @click="previewImg($event, item.file_url, index)"></image>
        </uni-grid-item>
    </uni-grid>
</uni-section>

// 图片预览
const previewImg = (__e, __url, __index) => {
    console.log('previewImg', __e, __url, __index)
    // const { img } = __e.currentTarget.dataset
    // console.log('img', img)
    // -----------------------
    const __tmpImgUrls = __myMsgList.value.map(item => item.file_url)
    // console.log('xx', __tmpImgUrls)
    uni.previewImage({
        current: __index,
        urls: __tmpImgUrls
    })
}
```

---------------------
- [Vue3中文站](https://cn.vuejs.org/)