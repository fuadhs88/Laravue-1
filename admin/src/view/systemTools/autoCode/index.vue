<template>
  <div>
    <!-- 从数据库直接获取字段 -->
    <el-collapse v-model="activeNames">
      <el-collapse-item name="1">
        <template slot="title">
          <div :style="{ fontSize: '16px', paddingLeft: '20px' }">
            点这里从现有数据库创建代码
            <i class="header-icon el-icon-thumb"></i>
          </div>
        </template>
        <el-form
          ref="getTableForm"
          :inline="true"
          :model="dbform"
          label-width="120px"
        >
          <el-form-item label="数据库名" prop="className">
            <el-select
              @change="getTable"
              v-model="dbform.dbName"
              filterable
              placeholder="请选择数据库"
            >
              <el-option
                v-for="item in dbOptions"
                :key="item.database"
                :label="item.database"
                :value="item.database"
              ></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="表名" prop="className">
            <el-select
              v-model="dbform.tableName"
              :disabled="!dbform.dbName"
              filterable
              placeholder="请选择表"
            >
              <el-option
                v-for="item in tableOptions"
                :key="item.tableName"
                :label="item.tableName"
                :value="item.tableName"
              ></el-option>
            </el-select>
          </el-form-item>
          <el-form-item>
            <el-button @click="getColume" type="primary"
              >使用此表创建</el-button
            >
          </el-form-item>
        </el-form>
      </el-collapse-item>
    </el-collapse>

    <!-- <el-divider></el-divider> -->
    <br />
    <!-- 初始版本自动化代码工具 -->
    <el-form
      ref="autoCodeForm"
      :rules="rules"
      :model="form"
      label-width="120px"
      :inline="true"
    >
      <el-form-item label="nameSpace" prop="nameSpace">
        <el-input v-model="form.nameSpace" placeholder="Business"></el-input>
      </el-form-item>
      <el-form-item label="ClassName" prop="className">
        <el-input
          v-model="form.className"
          placeholder="首字母自动转换大写"
        ></el-input>
      </el-form-item>
      <el-form-item label="tableName" prop="tableName">
        <el-input v-model="form.tableName" placeholder="指定表名"></el-input>
      </el-form-item>
      <el-form-item label="primaryKey" prop="primaryKey">
        <el-input v-model="form.primaryKey" placeholder="id"></el-input>
      </el-form-item>
      <el-form-item label="同时代码实现">
        <el-checkbox v-model="form.autoCode"></el-checkbox>
      </el-form-item>
    </el-form>
    <!-- 组件列表 -->
    <div class="button-box clearflex">
      <el-button @click="editAndAddField()" type="primary">新增Field</el-button>
    </div>
    <el-table :data="form.fields" border stripe>
      <el-table-column type="index" label="序列" width="100"></el-table-column>
      <el-table-column
        prop="fieldName"
        width="120"
        label="Field名"
      ></el-table-column>
      <el-table-column
        prop="fieldType"
        label="Field数据类型"
        width="130"
      ></el-table-column>
      <el-table-column
        prop="dataType"
        label="数据库字段类型"
        width="130"
      ></el-table-column>
      <el-table-column
        prop="dataTypeLong"
        label="数据库字段长度"
        width="130"
      ></el-table-column>
      <el-table-column
        prop="columnName"
        label="数据库字段"
        width="130"
      ></el-table-column>
      <el-table-column
        prop="comment"
        label="数据库字段描述"
        width="130"
      ></el-table-column>
      <el-table-column
        prop="fieldSearchType"
        label="搜索条件"
        width="130"
      ></el-table-column>
      <el-table-column
        prop="dictType"
        label="字典"
        width="130"
      ></el-table-column>
      <el-table-column label="操作" width="300">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="primary"
            icon="el-icon-edit"
            @click="editAndAddField(scope.row)"
            >编辑</el-button
          >
          <el-button
            size="mini"
            type="text"
            :disabled="scope.$index == 0"
            @click="moveUpField(scope.$index)"
            >上移</el-button
          >
          <el-button
            size="mini"
            type="text"
            :disabled="scope.$index + 1 == form.fields.length"
            @click="moveDownField(scope.$index)"
            >下移</el-button
          >
          <el-popover placement="top" v-model="scope.row.visible">
            <p>确定删除吗？</p>
            <div style="text-align: right; margin: 0">
              <el-button
                size="mini"
                type="text"
                @click="scope.row.visible = false"
                >取消</el-button
              >
              <el-button
                type="primary"
                size="mini"
                @click="deleteField(scope.$index)"
                >确定</el-button
              >
            </div>
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-delete"
              slot="reference"
              >删除</el-button
            >
          </el-popover>
        </template>
      </el-table-column>
    </el-table>
    <el-tag type="danger"
      >id , created_at , updated_at , deleted_at 会自动生成请勿重复创建</el-tag
    >
    <!-- 组件列表 -->
    <div class="button-box clearflex">
      <el-button @click="enterForm" type="primary">生成代码包</el-button>
    </div>
    <!-- 组件弹窗 -->
    <el-dialog title="组件内容" :visible.sync="dialogFlag">
      <FieldDialog
        v-if="dialogFlag"
        :dialogMiddle="dialogMiddle"
        ref="fieldDialog"
      />
      <div slot="footer" class="dialog-footer">
        <el-button @click="closeDialog">取 消</el-button>
        <el-button type="primary" @click="enterDialog">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
