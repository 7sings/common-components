<template>
  <div id="user">
    <eletable
      :dataSource="dataSource"
      :row-class-name="rowStyle"
      :columns="columns"
      :searchProps="searchProps"
      @changeTableData="dataChange"
      :loading="loading"
      :init-table-data="tableData"
      :add-data="showDialog"
      excel-name="病人信息"
      :buttons="buttons"
    >
    </eletable>

    <el-dialog
      title="病人登记"
      class="pane"
      :visible.sync="dialogVisible"
      v-if="dialogVisible"
    >
      <el-form
        label-width="100px"
        :model="formData"
        ref="createForm"
        :rules="addRules"
      >
        <el-form-item label="身份证号码" prop="idcard">
          <el-input v-model="formData.idcard" maxlength="18"></el-input>
        </el-form-item>

        <!-- <el-form-item label="住院日期">
                <el-date-picker  v-model="formData.time" type="datetime"></el-date-picker>
            </el-form-item> -->

        <el-form-item label="姓名" prop="name">
          <el-input
            v-model="formData.name"
            maxlength="18"
            style="width: 100px"
          ></el-input>
        </el-form-item>

        <el-form-item label="性别" prop="sex">
          <el-radio-group v-model="formData.sex">
            <el-radio label="男"></el-radio>
            <el-radio label="女"></el-radio>
          </el-radio-group>
        </el-form-item>

        <el-form-item label="电话号码" prop="tel">
          <el-input
            v-model="formData.tel"
            maxlength="18"
            style="width: 180px"
          ></el-input>
        </el-form-item>

        <el-form-item label="押金" prop="deposit">
          <el-input
            v-model="formData.deposit"
            maxlength="18"
            style="width: 180px"
            type="number"
          ></el-input>
        </el-form-item>

        <el-form-item label="科室" prop="department">
          <el-select
            v-model.number="formData.departmentId"
            placeholder="请选择"
            @visible-change="getDepartments"
          >
            <el-option
              v-for="item in departments"
              :key="item.departmentId"
              :label="item.name"
              :value="item.id"
            >
            </el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="主治医师" prop="doctor">
          <el-select
            v-model.number="formData.doctorId"
            placeholder="请选择"
            @visible-change="getDoctors"
          >
            <el-option
              v-for="item in doctors"
              :key="item.id"
              :label="item.info"
              :value="item.id"
            >
            </el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="床号" prop="bed">
          <el-select
            v-model.number="formData.bedId"
            placeholder="请选择"
            @visible-change="getCreateBeds"
          >
            <el-option
              v-for="item in bedIDs"
              :key="item.bedId"
              :label="item.info"
              :value="item.bedId"
            >
            </el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="生日" prop="birthday">
          <el-date-picker
            v-model="formData.birthday"
            type="date"
          ></el-date-picker>
        </el-form-item>

        <el-form-item>
          <el-button type="primary" @click="addPatient">立即创建</el-button>
          <el-button @click="dialogVisible = false">取消</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>

    <el-dialog
      title="修改病人信息"
      class="pane"
      :visible.sync="updateDialogVisible"
    >
      <el-form
        label-width="100px"
        :model="updateFormData"
        ref="updateForm"
        :rules="updateRules"
      >
        <el-form-item label="身份证号码" prop="idcard">
          <el-input v-model="updateFormData.idcard" maxlength="18"></el-input>
        </el-form-item>

        <el-form-item label="姓名" prop="name">
          <el-input
            v-model="updateFormData.name"
            maxlength="18"
            style="width: 100px"
          ></el-input>
        </el-form-item>

        <el-form-item label="性别">
          <el-radio-group v-model="updateFormData.sex">
            <el-radio label="男"></el-radio>
            <el-radio label="女"></el-radio>
          </el-radio-group>
        </el-form-item>

        <el-form-item label="电话号码" prop="tel">
          <el-input
            v-model="updateFormData.tel"
            maxlength="18"
            style="width: 180px"
          ></el-input>
        </el-form-item>

        <el-form-item label="床号" prop="bed">
          <el-select
            v-model="updateFormData.bedNo"
            placeholder="请选择"
            @visible-change="getBeds"
            @change="getBedNo"
          >
            <el-option
              v-for="item in bedIDs"
              :key="item.bedId"
              :label="item.bedNo"
              :value="item.bedNo"
            >
            </el-option>
          </el-select>
        </el-form-item>

        <el-form-item>
          <el-button type="primary" @click="updatePatient">保存</el-button>
          <el-button @click="updateDialogVisible = false">取消</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>

