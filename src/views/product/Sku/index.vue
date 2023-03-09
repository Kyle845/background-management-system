<template lang="">
    <div>
        <el-table style="width: 100%" border :data="records">
            <el-table-column type="index" label="序号" width="80" align="center"></el-table-column>
            <el-table-column prop="skuName" label="名称" width="width"></el-table-column>
            <el-table-column prop="skuDesc" label="描述" width="width"></el-table-column>
            <el-table-column label="默认图片" width="110">
                <template slot-scope="{row,$index}">
                      <img :src="row.skuDefaultImg" alt="" style="width:80px;height:80px">
                </template>
            </el-table-column>
            <el-table-column prop="weight" label="重量" width="80"></el-table-column>
            <el-table-column prop="price" label="价格" width="80"></el-table-column>
            <el-table-column prop="prop" label="操作" width="width">
              <template slot-scope="{row,$index}">
                    <el-button 
                      type="success" 
                      icon="el-icon-sort-down" 
                      size="mini" 
                      v-if="row.isSale == 0"
                      @click="sale(row)"></el-button>
                      <el-button 
                      type="success" 
                      icon="el-icon-sort-up" 
                      size="mini" 
                      v-else
                      @click="cancel(row)"></el-button>
                      <el-button 
                      type="primary" 
                      icon="el-icon-edit" 
                      size="mini" 
                      @click="edit"></el-button>
                      <el-button 
                      type="info" 
                      icon="el-icon-info" 
                      size="mini" 
                      @click="getSkuInfo(row)"></el-button>
                      <el-button 
                      type="danger" 
                      icon="el-icon-delete" 
                      size="mini">
                    </el-button>
              </template>
            </el-table-column>
        </el-table>
        <el-pagination
              @current-change="getSkuList"
              @size-change="handleSizeChange"
              style="text-align: center"
              :current-page="page"
              :page-sizes="[3, 5, 10]"
              :page-size="limit"
              layout="prev, pager, next, jumper,->, sizes,total"
              :total="total">
        </el-pagination>
        <el-drawer :visible.sync="show" :show-close="true" size="50%">
              <el-row>
                    <el-col :span="5">名称</el-col>
                    <el-col :span="10">sku</el-col>
              </el-row>
        </el-drawer>
    </div>
</template>
<script>
export default {
  name: 'Sku',
  data(){
    return {
      page:1,
      limit:10,
      records:[],
      total:0,
      skuInfo:{},
      show:false,
    };
  },
  mounted() {
    this.getSkuList();
  },
  methods: {
    async getSkuList(pages = 1){
        this.page 
        const {page,limit} = this;
        let result = await this.$API.sku.reqSkuList(page,limit);
        console.log(result);
        if (result.code == 200) {
          this.total = result.data.total;
          this.records = result.data.records;
        }
    },
    handleSizeChange(limit){
        this.limit = limit;
        this.getSkuList();
    },
    async sale(row) {
      let result = await this.$API.sku.reqSale(row.id);
      if (result.code == 200) {
        row.isSale = 1;
        this.$message({type:"success",message:"上架成功"});
      }
    },
    async cancel(row) {
      let result = await this.$API.sku.reqCancel(row.id);
      if (result.code == 200) {
        row.isSale = 0;
        this.$message({type:"success",message:"下架成功"});
      }
    },
    edit(){

    },
    getSkuInfo(){

    }
  },
}
</script>
<style lang="">
</style>
