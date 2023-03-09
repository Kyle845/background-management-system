<template lang="">
    <div>
       <el-form ref="form" label-width="80px" :model="spu">
           <el-form-item label="SPU名称">
                <el-input placeholder="SPU名称" v-model="spu.spuName"></el-input>
           </el-form-item>
           <el-form-item label="品牌">
             <el-select placeholder="请选择品牌" v-model="spu.tmId">
              <el-option
                  :label="tm.tmName"
                  :value="tm.id"
                  v-for="(tm, index) in tradeMarkList"
                  :key="tm.id"
          ></el-option>
             </el-select>
            </el-form-item> 
           <el-form-item label="SPU描述">
             <el-input 
             type="textarea" 
             rows="4" 
             placeholder="描述" 
             v-model="spu.description"
             ></el-input>
           </el-form-item>
           <el-form-item label="SPU图片">
            <el-upload
                action="/dev-api/admin/product/fileUpload"
                list-type="picture-card"
                :on-preview="handlePictureCardPreview"
                :on-remove="handleRemove"
                :on-success="handleSuccess"
                :file-list="spuImageList">
                <i class="el-icon-plus"></i>
            </el-upload>
            <el-dialog :visible.sync="dialogVisible">
                <img width="100%" :src="dialogImageUrl" alt="">   
            </el-dialog>
           </el-form-item>
           <el-form-item label="销售属性">
            <el-select 
            :placeholder="`还有${unSelectSaleAttr.length}未选择`" 
            v-model="attrIdAndAttrName">
              <el-option 
                :label="unselect.name" 
                :value="`${unselect.id}:${unselect.name}`" 
                v-for="(unselect,index) in unSelectSaleAttr" 
                :key="unselect.id"></el-option>
            </el-select>
            <el-button 
              type="primary" 
              icon="el-icon-plus" 
              :disabled="!attrIdAndAttrName" 
              @click="addSaleAttr">
              添加销售属性</el-button>
            <el-table style="width: 100%" border :data="spu.spuSaleAttrList">
                <el-table-column 
                    type="index" 
                    label="序号" 
                    width="80px" 
                    align="center">
                </el-table-column>
                <el-table-column prop="saleAttrName" label="属性名" width="width">
                </el-table-column>
                <el-table-column prop="prop" label="属性名称列表" width="width">
                    <template slot-scope="{row,$index}">
                      <el-tag 
                        :key="tag.id" 
                        v-for="(tag,index) in row.spuSaleAttrValueList" 
                        closable
                        :disable-transitions="false" 
                        @close="row.spuSaleAttrValueList.splice(index, 1)">
                        {{tag.saleAttrValueName}}
                      </el-tag>
                      <el-input class="input-new-tag" 
                        v-if="row.inputVisible"
                        v-model="row.inputValue"
                        ref="saveTagInput"
                        size="small"
                        @blur="handleInputConfirm(row)"
                        >
                      </el-input>
                      <el-button 
                        v-else 
                        class="button-new-tag" 
                        size="small" 
                        @click="addSaleAttrValue(row)">添加</el-button>
                    </template>
                </el-table-column>
                <el-table-column prop="prop" label="操作" width="width">
                  <template slot-scope="{row,$index}">
                      <el-button 
                        type="danger" 
                        icon="el-icon-delete" 
                        size="mini" 
                        @click="spu.spuSaleAttrList.splice($index,1)"></el-button>
                  </template>
                </el-table-column>
            </el-table>
           </el-form-item>
           <el-form-item>
                <el-button type="primary" @click="addOrUpdateSpu">保存</el-button>
                <el-button @click="cancel">取消</el-button>
           </el-form-item>
           <el-form>
        </el-form>  
        </el-form>
    </div>
