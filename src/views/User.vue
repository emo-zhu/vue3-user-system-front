<template>
  <div class="user-manage">
    <!--  -->
    <div class="query-form">
      <!-- 
        el-form 表单
        ref="form"      表单验证
        :inline="true"  在一行显示
        :model="user"   数据绑定
      -->
      <el-form ref="form" :inline="true" :model="user">
        <!-- 
        el-form-item  子表单
        label="用户ID"  标签提示
        prop="userId"   
        -->
        <el-form-item label="用户ID" prop="userId">
          <!--  
            el-input  输入框
            v-model="user.userId" 数据绑定
            placeholder="请输入用户ID"  表单提示
          -->
          <el-input v-model="user.userId" placeholder="请输入用户ID" />
        </el-form-item>
        <el-form-item label="用户名称" prop="userName">
          <el-input v-model="user.userName" placeholder="请输入用户名称" />
        </el-form-item>
        <el-form-item label="状态" prop="state">
          <!-- 
            el-select  下拉框
            v-model="user.state"  默认状态
            :value="0"    对应值
            label="所有"   标签提示
           -->
          <el-select v-model="user.state">
            <el-option :value="0" label="所有"></el-option>
            <el-option :value="1" label="在职"></el-option>
            <el-option :value="2" label="离职"></el-option>
            <el-option :value="3" label="试用期"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="handleQuery">查询</el-button>
          <!-- 
            "handleReset('form')"  ref="form" 作为参数
           -->
          <el-button @click="handleReset('form')">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
    <!--  -->
    <div class="base-table">
      <div class="action">
        <!-- 
          v-has="'user-create'"  权限控制 
         -->
        <el-button type="primary" @click="handleCreate" v-has="'user-create'"
          >新增</el-button
        >
        <el-button
          type="danger"
          @click="handlePatchDel"
          v-has="'user-patch-delete'"
          >批量删除</el-button
        >
      </div>
      <!-- 
        el-table 表格
        :data="userList"   表格数据
         @selection-change="handleSelectionChange"  表格多选函数
       -->
      <el-table :data="userList" @selection-change="handleSelectionChange">
        <!-- 
          el-table-column 表格列
          type="selection"   多选类型
          width="55" 
         -->
        <el-table-column type="selection" width="55" />
        <!-- 
           v-for="item in columns"   动态渲染
          :key="item.prop"        
          :prop="item.prop"
          :label="item.label"
          :width="item.width"
          :formatter="item.formatter"
         -->
        <el-table-column
          v-for="item in columns"
          :key="item.prop"
          :prop="item.prop"
          :label="item.label"
          :width="item.width"
          :formatter="item.formatter"
        >
        </el-table-column>
        <el-table-column label="操作" width="160">
          <template #default="scope">
            <el-button
              @click="handleEdit(scope.row)"
              size="small"
              v-has="'user-edit'"
              >编辑</el-button
            >
            <el-button
              type="danger"
              size="small"
              @click="handleDel(scope.row)"
              v-has="'user-delete'"
              >删除</el-button
            >
          </template>
        </el-table-column>
      </el-table>
      <!-- 
        el-pagination  分页
        class="pagination"
        background      颜色
        layout="prev, pager, next"    
        :total="pager.total"    总长度
        :page-size="pager.pageSize"   页码
        @current-change="handleCurrentChange"   切换
       -->
      <el-pagination
        class="pagination"
        background
        layout="prev, pager, next"
        :total="pager.total"
        :page-size="pager.pageSize"
        @current-change="handleCurrentChange"
      />
    </div>
    <!-- 新增/编辑弹框 -->
    <el-dialog width="35%" title="用户新增" v-model="showModal">
      <el-form
        ref="dialogForm"
        :model="userForm"
        label-width="100px"
        :rules="rules"
      >
        <el-form-item label="用户名" prop="userName">
          <el-input
            v-model="userForm.userName"
            :disabled="action == 'edit'"
            placeholder="请输入用户名称"
          />
        </el-form-item>
        <el-form-item label="邮箱" prop="userEmail">
          <el-input
            v-model="userForm.userEmail"
            :disabled="action == 'edit'"
            placeholder="请输入用户邮箱"
          >
            <template #append>@qq.com</template>
          </el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="userForm.mobile" placeholder="请输入手机号" />
        </el-form-item>
        <el-form-item label="岗位" prop="job">
          <el-input v-model="userForm.job" placeholder="请输入岗位" />
        </el-form-item>
        <el-form-item label="状态" prop="state">
          <el-select v-model="userForm.state">
            <el-option :value="1" label="在职"></el-option>
            <el-option :value="2" label="离职"></el-option>
            <el-option :value="3" label="试用期"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="系统角色" prop="roleList">
          <el-select
            v-model="userForm.roleList"
            placeholder="请选择用户系统角色"
            multiple
            style="width: 100%"
          >
            <el-option
              v-for="role in roleList"
              :key="role._id"
              :label="role.roleName"
              :value="role._id"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="部门" prop="deptId">
          <!-- 
            级联选择器
           -->
          <el-cascader
            v-model="userForm.deptId"
            placeholder="请选择所属部门"
            :options="deptList"
            :props="{ checkStrictly: true, value: '_id', label: 'deptName' }"
            clearable
            style="width: 100%"
          ></el-cascader>
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="handleClose">取 消</el-button>
          <el-button type="primary" @click="handleSubmit">确 定</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>
