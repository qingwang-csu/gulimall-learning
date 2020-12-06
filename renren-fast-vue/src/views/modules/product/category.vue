<template>
  <div>
    <!--分类树形列表 开始-->
    <el-tree
      :data="data"
      :props="defaultProps"
      :expand-on-click-node="false"
      node-key="catId"
      :default-expanded-keys="array"
      :draggable="true"
      :allow-drop="allowDrop"
      @node-drop="handleDrop"
      :highlight-current="true"
    >
    <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            type="text"
            size="mini"
            @click="() => append(data)" v-if="node.level == 1 || node.level == 2">
              <span style="color: #00a0e9">
                新增
              </span>
          </el-button>

          <el-button
            type="text"
            size="mini"
            @click="() => edit(data)" >
              编辑
          </el-button>

          <el-button
            type="text"
            size="mini"
            @click="() => remove(node, data)"  v-if="node.childNodes.length < 1">
              <span style="color: #dd0000">
                删除
              </span>
          </el-button>


        </span>
    </span>
    </el-tree>
    <!--分类树形列表 结束-->

    <!--新增弹框开始-->
    <el-dialog title="分类信息" :visible.sync="dialogFormVisible">
      <el-form :model="category">
        <el-form-item label="上级分类名" :label-width="formLabelWidth" v-if="saveType == 'save'">
          <el-input v-model="category.parent_name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="分类名" :label-width="formLabelWidth">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="排序" :label-width="formLabelWidth">
          <el-input-number v-model="category.sort" :min="0" label="排序" autocomplete="off"></el-input-number>
        </el-form-item>
        <el-form-item label="icon" :label-width="formLabelWidth">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位" :label-width="formLabelWidth">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="closeDialog()">取 消</el-button>
        <el-button type="primary" @click="submitCategory()">确 定</el-button>
      </div>
    </el-dialog>
    <!--新增弹框结束-->
  </div>
