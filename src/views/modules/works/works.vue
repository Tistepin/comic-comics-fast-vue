<template>
  <div class="mod-config">
    <el-form
      :inline="true"
      :model="dataForm"
      @keyup.enter.native="getDataList(pageIndex)"
    >
      <el-form-item>
        <el-input
          v-model="dataForm.key"
          placeholder="参数名"
          clearable
        ></el-input>
      </el-form-item>
      <el-form-item>
        <el-select v-model="reviewStatus" placeholder="请选择" clearable>
          <el-option
            v-for="item in options1"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
        <el-select v-model="deleteStatus" placeholder="请选择" clearable>
          <el-option
            v-for="item in options2"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList(1)">查询</el-button>
        <el-button
          v-if="isAuth('works:works:save')"
          type="primary"
          @click="addOrUpdateHandle()"
          >新增</el-button
        >
        <el-button
          v-if="isAuth('works:works:delete')"
          type="danger"
          @click="deleteHandle()"
          :disabled="dataListSelections.length <= 0"
          >批量删除</el-button
        >
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%"
    >
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50"
      >
      </el-table-column>
      <el-table-column
        prop="worksId"
        header-align="center"
        align="center"
        label="id"
      >
      </el-table-column>
      <el-table-column
        prop="worksName"
        header-align="center"
        align="center"
        label="作品名"
      >
      </el-table-column>
      <el-table-column
        prop="defaultImage"
        header-align="center"
        align="center"
        label="作品默认展示图片"
      >
        <template slot-scope="scope">
          <img
            style="width: 100px; height: 100px"
            :src="scope.row.defaultImage"
          />
        </template>
      </el-table-column>
      <el-table-column
        prop="creator"
        header-align="center"
        align="center"
        label="创建人"
      >
      </el-table-column>
      <el-table-column
        prop="worksCreator"
        header-align="center"
        align="center"
        label="作品创作者"
      >
      </el-table-column>
      <el-table-column
        prop="worksCreateTime"
        header-align="center"
        align="center"
        label="作品创作时间"
      >
      </el-table-column>
      <el-table-column
        prop="worksArea"
        header-align="center"
        align="center"
        label="作品创作地区"
      >
      </el-table-column>
      <el-table-column
        prop="worksType"
        header-align="center"
        align="center"
        label="作品类型"
      >
        <template slot-scope="scope">
          {{ scope.row.worksType == 1 ? "漫画" : "小说" }}
        </template>
      </el-table-column>
      <el-table-column
        prop="worksScore"
        header-align="center"
        align="center"
        label="作品评分 10分满分"
      >
      </el-table-column>
      <el-table-column
        prop="worksRenew"
        header-align="center"
        align="center"
        label="作品更新至多少"
      >
      </el-table-column>
      <el-table-column
        prop="worksPopularity"
        header-align="center"
        align="center"
        label="作品人气 用户阅读10章以上为1"
      >
      </el-table-column>
      <el-table-column
        prop="worksDescribe"
        header-align="center"
        align="center"
        label="作品描述"
      >
      </el-table-column>
      <el-table-column
        prop="worksCategory"
        header-align="center"
        align="center"
        label="作品分类"
      >
      </el-table-column>
      <el-table-column
        prop="status"
        header-align="center"
        align="center"
        label="作品状态"
      >
        <template slot-scope="scope">
          {{ scope.row.status == 1 ? "可以看" : "不可以看" }}
        </template>
      </el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        label="注册时间"
      >
      </el-table-column>
      <el-table-column
        prop="editTime"
        header-align="center"
        align="center"
        label="修改时间"
      >
      </el-table-column>
      <el-table-column
        prop="deleteStatus"
        header-align="center"
        align="center"
        label="逻辑删除状态"
      >
        <template slot-scope="scope">
          {{ scope.row.deleteStatus == 0 ? "已删除" : "未删除" }}
        </template>
      </el-table-column>
      <el-table-column
        prop="worksStatus"
        header-align="center"
        align="center"
        label="作品状态"
      >
        <template slot-scope="scope">
          {{ scope.row.worksStatus == 1 ? "更新中" : "完结" }}
        </template>
      </el-table-column>
      <el-table-column
        prop="reviewStatus"
        header-align="center"
        align="center"
        label="review_status审核状态"
      >
        <template slot-scope="scope">
          {{
            scope.row.reviewStatus == 0
              ? "审核中"
              : scope.row.reviewStatus == 1
              ? "审核成功"
              : "审核失败"
          }}
        </template>
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作"
      >
        <template slot-scope="scope">
          <el-button
            type="text"
            size="small"
            @click="addOrUpdateHandle(scope.row.worksId)"
            >修改</el-button
          >
          <el-button
            type="text"
            size="small"
            @click="deleteHandle(scope.row.worksId)"
            >删除</el-button
          >
          <el-button
            type="text"
            size="small"
            @click="shHandle(scope.row)"
            v-if="scope.row.reviewStatus == 1 ? false : true"
            >审核</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper"
    >
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update
      v-if="addOrUpdateVisible"
      ref="addOrUpdate"
      @refreshDataList="getDataList(1)"
    ></add-or-update>
    <el-dialog
      title="审核"
      :close-on-click-modal="false"
      :visible.sync="shclick"
    >
      <el-form :model="shdialog" ref="shdialog" label-width="80px">
        <el-form-item label="作品ID" prop="worksID">
          <el-input
            v-model="shdialog.worksId"
            placeholder="作品ID"
            :disabled="true"
          ></el-input>
        </el-form-item>
        <el-form-item label="作品名称" prop="worksName">
          <el-input
            v-model="shdialog.worksName"
            placeholder="作品名称"
            :disabled="true"
          ></el-input>
        </el-form-item>

        <el-form-item label="审核结果" prop="worksName">
          <el-radio-group v-model="worksEntity.reviewStatus">
            <el-radio :label="1">审核成功</el-radio>
            <el-radio :label="2">审核失败</el-radio>
          </el-radio-group>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="shclick = false">取消</el-button>
        <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import AddOrUpdate from "./works-add-or-update";