<script>
import Vue from "vue";
import qs from "qs";
import eletable from "../components/Table.vue";

export default {
  name: "patients",
  components: {
    eletable,
  },
  data() {
    return {
      // dataOptions: {
      //     disabledDate(time) {
      //         return time.getTime() < Date.now() - 8.64e7;
      //     },
      // },
      doctors: [],
      departments: [],
      dept: "",
      bedId: [],
      bedIDs: [],
      updateFormData: {},
      formData: {},
      addRules: {
        idcard: [
          {
            required: true,
            message: "请输入身份证号码",
            trigger: "blur",
          },
          {
            pattern:
              /^[1-9]\d{7}((0\d)|(1[0-2]))(([0|1|2]\d)|3[0-1])\d{3}$|^[1-9]\d{5}[1-9]\d{3}((0\d)|(1[0-2]))(([0|1|2]\d)|3[0-1])\d{3}([0-9]|X)$/,
            message: "请输入有效的身份证号码",
            trigger: "blur",
          },
        ],
        name: [
          {
            required: true,
            message: "请输入姓名",
            trigger: "blur",
          },
          {
            pattern: /^[\u4E00-\u9FA5\uf900-\ufa2d·s]{2,20}$/,
            message: "请输入正确的姓名",
            trigger: "blur",
          },
        ],
        sex: [
          {
            required: true,
            message: "请选择性别",
            trigger: "change",
          },
        ],
        tel: [
          {
            required: true,
            message: "请输入电话号码",
            trigger: "blur",
          },
          {
            pattern: /^1[34578]\d{9}$/,
            message: "请输入正确的电话号码",
            trigger: "blur",
          },
        ],
        deposit: [
          {
            required: true,
            message: "请输入有效押金金额",
            trigger: "blur",
          },
        ],
        // department:[
        //   {required:true,message:'请选择科室'}
        // ],
        // doctor:[
        //   {required:true,message:'请选择医生',trigger:'blur'}
        // ],
        // bed:[
        //   {required:true,message:'请选择床号',trigger:'blur'}
        // ],
        birthday: [
          {
            required: true,
            message: "请填写生日",
            trigger: "blur",
          },
        ],
      },
      updateRules: {
        idcard: [
          {
            pattern:
              /^[1-9]\d{7}((0\d)|(1[0-2]))(([0|1|2]\d)|3[0-1])\d{3}$|^[1-9]\d{5}[1-9]\d{3}((0\d)|(1[0-2]))(([0|1|2]\d)|3[0-1])\d{3}([0-9]|X)$/,
            message: "请输入有效的身份证号码",
            trigger: "blur",
          },
        ],
        name: [
          {
            pattern: /^[\u4E00-\u9FA5\uf900-\ufa2d·s]{2,20}$/,
            message: "请输入正确的姓名",
            trigger: "blur",
          },
        ],

        tel: [
          {
            pattern: /^1[345789]\d{9}$/,
            message: "请输入正确的电话号码",
            trigger: "blur",
          },
        ],
      },
      updateDialogVisible: false,
      dialogVisible: false,
      tableData: [],
      loading: true,
      searchProps: {
        prompt: "姓名",
        prop: "name",
      },
      buttons: [
        {
          id: 1,
          type: "warning",
          label: "编辑",
          operate: ($index, row) => {
            this.updateDialogVisible = true;
            this.updateFormData = row;
          },
        },
        {
          id: 2,
          type: "primary",
          label: "详情",
          operate: (index, row) => {
            this.$router.push(`/patientdetail/${row.patientId}`);
          },
        },
        {
          id: 3,
          type: "success",
          label: "出院",
          operate: (index, row) => {
            const url = this.common.server + "/state/leave";
            this.$confirm(
              "此操作将让住院号为" +
                row.patientId +
                "的病人" +
                row.name +
                "出院, 是否继续?",
              "提示",
              {
                confirmButtonText: "确定",
                cancelButtonText: "取消",
                type: "warning",
              }
            )
              .then(() => {
                var data = {
                  patientId: row.patientId,
                };
                this.$axios
                  .post(url, qs.stringify(data))
                  .then((response) => {
                    if (response.data.code == 200) {
                      this.$message({
                        type: "success",
                        message: response.data.msg,
                      });
                      this.$router.go(0);
                    } else {
                      this.$message({
                        type: "error",
                        message: response.data.msg,
                      });
                    }
                  })
                  .catch((err) => {
                    this.$message({
                      type: "error",
                      message: "服务器错误！",
                    });
                  });
              })
              .catch(() => {
                this.$message({
                  type: "info",
                  message: "已取消操作",
                });
              });
          },
        },
      ],
      dataSource: [], // 数据源
      columns: [
        {
          hasSort: false, //<Boolean> 是否排序
          isShow: true, //<Boolean> 是否展示
          prop: "patientId", //<String>  对应属性名
          label: "病人编号", //<String>   表头标签
          align: "center", //对齐方式
          width: 160, //列宽
        },
        {
          hasSort: false, //<Boolean> 是否排序
          isShow: true, //<Boolean> 是否展示
          prop: "idcard", //<String>  对应属性名
          label: "身份证号", //<String>   表头标签
          align: "center", //列宽
          width: 160,
        },
        {
          type: "date", //字段类型

          hasSort: false, //<Boolean> 是否排序
          isShow: true, //<Boolean> 是否展示
          prop: "time", //<String>  对应属性名
          label: "住院日期", //<String>   表头标签
          align: "center", //列宽
          width: 100,
        },
        {
          hasSort: false, //<Boolean> 是否排序
          isShow: true, //<Boolean> 是否展示
          prop: "name", //<String>  对应属性名
          label: "姓名", //<String>   表头标签
          align: "center", //列宽
          width: 100,
        },
        {
          hasSort: true, //<Boolean> 是否排序
          isShow: true, //<Boolean> 是否展示
          prop: "leaved", //<String>  对应属性名
          label: "是否出院", //<String>   表头标签
          align: "center", //列宽
          width: 120,
        },
        {
          hasSort: false, //<Boolean> 是否排序
          isShow: true, //<Boolean> 是否展示
          prop: "sex", //<String>  对应属性名
          label: "性别", //<String>   表头标签
          align: "center", //列宽
          width: 80,
        },

        {
          type: "date", //字段类型

          hasSort: false, //<Boolean> 是否排序
          isShow: true, //<Boolean> 是否展示
          prop: "birthday", //<String>  对应属性名
          label: "出生日期", //<String>   表头标签
          align: "center", //列宽
          width: 120,
        },
        {
          hasSort: false, //<Boolean> 是否排序
          isShow: true, //<Boolean> 是否展示
          prop: "tel", //<String>  对应属性名
          label: "电话", //<String>   表头标签
          align: "center",
          width: 120, // 列宽
        },

        {
          hasSort: false, //<Boolean> 是否排序
          isShow: true, //<Boolean> 是否展示
          prop: "doctorName", //<String>  对应属性名
          label: "主治医生", //<String>   表头标签
          align: "center",
          width: 80,
        },
        {
          hasSort: false, //<Boolean> 是否排序
          isShow: true, //<Boolean> 是否展示
          prop: "number", //<String>  对应属性名
          label: "病历号", //<String>   表头标签
          align: "center",
          width: 150,
        },
        {
          hasSort: true, //<Boolean> 是否排序
          isShow: true, //<Boolean> 是否展示
          prop: "bedNo", //<String>  对应属性名
          label: "床号", //<String>   表头标签
          align: "center",
          width: 120,
        },
      ],
    };
  },
  computed: {},
  created() {
    const url = this.common.server + "/patient/all";

    // const url = "/api/hos/patient/all";
    this.$axios
      .post(url)
      .then((response) => {
        this.dataSource = response.data;
        // console.log(this.dataSource);
        this.loading = false;
        this.tableData = response.data;
        console.log(this.tableData);
      })
      .catch((error) => {
        this.dataSource = [
          {
            idcard: "服务器错误",
            name: "wzq",
          },
          {
            idcard: "服务器错误",
            name: "qm",
          },
        ];

        this.loading = false;
        this.tableData = this.dataSource;
      });
  },
  methods: {
    rowStyle({ row }) {
      if (row.leaved === "已出院") {
        return "background-color: oldlace";
      }
    },
    detail(index, row) {
      this.$router.push(`/patientdetail/${row.patientId}`);
    },
    getCreateBeds() {
      const url = this.common.server + "/ward/free";
      var data = {
        departmentId: this.formData.departmentId,
      };
      this.$axios.post(url, qs.stringify(data)).then((response) => {
        // console.log(response.data);
        this.bedIDs = response.data;
        for (var i = 0; i < this.bedIDs.length; i++) {
          Vue.set(
            this.bedIDs[i],
            "info",
            this.bedIDs[i].wardNo + "室" + this.bedIDs[i].bedNo
          );
        }
      });
    },
    getDoctors() {
      const url = this.common.server + "/doctor/free";
      var data = {
        departmentId: this.formData.departmentId,
      };
      this.$axios.post(url, qs.stringify(data)).then((response) => {
        // console.log(response.data);
        this.doctors = response.data;
        for (var i = 0; i < this.doctors.length; i++) {
          Vue.set(
            this.doctors[i],
            "info",
            this.doctors[i].doctorName + "/" + this.doctors[i].count + "人"
          );
          // this.doctors[i].info = this.doctors[i].info,this.doctors[i].doctorName+'/'+this.doctors[i].count+'人';
        }
      });
    },
    getDept() {
      // console.log(this.formData.departmentId)
    },

    getDepartments() {
      const url = this.common.server + "/department/all";
      this.$axios.post(url).then((response) => {
        this.departments = response.data;
      });
    },

    getBedNo(val) {
      var bed = this.bedIDs.filter((item) => item.bedNo == val);
      this.updateFormData.bedId = bed[0].bedId;
      // console.log(this.updateFormData.bedId);
    },

    getBeds() {
      const url = this.common.server + "/ward/free";
      var data = {
        departmentId: this.updateFormData.departmentId,
      };
      this.$axios.post(url, qs.stringify(data)).then((response) => {
        // console.log(response.data);
        this.bedIDs = response.data;
      });
    },
    addPatient() {
      const url = this.common.server + "/patient/add";
      // const url = "api/hos/patient/add";
      this.$refs.createForm.validate((valid) => {
        console.log(this.formData);
        if (valid) {
          if (!this.formData.departmentId) {
            this.$message("请选择科室！");
            return false;
          }
          if (!this.formData.doctorId) {
            this.$message("请选择医生！");
            return false;
          }
          if (!this.formData.bedId) {
            this.$message("请选择床号！");
            return false;
          }
          this.$axios
            .post(url, qs.stringify(this.formData))
            .then((response) => {
              this.$message("添加成功!");
              this.$router.go(0);
            })
            .catch((error) => {
              this.$message("添加失败，请检查该病人信息已存在！");
            });
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    },
    showDialog() {
      this.dialogVisible = true;
    },
    dataChange(val) {
      // console.log(val);
      this.dataSource = val;
    },

    edit(index, row) {
      this.updateDialogVisible = true;
      this.updateFormData = row;
      // console.log(row);
    },

    updatePatient() {
      const url = this.common.server + "/patient/update";
      // const url = "/api/hos/patient/update";

      this.$axios
        .post(url, qs.stringify(this.updateFormData))
        .then((response) => {
          // console.log(response.data);
          if (response.data.code == 200) {
            this.$message("更新成功！");
            setTimeout(() => {
              this.$router.go(0);
            }, 100);
          }
        })
        .catch((error) => {
          console.log(error);
        });
    },
  },
};
</script>

<style scoped>
.excel {
  text-align: center;
  margin: 20px;
}
</style>
