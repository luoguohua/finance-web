<template>
  <div class="app-container">
    <el-row :gutter="30">
      <el-col v-for="(item,index) in data" :key="index" :span="6">
        <el-card :body-style="{ padding: '0px'}" style="margin-bottom: 30px;">
          <img src="@/assets/images/book.png" width="100%">
          <div style="padding: 14px;font-size: 16px;">
            <span>{{ item.name }}</span>
            <div class="bottom" style="margin: 5px 0px;">
              <time class="time">{{ item.createTime }}</time>
            </div>
            <el-button type="text" size="mini" style="font-size: 16px">操作按钮</el-button>
          </div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>
<script>

export default {
  name: 'MyLedger',
  data() {
    return {
      data: [
        {
          name: '默认账本1',
          createTime: '2022-07-05 14:06:23'
        },
        {
          name: '默认账本2',
          createTime: '2022-07-05 14:06:23'
        },
        {
          name: '默认账本3',
          createTime: '2022-07-05 14:06:23'
        },
        {
          name: '默认账本4',
          createTime: '2022-07-05 14:06:23'
        },
        {
          name: '默认账本4',
          createTime: '2022-07-05 14:06:23'
        }

      ]
    }
  },
  created() {
    console.log(this.title)
  },
  methods: {}
}

</script>
<style>
  .el-button{
    padding:0!important;
  }
</style>
