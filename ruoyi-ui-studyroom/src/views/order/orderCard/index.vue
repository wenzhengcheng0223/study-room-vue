<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryForm" size="small" :inline="true" v-show="showSearch" label-width="68px">
                  <el-form-item label="用户ID" prop="userId">
                    <el-input
                        v-model="queryParams.userId"
                        placeholder="请输入用户ID"
                        clearable
                        @keyup.enter.native="handleQuery"
                    />
                  </el-form-item>
                  <el-form-item label="优惠卡ID" prop="cardId">
                    <el-input
                        v-model="queryParams.cardId"
                        placeholder="请输入优惠卡ID"
                        clearable
                        @keyup.enter.native="handleQuery"
                    />
                  </el-form-item>
                  <el-form-item label="订单编号" prop="orderId">
                    <el-input
                        v-model="queryParams.orderId"
                        placeholder="请输入订单编号"
                        clearable
                        @keyup.enter.native="handleQuery"
                    />
                  </el-form-item>
                  <el-form-item label="描述" prop="description">
                    <el-input
                        v-model="queryParams.description"
                        placeholder="请输入描述"
                        clearable
                        @keyup.enter.native="handleQuery"
                    />
                  </el-form-item>
                  <el-form-item label="账单金额" prop="amountTotal">
                    <el-input
                        v-model="queryParams.amountTotal"
                        placeholder="请输入账单金额"
                        clearable
                        @keyup.enter.native="handleQuery"
                    />
                  </el-form-item>
                  <el-form-item label="付款状态" prop="payStatus">
                    <el-select v-model="queryParams.payStatus" placeholder="请选择付款状态" clearable>
                      <el-option
                          v-for="dict in dict.type.pay_status"
                          :key="dict.value"
                          :label="dict.label"
                          :value="dict.value"
                      />
                    </el-select>
                  </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
        <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
            type="primary"
            plain
            icon="el-icon-plus"
            size="mini"
            @click="handleAdd"
            v-hasPermi="['order:orderCard:add']"
        >新增</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
            type="success"
            plain
            icon="el-icon-edit"
            size="mini"
            :disabled="single"
            @click="handleUpdate"
            v-hasPermi="['order:orderCard:edit']"
        >修改</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
            type="danger"
            plain
            icon="el-icon-delete"
            size="mini"
            :disabled="multiple"
            @click="handleDelete"
            v-hasPermi="['order:orderCard:remove']"
        >删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
            type="warning"
            plain
            icon="el-icon-download"
            size="mini"
            @click="handleExport"
            v-hasPermi="['order:orderCard:export']"
        >导出</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="orderCardList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
              <el-table-column label="主键ID" align="center" prop="id" :show-overflow-tooltip="true" v-if="true"/>
              <el-table-column label="用户ID" align="center" prop="userId" min-width="100" :show-overflow-tooltip="true" />
              <el-table-column label="优惠卡ID" align="center" prop="cardId" min-width="100" :show-overflow-tooltip="true" />
              <el-table-column label="订单编号" align="center" prop="orderId" min-width="100" :show-overflow-tooltip="true" />
              <el-table-column label="描述" align="center" prop="description" min-width="100" :show-overflow-tooltip="true" />
              <el-table-column label="账单金额" align="center" prop="amountTotal" min-width="100" :show-overflow-tooltip="true" />
              <el-table-column label="付款状态" align="center" prop="payStatus">
                <template slot-scope="scope">
                      <dict-tag :options="dict.type.pay_status" :value="scope.row.payStatus"/>
                </template>
              </el-table-column>
      <el-table-column label="操作" fixed="right" width="120" align="center" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button
              size="mini"
              type="text"
              icon="el-icon-edit"
              @click="handleUpdate(scope.row)"
              v-hasPermi="['order:orderCard:edit']"
          >修改</el-button>
          <el-button
              size="mini"
              type="text"
              icon="el-icon-delete"
              @click="handleDelete(scope.row)"
              v-hasPermi="['order:orderCard:remove']"
          >删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination
        v-show="total>0"
        :total="total"
        :page.sync="queryParams.pageNum"
        :limit.sync="queryParams.pageSize"
        @pagination="getList"
    />

    <!-- 添加或修改购卡管理对话框 -->
    <el-dialog :title="title" :visible.sync="open" width="500px" append-to-body>
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
                    <el-form-item label="用户ID" prop="userId">
                      <el-input v-model="form.userId" placeholder="请输入用户ID" />
                    </el-form-item>
                    <el-form-item label="优惠卡ID" prop="cardId">
                      <el-input v-model="form.cardId" placeholder="请输入优惠卡ID" />
                    </el-form-item>
                    <el-form-item label="订单编号" prop="orderId">
                      <el-input v-model="form.orderId" placeholder="请输入订单编号" />
                    </el-form-item>
                    <el-form-item label="描述" prop="description">
                      <el-input v-model="form.description" placeholder="请输入描述" />
                    </el-form-item>
                    <el-form-item label="账单金额" prop="amountTotal">
                      <el-input v-model="form.amountTotal" placeholder="请输入账单金额" />
                    </el-form-item>
                    <el-form-item label="付款状态">
                      <el-radio-group v-model="form.payStatus">
                        <el-radio
                            v-for="dict in dict.type.pay_status"
                            :key="dict.value"
