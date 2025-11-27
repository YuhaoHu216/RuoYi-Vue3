<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryRef" :inline="true" v-show="showSearch" label-width="68px">
      <el-form-item label="商品名称" prop="goodsName">
        <el-input
          v-model="queryParams.goodsName"
          placeholder="请输入商品名称"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="商品类型" prop="goodsType">
        <el-select v-model="queryParams.goodsType" placeholder="请选择商品类型" clearable>
          <el-option
            v-for="dict in sys_goods_type"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="商品价格" prop="goodsPrice">
        <el-input
          v-model="queryParams.goodsPrice"
          placeholder="请输入商品价格"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="商品库存" prop="goodsStock">
        <el-input
          v-model="queryParams.goodsStock"
          placeholder="请输入商品库存"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="Search" @click="handleQuery">搜索</el-button>
        <el-button icon="Refresh" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
          type="primary"
          plain
          icon="Plus"
          @click="handleAdd"
          v-hasPermi="['goods:goods:add']"
        >新增</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="success"
          plain
          icon="Edit"
          :disabled="single"
          @click="handleUpdate"
          v-hasPermi="['goods:goods:edit']"
        >修改</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="Delete"
          :disabled="multiple"
          @click="handleDelete"
          v-hasPermi="['goods:goods:remove']"
        >删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="Download"
          @click="handleExport"
          v-hasPermi="['goods:goods:export']"
        >导出</el-button>
      </el-col>
      <right-toolbar v-model:showSearch="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="goodsList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="商品ID" align="center" prop="id" />
      <el-table-column label="商品名称" align="center" prop="goodsName" />
      <el-table-column label="商品类型" align="center" prop="goodsType">
        <template #default="scope">
          <dict-tag :options="sys_goods_type" :value="scope.row.goodsType"/>
        </template>
      </el-table-column>
      <el-table-column label="商品价格" align="center" prop="goodsPrice" />
      <el-table-column label="商品库存" align="center" prop="goodsStock" />
      <el-table-column label="商品主图" align="center" prop="goodsMainImg" width="100">
        <template #default="scope">
          <image-preview :src="scope.row.goodsMainImg" :width="50" :height="50"/>
        </template>
      </el-table-column>
      <el-table-column label="商品图片" align="center" prop="goodsImages" width="100">
        <template #default="scope">
          <image-preview :src="scope.row.goodsImages" :width="50" :height="50"/>
        </template>
      </el-table-column>
     <el-table-column label="商品详情" align="center" prop="goodsDetail" width="300">
  <template #default="scope">
    <div v-html="scope.row.goodsDetail" class="goods-detail-preview"></div>
  </template>
</el-table-column>



      <el-table-column label="状态" align="center" prop="status" />
      <el-table-column label="备注" align="center" prop="remark" />
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template #default="scope">
          <el-button link type="primary" icon="Edit" @click="handleUpdate(scope.row)" v-hasPermi="['goods:goods:edit']">修改</el-button>
          <el-button link type="primary" icon="Delete" @click="handleDelete(scope.row)" v-hasPermi="['goods:goods:remove']">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    
    <pagination
      v-show="total>0"
      :total="total"
      v-model:page="queryParams.pageNum"
      v-model:limit="queryParams.pageSize"
      @pagination="getList"
    />

    <!-- 添加或修改商品信息管理对话框 -->
    <el-dialog :title="title" v-model="open" width="500px" append-to-body>
      <el-form ref="goodsRef" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="商品名称" prop="goodsName">
          <el-input v-model="form.goodsName" placeholder="请输入商品名称" />
        </el-form-item>
        <el-form-item label="商品类型" prop="goodsType">
          <el-select v-model="form.goodsType" placeholder="请选择商品类型">
            <el-option
              v-for="dict in sys_goods_type"
              :key="dict.value"
              :label="dict.label"
              :value="dict.value"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="商品价格" prop="goodsPrice">
          <el-input v-model="form.goodsPrice" placeholder="请输入商品价格" />
        </el-form-item>
        <el-form-item label="商品库存" prop="goodsStock">
          <el-input v-model="form.goodsStock" placeholder="请输入商品库存" />
        </el-form-item>
        <el-form-item label="商品主图" prop="goodsMainImg">
          <image-upload v-model="form.goodsMainImg"/>
        </el-form-item>
        <el-form-item label="商品图片" prop="goodsImages">
          <image-upload v-model="form.goodsImages"/>
        </el-form-item>
        <el-form-item label="商品详情">
          <editor v-model="form.goodsDetail" :min-height="192"/>
        </el-form-item>
        <el-form-item label="备注" prop="remark">
          <el-input v-model="form.remark" type="textarea" placeholder="请输入内容" />
        </el-form-item>
        <el-form-item label="删除标志" prop="delFlag">
          <el-input v-model="form.delFlag" placeholder="请输入删除标志" />
        </el-form-item>
      </el-form>
      <template #footer>
        <div class="dialog-footer">
          <el-button type="primary" @click="submitForm">确 定</el-button>
          <el-button @click="cancel">取 消</el-button>
        </div>
      </template>
    </el-dialog>
  </div>