<script>
import { getCurrentInstance, onMounted, reactive, ref, toRaw } from "vue";
import utils from "./../utils/utils";

export default {
  name: "user",
  setup() {
    // 获取Composition API 上下文对象
    const { ctx, proxy } = getCurrentInstance();
    // 初始化用户表单对象
    const user = reactive({
      state: 1,
    });
    // 初始化用户列表数据
    const userList = ref([]);
    // 初始化分页对象
    const pager = reactive({
      pageNum: 1,
      pageSize: 10,
    });
    // 选中用户列表对象
    const checkedUserIds = ref([]);
    // 弹框显示对象
    const showModal = ref(false);
    // 新增用户Form对象
    const userForm = reactive({
      state: 3,
    });
    // 角色列表
    const roleList = ref([]);
    // 部门列表
    const deptList = ref([]);
    // 定义用户操作行为
    const action = ref("add");
    // 定义表单校验规则
    const rules = reactive({
      userName: [
        {
          required: true,
          message: "请输入用户名称",
          trigger: "blur",
        },
      ],
      userEmail: [
        { required: true, message: "请输入用户邮箱", trigger: "blur" },
      ],
      mobile: [
        {
          pattern: /1[3-9]\d{9}/,
          message: "请输入正确的手机号格式",
          trigger: "blur",
        },
      ],
      deptId: [
        {
          required: true,
          message: "请选择部门",
          trigger: "blur",
        },
      ],
    });
    // 定义动态表格-格式
    const columns = reactive([
      {
        label: "用户ID",
        prop: "userId",
      },
      {
        label: "用户名",
        prop: "userName",
      },
      {
        label: "用户邮箱",
        prop: "userEmail",
      },
      {
        label: "用户角色",
        prop: "role",
        formatter(row, column, value) {
          return {
            0: "管理员",
            1: "普通用户",
          }[value];
        },
      },
      {
        label: "用户状态",
        prop: "state",
        formatter(row, column, value) {
          return {
            1: "在职",
            2: "离职",
            3: "试用期",
          }[value];
        },
      },
      {
        label: "注册时间",
        prop: "createTime",
        width: 180,
        formatter: (row, column, value) => {
          return utils.formateDate(new Date(value));
        },
      },
      {
        label: "最后登录时间",
        prop: "lastLoginTime",
        width: 180,
        formatter: (row, column, value) => {
          return utils.formateDate(new Date(value));
        },
      },
    ]);

    // 初始化接口调用
    onMounted(() => {
      getUserList();
      getRoleAllList();
      getDeptList();
    });
    // 获取用户列表
    const getUserList = async () => {
      // 获取用户状态和页码
      let params = { ...user, ...pager };
      // 调用接口获取数据 params 作为参数
      const { list, page } = await proxy.$api.getUserList(params);
      userList.value = list;
      // console.log(userList.value);
      pager.total = page.total;
    };

    // 查询
    const handleQuery = () => {
      // 重新调用用户列表
      getUserList();
    };
    // 重置
    const handleReset = (form) => {
      // 清空表单
      proxy.$refs[form].resetFields();
    };
    // 分页
    const handleCurrentChange = (current) => {
      pager.pageNum = current;
      getUserList();
    };
    // 删除
    const handleDel = async (row) => {
      // 获取row所有数据 ，根据id 删除
      const id = await proxy.$api.userDel({
        userIds: [row.userId], // 可单个删除,也可批量删除
      });
      // console.log([row.userId]);
      proxy.$message.success("删除成功");
      // 重新调用接口更新数据
      getUserList();
    };
    // 批量删除
    const handlePatchDel = async () => {
      if (checkedUserIds.value.length == 0) {
        proxy.$message.error("请选择要删除的用户");
        return;
      }
      // 遍历获取用户id集合  checkedUserIds 复选框数据
      const userId = checkedUserIds.value.map(function (i, index, arr) {
        // console.log(i.userId);
        return i.userId;
      });
      console.log(userId);

      const res = await proxy.$api.userDel({
        // userIds: checkedUserIds.value, // 可单个删除,也可以批量删除
        userIds: userId,
      });

      if (res) {
        proxy.$message.success("删除成功");
        getUserList();
      } else {
        proxy.$message.success("删除失败");
      }
    };
    // 表格多选
    const handleSelectionChange = (list) => {
      let arr = [];
      list.map((item) => {
        arr.push(item.userId);
      });
      checkedUserIds.value = list;
    };
    // 用户新增
    const handleCreate = () => {
      action.value = "add";
      showModal.value = true;
    };
    // 获取角色
    const getRoleAllList = async () => {
      let res = await proxy.$api.getRoleAllList();
      roleList.value = res;
      console.log(roleList.value);
      // console.log(res[0].roleName);
      // console.log(res.rolesName);
    };
    // 获取部门
    const getDeptList = async () => {
      let res = await proxy.$api.getDeptList();
      deptList.value = res;
    };
    // 用户弹窗关闭
    const handleClose = () => {
      showModal.value = false;
      handleReset("dialogForm");
    };
    // 用户提交
    const handleSubmit = () => {
      proxy.$refs.dialogForm.validate(async (valid) => {
        if (valid) {
          let params = toRaw(userForm);
          params.userEmail += "@qq.com";
          params.action = action.value;
          let res = await proxy.$api.userSubmit(params);
          if (res) {
            showModal.value = false;
            proxy.$message.success("用户创建成功");
            handleReset("dialogForm");
            getUserList();
          }
        }
      });
    };
    // 用户编辑
    const handleEdit = (row) => {
      action.value = "edit";
      showModal.value = true;
      proxy.$nextTick(() => {
        Object.assign(userForm, row);
      });
    };
    return {
      user,
      userList,
      columns,
      pager,
      checkedUserIds,
      showModal,
      userForm,
      rules,
      roleList,
      deptList,
      action,
      getUserList,
      handleQuery,
      handleReset,
      handleCurrentChange,
      handleDel,
      handlePatchDel,
      handleSelectionChange,
      handleCreate,
      getRoleAllList,
      getDeptList,
      handleClose,
      handleSubmit,
      handleEdit,
    };
  },
};
</script>

<style lang="scss"></style>