</template>
<script>
export default {
    name:"",
    data(){
        return {
            dialogImageUrl:'',
            dialogVisible:false,
            spu: {
        //三级分类的id
                category3Id: 0,
                //描述
                description: "",
                //spu名称
                spuName: "",
                //平台的id
                tmId: "",
                //收集SPU图片的信息
                spuImageList: [
                  // {
                  //   id: 0,
                  //   imgName: "string",
                  //   imgUrl: "string",
                  //   spuId: 0,
                  // },
                ],
                //平台属性与属性值收集
                spuSaleAttrList: [
                  // {
                  //   baseSaleAttrId: 0,
                  //   id: 0,
                  //   saleAttrName: "string",
                  //   spuId: 0,
                  //   spuSaleAttrValueList: [
                  //     {
                  //       baseSaleAttrId: 0,
                  //       id: 0,
                  //       isChecked: "string",
                  //       saleAttrName: "string",
                  //       saleAttrValueName: "string",
                  //       spuId: 0,
                  //     },
                  //   ],
                  // },
                ],
      },
            tradeMarkList:[],
            spuImageList:[],
            saleAttrList:[],
            attrIdAndAttrName:'',
        }
    },
    methods: {
      handleRemove(file, fileList) { 
        this.spuImageList = fileList;
      },
      handlePictureCardPreview(file) {
        this.dialogImageUrl = file.url;
        this.dialogVisible = true;   
      },
      async initSpuData(spu){
        let spuResult = await this.$API.spu.reqSpu(spu.id);
        if (spuResult.code == 200) {
            this.spu = spuResult.data;
        }
        let tradeMarkResult = await this.$API.spu.reqTradeMarkList();
        console.log(tradeMarkResult);
        if (tradeMarkResult.code == 200) {
            this.tradeMarkList = tradeMarkResult.data;
        }
        let spuImageResult = await this.$API.spu.reqSpuImageList(spu.id);
        if(spuImageResult.code == 200) {
                let listArr = spuImageResult.data;
                listArr.forEach(item => {
                    item.name = item.imgName;
                    item.url = item.imgUrl;
                });
                this.spuImageList = listArr;
        }
        let saleResult = await this.$API.spu.reqBaseSaleAttrList();
        if(saleResult.code == 200) {
                this.saleAttrList = saleResult.data;
        }
      },
      handleSuccess(response,file,fileList){
            this.spuImageList = fileList;
      },
      addSaleAttr(){
        const [baseSaleAttrId,saleAttrName]=this.attrIdAndAttrName.split(':');
        let newSaleAttr = {baseSaleAttrId,saleAttrName,spuSaleAttrValueList:[]};
        this.spu.spuSaleAttrValueList.push(newSaleAttr);
        this.attrIdAndAttrName = '';
      },
      addSaleAttrValue(row){
          this.$set(row,'inputVisible',true);
          this.set(row,'inputValue','');
      },
      handleInputConfirm(row){
        const {baseSaleAttrId,inputValue} = row;
        if(inputValue.trim()==''){
          this.$message('属性值不能为空');
          return;
        }
        let result = row.spuSaleAttrValueList.every(
          item=>item.saleAttrValueName != inputValue);
        if(!result) return;
        console.log(result);
        let newSaleAttrValue = {baseSaleAttrId,saleAttrValueName:inputValue};
        row.spuSaleAttrValueList.push(newSaleAttrValue);
        row.inputVisible = false;
      },
      async addOrUpdateSpu() {
      //整理参数：需要整理照片墙的数据
      //携带参数：对于图片，需要携带imageName与imageUrl字段
      this.spu.spuImageList = this.spuImageList.map((item) => {
        return {
          imageName: item.name,
          imageUrl: (item.response && item.response.data) || item.url
        };
      }); 
      //发请求
      let result = await this.$API.spu.reqAddOrUpdateSpu(this.spu);
      if (result.code == 200) {
        //提示
        this.$message({ type: "success", message: "保存成功" });
        //通知父组件回到场景0
        this.$emit("changeScene", {
          scene: 0,
          flag: this.spu.id ? "修改" : "添加",
        });
      }
      // //清除数据
      Object.assign(this._data, this.$options.data());
    },
    async addSpuData(category3Id) {
      //添加SPU的时候收集三级分类的id
      this.spu.category3Id = category3Id;
      //获取品牌的信息
      let tradeMarkResult = await this.$API.spu.reqTradeMarkList();
      if (tradeMarkResult.code == 200) {
        this.tradeMarkList = tradeMarkResult.data;
      }
      //获取平台全部的销售属性
      let saleResult = await this.$API.spu.reqBaseSaleAttrList();
      if (saleResult.code == 200) {
        this.saleAttrList = saleResult.data;
      }
    },
    cancel() {
      //取消按钮的回调，通知父亲切换场景为0
      this.$emit("changeScene", { scene: 0, flag: "" });
      //清理数据
      //Object.assign:es6中新增的方法可以合并对象
      //组件实例this._data,可以操作data当中响应式数据
      //this.$options可以获取配置对象，配置对象的data函数执行，返回的响应式数据为空的
      Object.assign(this._data, this.$options.data());
    },
    },
    computed:{
      unSelectSaleAttr(){
        let result = this.saleAttrList.filter(item =>{
              return this.spu.spuSaleAttrList.every((item1) => {
                  return item.name != item1.saleAttrName;
              })
        })
        return result;
      }
    }
}
</script>
<style>
  .el-tag + .el-tag {
    margin-left: 10px;
  }
  .button-new-tag {
    margin-left: 10px;
    height: 32px;
    line-height: 30px;
    padding-top: 0;
    padding-bottom: 0;
  }
  .input-new-tag {
    width: 90px;
    margin-left: 10px;
    vertical-align: bottom;
  }
</style>