</template>

<script setup name="Goods">
import { listGoods, getGoods, delGoods, addGoods, updateGoods } from "@/api/goods/goods"

const { proxy } = getCurrentInstance()
const { sys_goods_type } = proxy.useDict('sys_goods_type')

const goodsList = ref([])
const open = ref(false)
const loading = ref(true)
const showSearch = ref(true)
const ids = ref([])
const single = ref(true)
const multiple = ref(true)
const total = ref(0)
const title = ref("")

function extractFirstImage(html) {
  if (!html) return '';
  const match = html.match(/<img[^>]+src="([^">]+)"/i);
  return match ? match[1] : '';
}



const data = reactive({
  form: {},
  queryParams: {
    pageNum: 1,
    pageSize: 10,
    goodsName: null,
    goodsType: null,
    goodsPrice: null,
    goodsStock: null,
    goodsMainImg: null,
    goodsImages: null,
    goodsDetail: null,
    status: null,
  },
  rules: {
    goodsName: [
      { required: true, message: "商品名称不能为空", trigger: "blur" }
    ],
  }
})

const { queryParams, form, rules } = toRefs(data)

/** 查询商品信息管理列表 */
function getList() {
  loading.value = true
  listGoods(queryParams.value).then(response => {
    goodsList.value = response.rows
    total.value = response.total
    loading.value = false
  })
}

// 取消按钮
function cancel() {
  open.value = false
  reset()
}

// 表单重置
function reset() {
  form.value = {
    id: null,
    goodsName: null,
    goodsType: null,
    goodsPrice: null,
    goodsStock: null,
    goodsMainImg: null,
    goodsImages: null,
    goodsDetail: null,
    status: null,
    createBy: null,
    createTime: null,
    updateBy: null,
    updateTime: null,
    remark: null,
    delFlag: null
  }
  proxy.resetForm("goodsRef")
}

/** 搜索按钮操作 */
function handleQuery() {
  queryParams.value.pageNum = 1
  getList()
}

/** 重置按钮操作 */
function resetQuery() {
  proxy.resetForm("queryRef")
  handleQuery()
}

// 多选框选中数据
function handleSelectionChange(selection) {
  ids.value = selection.map(item => item.id)
  single.value = selection.length != 1
  multiple.value = !selection.length
}

/** 新增按钮操作 */
function handleAdd() {
  reset()
  open.value = true
  title.value = "添加商品信息管理"
}

/** 修改按钮操作 */
function handleUpdate(row) {
  reset()
  const _id = row.id || ids.value
  getGoods(_id).then(response => {
    form.value = response.data
    open.value = true
    title.value = "修改商品信息管理"
  })
}

/** 提交按钮 */
function submitForm() {
  proxy.$refs["goodsRef"].validate(valid => {
    if (valid) {
      if (form.value.id != null) {
        updateGoods(form.value).then(response => {
          proxy.$modal.msgSuccess("修改成功")
          open.value = false
          getList()
        })
      } else {
        addGoods(form.value).then(response => {
          proxy.$modal.msgSuccess("新增成功")
          open.value = false
          getList()
        })
      }
    }
  })
}

/** 删除按钮操作 */
function handleDelete(row) {
  const _ids = row.id || ids.value
  proxy.$modal.confirm('是否确认删除商品信息管理编号为"' + _ids + '"的数据项？').then(function() {
    return delGoods(_ids)
  }).then(() => {
    getList()
    proxy.$modal.msgSuccess("删除成功")
  }).catch(() => {})
}

/** 导出按钮操作 */
function handleExport() {
  proxy.download('goods/goods/export', {
    ...queryParams.value
  }, `goods_${new Date().getTime()}.xlsx`)
}

getList()
</script>
