<template>
  <div class="mod-config">
    <el-form
      :inline="true"
      :model="dataForm"
      @keyup.enter.native="getDataList()"
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
        <el-button @click="getDataList()">查询</el-button>
        <el-button
          v-if="isAuth('works:cartoonworksdetails:save')"
          type="primary"
          @click="addOrUpdateHandle()"
          >新增</el-button
        >
        <el-button
          v-if="isAuth('works:cartoonworksdetails:delete')"
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
        prop="id"
        header-align="center"
        align="center"
        label="id"
      >
      </el-table-column>
      <el-table-column
        prop="worksName"
        header-align="center"
        align="center"
        label="作品名称"
      >
      </el-table-column>
      <el-table-column
        prop="cartoonChapterId"
        header-align="center"
        align="center"
        label="漫画章节ID 第几话"
      >
      </el-table-column>
      <el-table-column
        prop="cartoonChapterName"
        header-align="center"
        align="center"
        label="漫画章节名字"
      >
      </el-table-column>
      <el-table-column
        prop="cartoonPages"
        header-align="center"
        align="center"
        label="漫画页数"
      >
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
        label="逻辑删除状态 0-已删除 1-未删除"
        ><template slot-scope="scope">
          {{ scope.row.deleteStatus == 0 ? "已删除" : "未删除" }}
        </template>
      </el-table-column>
      <el-table-column
        prop="reviewStatus"
        header-align="center"
        align="center"
        label="审核状态 0-审核中 1-审核成功 2-审核失败"
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
            @click="addOrUpdateHandle(scope.row.id)"
            >修改</el-button
          >
          <el-button
            type="text"
            size="small"
            @click="deleteHandle(scope.row.id)"
            >删除</el-button
          >
          <el-button type="text" size="small" @click="shHandle(scope.row)"
            >审核</el-button
          >
          <el-button type="text" size="small" @click="xzHandle(scope.row)"
            >下载作品章节</el-button
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
      @refreshDataList="getDataList"
    ></add-or-update>

    <el-dialog
      title="审核"
      :close-on-click-modal="false"
      :visible.sync="shclick"
    >
      <el-form :model="shdialog" ref="shdialog" label-width="80px">
        <el-form-item label="章节ID" prop="ID">
          <el-input
            v-model="shdialog.id"
            placeholder="章节ID"
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
          <el-radio-group v-model="shdialog.reviewStatus">
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

    <el-dialog
      title="编辑章节下载目录"
      :close-on-click-modal="false"
      :visible.sync="shclick2"
    >
      <el-form :model="shdialog" ref="shdialog" label-width="80px">
        <el-form-item label="章节ID" prop="ID">
          <el-input
            v-model="shdialog.id"
            placeholder="章节ID"
            :disabled="true"
          ></el-input>
        </el-form-item>
        <el-form-item label="章节存放地址" prop="ZipFilePath">
          <el-input
            v-model="shdialog.zipFilePath"
            placeholder="C:\\Users\\F3863479\\Desktop\\TestZip"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="shclick2 = false">取消</el-button>
        <el-button type="primary" @click="dataFormSubmit2()">确定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import AddOrUpdate from "./cartoonworksdetails-add-or-update";
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
      shdialog: {}, // 章节审核data
      shclick: false, // 框的显示和隐藏
      shclick2: false,
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
    this.getDataList();
  },
  methods: {
    dataFormSubmit2() {
      console.log(this.shdialog);
      this.$http({
        url: this.$http.adornUrl("/works/cartoonworksdetails/getChapterWorksZip"),
        method: "get",
        params: {
          worksChapterId:this.shdialog.id ,
          zipFilePath:this.shdialog.zipFilePath
        },
      }).then(({ data }) => {
        if (data && data.code === 20000) {
          this.$message({
            showClose: true,
            message: "成功",
            type: "success",
          });
          this.getDataList();
        } else {
          this.$message({
            showClose: true,
            message: "失败",
            type: "error",
          });
        }
        this.shclick2 = false;

        this.getDataList();
      });
    },
    // 下載章節目錄
    xzHandle(row) {
      // console.log(row);
      this.shdialog = row;
      this.shclick2 = true;
    },
    // 发出请求审核
    dataFormSubmit() {
      this.$http({
        url: this.$http.adornUrl("/works/cartoonworksdetails/review"),
        method: "post",
        data: JSON.stringify(this.shdialog),
      }).then(({ data }) => {
        if (data && data.code === 20000) {
          this.$message({
            showClose: true,
            message: "审核成功",
            type: "success",
          });
          this.getDataList();
        } else {
          this.$message({
            showClose: true,
            message: "审核失败",
            type: "error",
          });
        }
        this.shclick = false;

        this.getDataList();
      });
    },
    // 开启审核框
    shHandle(row) {
      this.shdialog = row;
      this.shclick = true;
    },
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/works/cartoonworksdetails/getReviewList"),
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
          console.log(data);
          this.dataList = data.data.page.list;
          console.log(this.dataList);
          this.totalPage = data.data.page.total;
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
      this.pageIndex = 1;
      this.getDataList();
    },
    // 当前页
    currentChangeHandle(val) {
      this.pageIndex = val;
      this.getDataList();
    },
    // 多选
    selectionChangeHandle(val) {
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
            return item.id;
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
          url: this.$http.adornUrl("/works/cartoonworksdetails/delete"),
          method: "post",
          data: this.$http.adornData(ids, false),
        }).then(({ data }) => {
          if (data && data.code === 20000) {
            this.$message({
              message: "操作成功",
              type: "success",
              duration: 1500,
              onClose: () => {
                this.getDataList();
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
