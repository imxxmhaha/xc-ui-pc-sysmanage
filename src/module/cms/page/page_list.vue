<template>
  <div>
    <!--查询表单-->
    <el-form :model="params">
      <el-select v-model="params.siteId" placeholder="请选择站点">
        <el-option
          v-for="item in siteList"
          :key="item.siteId"
          :label="item.siteName"
          :value="item.siteId">
        </el-option>
      </el-select>
      页面别名：
      <el-input v-model="params.pageAliase" style="width: 100px"></el-input>
      <el-button type="primary" v-on:click="query" size="small">查询</el-button>

      <router-link class="mui-tab-item" :to="{path:'/cms/page/add/',query:{
  page: this.params.page,
  siteId: this.params.siteId}}">
        <el-button type="primary" size="small">新增页面</el-button>
      </router-link>

    </el-form>
    <el-table
      :data="list"
      stripe
      style="width: 100%">
      <el-table-column type="index" width="60">
      </el-table-column>
      <el-table-column prop="pageName" label="页面名称" width="100">
      </el-table-column>
      <el-table-column prop="pageAliase" label="别名" width="120">
      </el-table-column>
      <el-table-column prop="pageType" label="页面类型" width="50">
      </el-table-column>
      <el-table-column prop="pageWebPath" label="访问路径" width="250">
      </el-table-column>
      <el-table-column prop="pagePhysicalPath" label="物理路径" width="250">
      </el-table-column>

      <el-table-column label="操作" width="180">
        <template slot-scope="page">
          <el-button
            size="small" type="text"
            @click="edit(page.row.pageId)">编辑
          </el-button>
          <el-button
            size="small" type="text"
            @click="del(page.row.pageId)">删除
          </el-button>
          <el-button @click="preview(page.row.pageId)" type="text" size="small">页面预览</el-button>
          <el-button plain @click="postPage(page.row.pageId)" type="primary" size="small">发布</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      layout="prev, pager, next"
      :total="total"
      :current-page="params.page"
      :page-size="params.size"
      @current-change="changePage"
      style="float:right">
    </el-pagination>
  </div>
</template>

<script>
  import * as cmsApi from "../api/cms"

  export default {
    data() {
      return {
        list: [],
        total: 0,
        params: {
          page: 1,
          size: 10,
          siteId: "",
          pageAliase: ""
        },
        siteList: []
      }
    },
    created() {
      //取出路由中的参数,赋值给数据对象
      this.params.page = Number.parseInt(this.$route.query.page || 1); // 双竖线 类似于三元表达式
      this.params.siteId = this.$route.query.siteId || null;
    },
    mounted() {
      //初始化站点列表
      this.params.siteId = this.params.siteId == null ? "5a751fab6abb5044e0d19ea1" : this.params.siteId;
      this.siteList = [
        {
          siteId: '5a751fab6abb5044e0d19ea1',
          siteName: '门户主站'
        },
        {
          siteId: '102',
          siteName: '测试站'
        }
      ]
      this.query();
    },
    methods: {
      //页面预览
      preview(pageId) {
        window.open("http://www.xuecheng.com/cms/preview/" + pageId)
      },
      postPage(pageId) {
        this.$confirm('确认发布该页面吗?', '提示', {}).then(() => {
          cmsApi.page_postPage(pageId).then((res) => {
            if (res.success) {
              console.log('发布页面pageId=' + pageId);
              this.$message.success('发布成功，请稍后查看结果');
            } else {
              this.$message.error('发布失败');
            }
          });
        }).catch(() => {

        });
      },
      query() {
        // alert("查询");
        //调用服务端接口
        cmsApi.page_list(this.params.page, this.params.size, this.params).then((res) => {
            // 将res结果数据赋值给数据模型对象
            this.list = res.queryResult.list;
            this.total = res.queryResult.total;
          }
        )
      },
      changePage(currentPage) { // 这个形参就是当前页
        // alert(currentPage);
        this.params.page = currentPage;
        this.query();
      },
      //修改
      edit: function (pageId) {
        this.$router.push({
          path: '/cms/page/edit/' + pageId, query: {
            page: this.params.page,
            siteId: this.params.siteId
          }
        })
      },
      //删除
      del: function (pageId) {
        this.$confirm('确认删除此页面吗?', '提示', {}).then(() => {
          cmsApi.page_del(pageId).then((res) => {
            if (res.success) {
              this.$message({
                type: 'success',
                message: '删除成功!'
              });
              //查询页面
              this.query()
            } else {
              this.$message({
                type: 'error',
                message: '删除失败!'
              });
            }
          })

        })
      }
    }
  }
</script>
<style>
  /*编写页面样式，不是必须*/
</style>