:label="parseInt(dict.value)"
                        >{{dict.label}}</el-radio>
                      </el-radio-group>
                    </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button :loading="buttonLoading" type="primary" @click="submitForm">确 定</el-button>
        <el-button @click="cancel">取 消</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
  import { listOrderCard, getOrderCard, delOrderCard, addOrderCard, updateOrderCard } from "@/api/order/orderCard";

  export default {
    name: "OrderCard",
        dicts: ['pay_status'],
    data() {
      return {
        // 按钮loading
        buttonLoading: false,
        // 遮罩层
        loading: true,
        // 选中数组
        ids: [],
        // 非单个禁用
        single: true,
        // 非多个禁用
        multiple: true,
        // 显示搜索条件
        showSearch: true,
        // 总条数
        total: 0,
        // 购卡管理表格数据
              orderCardList: [],
        // 弹出层标题
        title: "",
        // 是否显示弹出层
        open: false,
        // 查询参数
        queryParams: {
          pageNum: 1,
          pageSize: 10,
                        userId: undefined,
                        cardId: undefined,
                        orderId: undefined,
                        description: undefined,
                        amountTotal: undefined,
                        payStatus: undefined,
        },
        // 表单参数
        form: {},
        // 表单校验
        rules: {
                        id: [
                    { required: true, message: "主键ID不能为空", trigger: "blur" }
                  ],
                        userId: [
                    { required: true, message: "用户ID不能为空", trigger: "blur" }
                  ],
                        cardId: [
                    { required: true, message: "优惠卡ID不能为空", trigger: "blur" }
                  ],
                        orderId: [
                    { required: true, message: "订单编号不能为空", trigger: "blur" }
                  ],
                        description: [
                    { required: true, message: "描述不能为空", trigger: "blur" }
                  ],
                        amountTotal: [
                    { required: true, message: "账单金额不能为空", trigger: "blur" }
                  ],
                        payStatus: [
                    { required: true, message: "付款状态不能为空", trigger: "blur" }
                  ],
        }
      };
    },
    created() {
      this.getList();
    },
    methods: {
      /** 查询购卡管理列表 */
      getList() {
        this.loading = true;
        listOrderCard(this.queryParams).then(response => {
          this.orderCardList = response.rows;
          this.total = response.total;
          this.loading = false;
        });
      },
      // 取消按钮
      cancel() {
        this.open = false;
        this.reset();
      },
      // 表单重置
      reset() {
        this.form = {
                        id: undefined,
                        userId: undefined,
                        cardId: undefined,
                        orderId: undefined,
                        description: undefined,
                        amountTotal: undefined,
                    payStatus: 0,
                        createBy: undefined,
                        createTime: undefined,
                        updateBy: undefined,
                        updateTime: undefined
        };
        this.resetForm("form");
      },
      /** 搜索按钮操作 */
      handleQuery() {
        this.queryParams.pageNum = 1;
        this.getList();
      },
      /** 重置按钮操作 */
      resetQuery() {
        this.resetForm("queryForm");
        this.handleQuery();
      },
      // 多选框选中数据
      handleSelectionChange(selection) {
        this.ids = selection.map(item => item.id)
        this.single = selection.length!==1
        this.multiple = !selection.length
      },
      /** 新增按钮操作 */
      handleAdd() {
        this.reset();
        this.open = true;
        this.title = "添加购卡管理";
      },
      /** 修改按钮操作 */
      handleUpdate(row) {
        this.loading = true;
        this.reset();
        const id = row.id || this.ids
        getOrderCard(id).then(response => {
          this.loading = false;
          this.form = response.data;
          this.open = true;
          this.title = "修改购卡管理";
        });
      },
      /** 提交按钮 */
      submitForm() {
        this.$refs["form"].validate(valid => {
          if (valid) {
            this.buttonLoading = true;
            if (this.form.id != null) {
              updateOrderCard(this.form).then(response => {
                this.$modal.msgSuccess("修改成功");
                this.open = false;
                this.getList();
              }).finally(() => {
                this.buttonLoading = false;
              });
            } else {
              addOrderCard(this.form).then(response => {
                this.$modal.msgSuccess("新增成功");
                this.open = false;
                this.getList();
              }).finally(() => {
                this.buttonLoading = false;
              });
            }
          }
        });
      },
      /** 删除按钮操作 */
      handleDelete(row) {
        const ids = row.id || this.ids;
        this.$modal.confirm('是否确认删除购卡管理编号为"' + ids + '"的数据项？').then(() => {
          this.loading = true;
          return delOrderCard(ids);
        }).then(() => {
          this.loading = false;
          this.getList();
          this.$modal.msgSuccess("删除成功");
        }).finally(() => {
          this.loading = false;
        });
      },
  /** 导出按钮操作 */
  handleExport() {
    this.download('order/orderCard/export', {
      ...this.queryParams
    }, `orderCard_${new Date().getTime()}.xlsx`)
  }
  }
  };
</script>