const fieldTemplate = {
  fieldName: "",
  fieldDesc: "",
  fieldType: "",
  dataType: "",
  fieldJson: "",
  columnName: "",
  dataTypeLong: "",
  comment: "",
  fieldSearchType: "",
  dictType: "",
};

import FieldDialog from "@/view/systemTools/autoCode/component/fieldDialog.vue";
import { toUpperCase, toHump } from "@/utils/stringFun.js";
import { createTemp, getDB, getTable, getColume } from "@/api/autoCode.js";
import { getDict } from "@/utils/dictionary";

export default {
  name: "autoCode",
  components: {
    FieldDialog,
  },
  data() {
    return {
      activeNames: [""],
      dbform: {
        dbName: "",
        tableName: "",
      },
      dbOptions: [],
      tableOptions: [],
      addFlag: "",
      fdMap: {},
      form: {
        nameSpace: "", // 命名空间
        className: "",         // 类名
        tableName: "",         // 表名
        apiName: "",           // API地址名（小驼峰）
        primaryKey: "id",      // 模型主键
        columns: [],           // 模型列
        autoCode: false,       // 是否直接生成代码
        fields: [],            // 字段用于vue文件生成处理
      },
      rules: {
        nameSpace: [
          { required: true, message: "请输入命名空间名称", trigger: "blur" },
        ],
        className: [
          { required: true, message: "请输入结构体名称", trigger: "blur" },
        ],
        tableName: [
          { required: true, message: "请输入数据库表名称", trigger: "blur" },
        ],
        primaryKey: [
          { required: true, message: "请输入主键字段", trigger: "blur" },
        ],
      },
      dialogMiddle: {},
      bk: {},
      dialogFlag: false,
    };
  },
  methods: {
    editAndAddField(item) {
      this.dialogFlag = true;
      if (item) {
        this.addFlag = "edit";
        this.bk = JSON.parse(JSON.stringify(item));
        this.dialogMiddle = item;
      } else {
        this.addFlag = "add";
        this.dialogMiddle = JSON.parse(JSON.stringify(fieldTemplate));
      }
    },
    moveUpField(index) {
      if (index == 0) {
        return;
      }
      const oldUpField = this.form.fields[index - 1];
      this.form.fields.splice(index - 1, 1);
      this.form.fields.splice(index, 0, oldUpField);
    },
    moveDownField(index) {
      const fCount = this.form.fields.length;
      if (index == fCount - 1) {
        return;
      }
      const oldDownField = this.form.fields[index + 1];
      this.form.fields.splice(index + 1, 1);
      this.form.fields.splice(index, 0, oldDownField);
    },
    enterDialog() {
      this.$refs.fieldDialog.$refs.fieldDialogFrom.validate((valid) => {
        if (valid) {
          this.dialogMiddle.fieldName = toUpperCase(
            this.dialogMiddle.fieldName
          );
          if (this.addFlag == "add") {
            this.form.fields.push(this.dialogMiddle);
          }
          this.dialogFlag = false;
        } else {
          return false;
        }
      });
    },
    closeDialog() {
      if (this.addFlag == "edit") {
        this.dialogMiddle = this.bk;
      }
      this.dialogFlag = false;
    },
    deleteField(index) {
      this.form.fields.splice(index, 1);
    },
    async enterForm() {
      if (this.form.fields.length <= 0) {
        this.$message({
          type: "error",
          message: "请填写至少一个field",
        });
        return false;
      }
      if (
        this.form.fields.some((item) => item.fieldName == this.form.className)
      ) {
        this.$message({
          type: "error",
          message: "存在与类同名的字段",
        });
        return false;
      }
      var tmpArr = ["id", "created_at", "updated_at", "deleted_at"];
      this.form.columns = JSON.stringify(tmpArr.concat(this.form.columns));
      if (this.form.columns.length > 0) {
        this.form.fields.forEach(function (item) {
          tmpArr.push(item.columnName);
        });
      }
      this.form.columns = JSON.stringify(tmpArr);
      this.$refs.autoCodeForm.validate(async (valid) => {
        if (valid) {
          this.form.className = toUpperCase(this.form.className);
          const data = await createTemp(this.form).catch(res => console.log(res));
          const blob = new Blob([data]);
          const fileName = "Laravue.zip";
          if ("download" in document.createElement("a")) {
            // 不是IE浏览器
            let url = window.URL.createObjectURL(blob);
            let link = document.createElement("a");
            link.style.display = "none";
            link.href = url;
            link.setAttribute("download", fileName);
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link); // 下载完成移除元素
            window.URL.revokeObjectURL(url); // 释放掉blob对象
          } else {
            // IE 10+
            window.navigator.msSaveBlob(blob, fileName);
          }
        } else {
          return false;
        }
      });
    },
    async getDb() {
      const res = await getDB();
      if (res.code == 200) {
        this.dbOptions = res.data.dbs;
      }
    },
    async getTable() {
      const res = await getTable({ dbName: this.dbform.dbName });
      if (res.code == 200) {
        this.tableOptions = res.data.tables;
      }
      this.dbform.tableName = "";
    },
    async getColume() {
      const gormModelList = ["id", "created_at", "updated_at", "deleted_at"];
      const res = await getColume(this.dbform);
      if (res.code == 200) {
        const tbHump = toHump(this.dbform.tableName);
        this.form.className = toUpperCase(tbHump);
        this.form.tableName = this.dbform.tableName;
        this.form.apiName = tbHump;
        this.form.autoCode = false;
        this.form.fields = [];
        res.data.columes &&
          res.data.columes.map((item) => {
            if (!gormModelList.some((gormfd) => gormfd == item.columeName)) {
              const fbHump = item.columeName;
              this.form.columns.push(item.columeName);
              this.form.fields.push({
                fieldName: item.columeName,
                fieldDesc: item.columeComment || fbHump + "字段",
                fieldType: this.fdMap[item.dataType],
                dataType: item.dataType,
                fieldJson: fbHump,
                dataTypeLong: item.dataTypeLong,
                columnName: item.columeName,
                comment: item.columeComment,
                fieldSearchType: "",
                dictType: "",
              });
            }
          });
      }
    },
    async setFdMap() {
      const fdTpyes = ["string", "int", "bool", "float64", "time.Time"];
      fdTpyes.map(async (fdtype) => {
        const res = await getDict(fdtype);
        res.map((item) => {
          this.fdMap[item.label] = fdtype;
        });
      });
    },
  },
  created() {
    this.getDb();
    this.setFdMap();
  },
};
</script>
<style scope lang="scss">
.button-box {
  padding: 10px 20px;
  .el-button {
    float: right;
  }
}
</style>