export default {
  data() {
    return {
      dataForm: {
        key: "",
      },
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      shdialog: {}, // 审核框显示数据
      shclick: false, // 审核框的显示和隐藏
      worksEntity: {
        worksId: "",
        reviewStatus: 1,
      },
      options1: [
        {
          value: "0",
          label: "未审核",
        },
        {
          value: "1",
          label: "已审核",
        },
        {
          value: "2",
          label: "审核失败",
        },
      ],
      options2: [
        {
          value: "1",
          label: "未删除",
        },
        {
          value: "0",
          label: "已删除",
        },
      ],
      deleteStatus: null,
      reviewStatus: null,
    };
  },
  components: {
    AddOrUpdate,
  },
  activated() {
    this.getDataList(1);
  },
  methods: {
    // 发出请求审核
    dataFormSubmit() {
      this.worksEntity.worksId = this.shdialog.worksId;
      this.$http({
        url: this.$http.adornUrl("/works/works/review"),
        method: "post",
        data: JSON.stringify(this.worksEntity),
      }).then(({ data }) => {
        if (data && data.code === 20000) {
          this.$message({
            showClose: true,
            message: "审核成功",
            type: "success",
          });
          this.getDataList(1);
        } else {
          this.$message({
            showClose: true,
            message: "审核失败",
            type: "error",
          });
        }
        this.shclick = false;
      });
    },
    // 开启审核框
    shHandle(row) {
      this.shdialog = row;
      this.shclick = true;
    },
    // 获取数据列表
    getDataList(val) {
      console.log(val);
      this.pageIndex = val;
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/works/works/list"),
        method: "get",
        params: this.$http.adornParams({
          deleteStatus: this.deleteStatus,
          reviewStatus: this.reviewStatus,
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key,
        }),
      }).then(({ data }) => {
        if (data && data.code === 20000) {
          this.dataList = data.data.page.list;
          this.totalPage = data.data.page.totalCount;
        } else {
          this.dataList = [];
          this.totalPage = 0;
        }
        this.dataListLoading = false;
      });
    },
    // 每页数
    sizeChangeHandle(val) {
      this.pageSize = val;
      this.getDataList(1);
    },
    // 当前页
    currentChangeHandle(val) {
      this.getDataList(val);
    },
    // 多选
    selectionChangeHandle(val) {
      console.log(this.dataListSelections.length);
      this.dataListSelections = val;
    },
    // 新增 / 修改
    addOrUpdateHandle(id) {
      this.addOrUpdateVisible = true;
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id);
      });
    },
    // 删除
    deleteHandle(id) {
      var ids = id
        ? [id]
        : this.dataListSelections.map((item) => {
            return item.worksId;
          });
      this.$confirm(
        `确定对[id=${ids.join(",")}]进行[${id ? "删除" : "批量删除"}]操作?`,
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).then(() => {
        this.$http({
          url: this.$http.adornUrl("/works/works/delete"),
          method: "post",
          data: this.$http.adornData(ids, false),
        }).then(({ data }) => {
          if (data && data.code === 20000) {
            this.$message({
              message: "操作成功",
              type: "success",
              duration: 1500,
              onClose: () => {
                this.getDataList(1);
              },
            });
          } else {
            this.$message.error(data.msg);
          }
        });
      });
    },
  },
};
</script>