</template>
<script>
  export default {
    data() {
      return {
        data: [],
        defaultProps: {
          label: 'name',
          children: 'subcategories'
        },
        array:[],
        //控制对话框
        dialogFormVisible: false,
        //对话框绑定
        category:{
          sort:0
        },
        formLabelWidth: '120px',
        saveType: '',
        maxLevel : 0,
        updateNodes: [

        ],
        // flag: -1

      };
    },
    methods: {
      //获取分类树形列表
      getCategoryTreeList(){
        // this.$http.adornUrl(`/product/category/list/tree`)
        this.$http(
          {
            url: this.$http.adornUrl('/product/category/list/tree'),
            method: "get"
          }
        ).then(result => {
          // console.log(result)
          this.data = result.data.data;
        });
      },
      //删除分类
      remove(node,data){
        this.$confirm('是否确定删除'+ data.name+ '?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http(
            {
              url: this.$http.adornUrl('/product/category/delete'),
              method: "delete",
              data: [data.catId]
            }
          ).then(result => {
            if (result.data.msg == 'success'){
              this.$message({
                message: '删除成功',
                type: 'success'
              });
            }
            //刷新
            this.getCategoryTreeList();
            this.array = [data.parentCid]
          });
        }).catch(() => {

        });
      },
      //添加分类 打开对话框
      append(data){
        console.log(data)
        this.category.parentCid = data.catId
        this.category.catLevel = data.catLevel - 0 + 1
        this.category.parent_name = data.name
        //显示对话框
        this.saveType = "save"
        this.dialogFormVisible = true
      },
      //修改分类
      edit(data){
        //查询当前节点信息
        this.$http(
          {
            url: this.$http.adornUrl('/product/category/info/'+data.catId),
            method: "get"
          }
        ).then(result => {
          this.category = result.data.category
        })
        //显示对话框
        this.saveType = "edit"
        this.dialogFormVisible = true
      },
      //提交更新
      update(){
        console.log(this.category)
        this.$http(
          {
            url: this.$http.adornUrl('/product/category/update'),
            method: "post",
            data: this.category
          }
        ).then(result => {
          if (result.data.msg == 'success'){
            this.$message({
              message: '更新成功',
              type: 'success'
            });
          }
          this.dialogFormVisible = false
          //刷新
          this.getCategoryTreeList();
          this.array = [this.category.parentCid]
          this.category = {
            sort:0
          }
        });
      },
      //关闭对话框
      closeDialog(){
        this.dialogFormVisible = false
        this.category = {
          sort:0
        }
      },
      //确认保存 category
      save(){
        this.$http(
          {
            url: this.$http.adornUrl('/product/category/save'),
            method: "post",
            data: this.category
          }
        ).then(result => {
          if (result.data.msg == 'success'){
            this.$message({
              message: '保存成功',
              type: 'success'
            });
          }
          this.dialogFormVisible = false
          //刷新
          this.getCategoryTreeList();
          this.array = [this.category.parentCid]
          this.category = {
            sort:0
          }
        });
      },
      //提交submitCategory
      submitCategory(){
        if (this.saveType == 'save'){
          this.save()
        }else {
          this.update()
        }
      },

      //结点是否可以拖拽到对应位置
      allowDrop(draggingNode, dropNode, type) {
        this.countNodeLevel(draggingNode);
        let deep = this.maxLevel
        if (type === "inner") {
          this.maxLevel = 0
          return deep + dropNode.level <= 3;
        } else {
          this.maxLevel = 0
          return deep + dropNode.parent.level <= 3;
        }

      },
      countNodeLevel(node) {
        if (node.level === 1){
          this.maxLevel = 1
          //找到所有子节点，求出最大深度
          if (node.childNodes != null && node.childNodes.length > 0) {
            this.maxLevel = 2
            for (let i = 0; i < node.childNodes.length; i++) {
              if (node.childNodes[i] != null && node.childNodes[i].childNodes.length > 0) {
                  this.maxLevel = 3
              }
            }
          }
        }
        if (node.level === 2){
          //找到所有子节点，求出最大深度
          if (node.childNodes != null && node.childNodes.length > 0) {
            this.maxLevel = 2
          }else {
            this.maxLevel = 1
          }
        }
        if (node.level === 3){
          this.maxLevel = 1
        }
      },

      //拖拽成功之后 获取修改的数据
      handleDrop(draggingNode, dropNode, dropType, ev) {
        //拖拽到之内
        if (dropType === "inner"){
          let data = draggingNode.data ;
          data.parentCid = dropNode.data.catId
          data.catLevel = dropNode.data.catLevel*1 + 1
          // console.log(data)

          this.updateNodes.push({
            catId: data.catId,
            catLevel: data.catLevel,
            parentCid: data.parentCid,
            sort: data.sort,
            subcategories:[]
          })

          if (draggingNode.data.subcategories!= null && draggingNode.data.subcategories.length>0){
            for (let i = 0; i < draggingNode.data.subcategories.length; i++) {
              draggingNode.data.subcategories[i].catLevel = data.catLevel*1+1;
              this.updateNodes[0].subcategories.push({
                catId: draggingNode.data.subcategories[i].catId,
                catLevel: draggingNode.data.subcategories[i].catLevel,
                parentCid: draggingNode.data.subcategories[i].parentCid,
                sort: draggingNode.data.subcategories[i].sort,
                subcategories:[]
              })
              if (draggingNode.data.subcategories[i].subcategories != null && draggingNode.data.subcategories[i].subcategories.length>0){
                for (let j = 0; j < draggingNode.data.subcategories[i].subcategories.length; j++) {
                  draggingNode.data.subcategories[i].subcategories[j].catLevel = data.catLevel*1+2;
                  this.updateNodes[0].subcategories[i].subcategories.push({
                    catId: draggingNode.data.subcategories[i].subcategories[j].catId,
                    catLevel: draggingNode.data.subcategories[i].subcategories[j].catLevel,
                    parentCid: draggingNode.data.subcategories[i].subcategories[j].parentCid,
                    sort: draggingNode.data.subcategories[i].subcategories[j].sort,
                    subcategories:[]
                  })
                }
              }

            }
          }

          // console.log(this.updateNodes)

        }else {//拖拽到之前和之后
          this.updateNodes = []
          //顶层结点
          if (dropNode.level === 1){
            //排序
            for (let i = 0; i < this.data.length; i++) {
              this.data[i].sort = i
              // this.data[i].parentCid = 0
              // this.data[i].catLevel = 1
              // console.log(this.data[i])
              this.updateNodes.push({
                catId: this.data[i].catId,
                catLevel: this.data[i].catLevel,
                parentCid: this.data[i].parentCid,
                sort: this.data[i].sort,
                subcategories:[]
              })
              // this.updateNodes.push({
              //   catId: data.catId,
              //   catLevel: data.catLevel,
              //   parentCid: data.parentCid,
              //   sort: data.sort,
              //   subcategories:[]
              // })
            }
            // this.push2(this.data)
            // console.log(this.updateNodes)
            // return;
          }
          let data = draggingNode.data ;
          // data.parentCid = dropNode.data.parentCid
          // draggingNode.data.catLevel = dropNode.level
          // console.log(data.sort)
          // console.log(draggingNode)
          // console.log(data)
          // console.log(draggingNode)
          // console.log(dropNode)
          // console.log()
          // console.log(draggingNode.data.catLevel)
          // console.log(draggingNode.level)
          if (draggingNode.level != dropNode.level){
            // console.log(this.updateNodes)
            // console.log(this.updateNodes[data.sort])
            // if (dropNode.parent != null){
            //
            // }
            // console.log(dropNode.parent)
            console.log(this.updateNodes[data.sort])
            this.updateNodes[data.sort].parentCid = dropNode.data.parentCid
            this.updateNodes[data.sort].catLevel = dropNode.data.catLevel
            if (draggingNode.data.subcategories!= null && draggingNode.data.subcategories.length>0){
              // console.log("===================")
              for (let i = 0; i < draggingNode.data.subcategories.length; i++) {
                // console.log(draggingNode.data.subcategories[i].sort)
                draggingNode.data.subcategories[i].catLevel = dropNode.level*1+1;
                // console.log(this.updateNodes)
                this.updateNodes[data.sort].subcategories.push({
                  catId: draggingNode.data.subcategories[i].catId,
                  catLevel: draggingNode.data.subcategories[i].catLevel,
                  parentCid: draggingNode.data.subcategories[i].parentCid,
                  sort: draggingNode.data.subcategories[i].sort,
                  subcategories:[]
                })
                if (draggingNode.data.subcategories[i].subcategories != null && draggingNode.data.subcategories[i].subcategories.length>0){
                  for (let j = 0; j < draggingNode.data.subcategories[i].subcategories.length; j++) {
                    draggingNode.data.subcategories[i].subcategories[j].catLevel = dropNode.level*1+2;
                    this.updateNodes[data.sort].subcategories[i].subcategories.push({
                      catId: draggingNode.data.subcategories[i].subcategories[j].catId,
                      catLevel: draggingNode.data.subcategories[i].subcategories[j].catLevel,
                      parentCid: draggingNode.data.subcategories[i].subcategories[j].parentCid,
                      sort: draggingNode.data.subcategories[i].subcategories[j].sort,
                      subcategories:[]
                    })
                  }
                }
              }
            }

            // if (dropNode.parent != null && dropNode.parent.data.subcategories.length>0){
            //   for (let i = 0; i < dropNode.parent.data.subcategories.length; i++) {
            //     dropNode.parent.data.subcategories[i].sort = i
            //   }
            // }
          }
        }
        console.log(this.updateNodes)
        this.$http(
          {
            url: this.$http.adornUrl('/product/category/update/batch'),
            method: "post",
            data: this.updateNodes
          }
        ).then(result => {
          if (result.data.msg == 'success'){
            this.$message({
              message: '拖拽成功',
              type: 'success'
            });
          }
          //刷新
          this.getCategoryTreeList();
          this.array = [this.category.parentCid]
          this.updateNodes = []
        });
      }
    },

    created () {
      this.getCategoryTreeList();
    }
  };
</script>
<style>
  .custom-tree-node {
    flex: 2;
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 16px;
    padding-right: 8px;
  }
</style>
