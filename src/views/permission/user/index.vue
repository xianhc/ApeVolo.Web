<template>
  <div class="app-container">
    <el-row :gutter="20">
      <!--侧边部门数据-->
      <el-col :xs="9" :sm="6" :md="5" :lg="4" :xl="4">
        <div class="head-container">
          <el-input
            v-model="deptName"
            clearable
            size="small"
            placeholder="输入部门名称搜索"
            prefix-icon="el-icon-search"
            class="filter-item"
            @input="getDeptDatas"
          />
        </div>
        <el-tree
          :data="deptDatas"
          :load="getDeptDatas"
          :props="defaultProps"
          :expand-on-click-node="false"
          lazy
          @node-click="handleNodeClick"
        />
      </el-col>
      <!--用户数据-->
      <el-col :xs="15" :sm="18" :md="19" :lg="20" :xl="20">
        <!--工具栏-->
        <div class="head-container">
          <div v-if="crud.props.searchToggle">
            <!-- 搜索 -->
            <el-input
              v-model="query.keyWords"
              clearable
              size="small"
              placeholder="输入名称或者邮箱搜索"
              style="width: 200px"
              class="filter-item"
              @keyup.enter.native="crud.toQuery"
            />
            <date-range-picker v-model="query.createTime" class="date-item" />
            <el-select
              v-model="query.enabled"
              clearable
              size="small"
              placeholder="状态"
              class="filter-item"
              style="width: 90px"
              @change="crud.toQuery"
            >
              <el-option
                v-for="item in dict.user_status"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              />
            </el-select>
            <rrOperation />
          </div>
          <crudOperation show="" :permission="permission" />
        </div>
        <!--表单渲染-->
        <el-dialog
          append-to-body
          :close-on-click-modal="false"
          :before-close="crud.cancelCU"
          :visible.sync="crud.status.cu > 0"
          :title="crud.status.title"
          width="570px"
        >
          <el-form
            ref="form"
            :inline="true"
            :model="form"
            :rules="rules"
            size="small"
            label-width="66px"
          >
            <el-form-item label="用户名" prop="username">
              <el-input v-model="form.username" />
            </el-form-item>
            <el-form-item label="电话" prop="phone">
              <el-input v-model.number="form.phone" />
            </el-form-item>
            <el-form-item label="昵称" prop="nickName">
              <el-input v-model="form.nickName" />
            </el-form-item>
            <el-form-item label="邮箱" prop="email">
              <el-input v-model="form.email" />
            </el-form-item>
            <el-form-item label="部门" prop="dept.id">
              <treeselect
                v-model="form.dept.id"
                :options="depts"
                :load-options="loadDepts"
                style="width: 178px"
                placeholder="选择部门"
              />
            </el-form-item>
            <el-form-item label="岗位" prop="jobs">
              <el-select
                v-model="jobDatas"
                value-key="id"
                style="width: 178px"
                multiple
                placeholder="请选择"
                @remove-tag="deleteTag"
                @change="changeJob"
              >
                <el-option
                  v-for="item in jobs"
                  :key="item.id"
                  :label="item.name"
                  :value="item"
                />
              </el-select>
            </el-form-item>
            <el-form-item label="性别">
              <el-radio-group v-model="form.gender" style="width: 178px">
                <el-radio label="男">男</el-radio>
                <el-radio label="女">女</el-radio>
              </el-radio-group>
            </el-form-item>
            <el-form-item label="状态">
              <el-radio-group
                v-model="form.enabled"
                :disabled="form.id === user.id"
              >
                <el-radio
                  v-for="item in dict.user_status"
                  :key="item.id"
                  :label="item.value === 'true'"
                >{{ item.label }}
                </el-radio>
              </el-radio-group>
            </el-form-item>
            <el-form-item style="margin-bottom: 0" label="角色" prop="roles">
              <el-select
                v-model="roleDatas"
                value-key="id"
                style="width: 437px"
                multiple
                placeholder="请选择"
                @remove-tag="deleteTag"
                @change="changeRole"
              >
                <el-option
                  v-for="item in roles"
                  :key="item.id"
                  :disabled="level !== 1 && item.level <= level"
                  :label="item.name"
                  :value="item"
                />
              </el-select>
            </el-form-item>
            <el-form-item style="margin-bottom: 0" label="租户" prop="tenants">
              <el-select
                v-model="form.tenantId"
                value-key="id"
                style="width: 437px"
                clearable
              >
                <el-option
                  v-for="item in tenants"
                  :key="item.tenantId"
                  :label="item.name"
                  :value="parseInt(item.tenantId)"
                />
              </el-select>
            </el-form-item>
          </el-form>
          <div slot="footer" class="dialog-footer">
            <el-button type="text" @click="crud.cancelCU">取消</el-button>
            <el-button
              :loading="crud.status.cu === 2"
              type="primary"
              @click="crud.submitCU"
            >确认
            </el-button>
          </div>
        </el-dialog>
        <!--表格渲染-->
        <el-table
          ref="table"
          v-loading="crud.loading"
          :data="crud.data"
          style="width: 100%"
          @selection-change="crud.selectionChangeHandler"
        >
          <el-table-column
            :selectable="checkboxT"
            type="selection"
            width="55"
          />
          <el-table-column
            width="80"
            label="头像"
          >
            <img :src="user.avatarPath ? baseApi + user.avatarPath : Avatar" class="avatar" alt="用户头像">
          </el-table-column>
          <el-table-column
            :show-overflow-tooltip="true"
            prop="username"
            width="135"
            label="用户名"
          />
          <el-table-column
            :show-overflow-tooltip="true"
            prop="nickName"
            width="150"
            label="昵称"
          />
          <el-table-column
            :show-overflow-tooltip="true"
            prop="phone"
            width="100"
            label="电话"
          />
          <el-table-column
            :show-overflow-tooltip="true"
            width="135"
            prop="email"
            label="邮箱"
          />
          <el-table-column
            :show-overflow-tooltip="true"
            width="200"
            label="角色"
          >
            <template slot-scope="scope">
              <el-tag
                v-for="role in scope.row.roles"
                :key="role.id"
                type="primary"
                class="tag"
              >
                {{ role.name }}
              </el-tag>
            </template>
          </el-table-column>
          <el-table-column
            :show-overflow-tooltip="true"
            prop="dept"
            label="部门"
          >
            <template slot-scope="scope">
              <div>{{ scope.row.dept.name }}</div>
            </template>
          </el-table-column>
          <el-table-column
            :show-overflow-tooltip="true"
            width="200"
            label="岗位"
          >
            <template slot-scope="scope">
              <el-tag
                v-for="job in scope.row.jobs"
                :key="job.id"
                type="primary"
                class="tag"
              >
                {{ job.name }}
              </el-tag>
            </template>
          </el-table-column>
          <el-table-column
            :show-overflow-tooltip="true"
            label="租户"
          >
            <template slot-scope="scope">
              <div v-if="scope.row.tenant">{{ scope.row.tenant.name }}</div>
            </template>
          </el-table-column>
          <el-table-column label="状态" align="center" prop="enabled">
            <template slot-scope="scope">
              <el-switch
                v-model="scope.row.enabled"
                :disabled="user.id === scope.row.id"
                active-color="#409EFF"
                inactive-color="#F56C6C"
                @change="changeEnabled(scope.row, scope.row.enabled)"
              />
            </template>
          </el-table-column>
          <el-table-column
            :show-overflow-tooltip="true"
            prop="createTime"
            width="135"
            label="创建日期"
          />
          <el-table-column
            v-if="checkPer(['user_edit', 'user_del'])"
            label="操作"
            width="115"
            align="center"
            fixed="right"
          >
            <template slot-scope="scope">
              <udOperation
                :data="scope.row"
                :permission="permission"
                :disabled-dle="scope.row.id === user.id"
              />
            </template>
          </el-table-column>
        </el-table>
        <!--分页组件-->
        <pagination />
      </el-col>
    </el-row>
  </div>
</template>

<script>
import crudUser from '@/api/permission/user'
import { isvalidPhone } from '@/utils/validate'
import { getDepts, getDeptSuperior } from '@/api/permission/dept'
import { getAllRole, getLevel } from '@/api/permission/role'
import { getAllJob } from '@/api/permission/job'
import { getAllTenant } from '@/api/system/tenant'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'
import DateRangePicker from '@/components/DateRangePicker'
import Treeselect from '@riophae/vue-treeselect'
import { mapGetters } from 'vuex'
import '@riophae/vue-treeselect/dist/vue-treeselect.css'
import { LOAD_CHILDREN_OPTIONS } from '@riophae/vue-treeselect'
import Avatar from '@/assets/images/avatar.png'

let userRoles = []
let userJobs = []
const defaultForm = {
  id: null,
  username: null,
  nickName: null,
  gender: '男',
  email: null,
  enabled: true,
  roles: [],
  jobs: [],
  dept: { id: null },
  phone: '',
  tenantId: null
}
export default {
  name: 'User',
  components: {
    Treeselect,
    crudOperation,
    rrOperation,
    udOperation,
    pagination,
    DateRangePicker
  },
  cruds() {
    return CRUD({
      title: '用户',
      url: 'api/user/query',
      sortFields: ['id desc'],
      crudMethod: { ...crudUser }
    })
  },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  // 数据字典
  dicts: ['user_status'],
  data() {
    // 自定义验证
    const validPhone = (rule, value, callback) => {
      if (!value) {
        callback(new Error('请输入电话号码'))
      } else if (!isvalidPhone(value)) {
        callback(new Error('请输入正确的11位手机号码'))
      } else {
        callback()
      }
    }
    return {
      height: document.documentElement.clientHeight - 180 + 'px;',
      deptName: '',
      depts: [],
      deptDatas: [],
      jobs: [],
      level: 3,
      roles: [],
      tenants: [],
      jobDatas: [],
      roleDatas: [],
      defaultProps: { children: 'children', label: 'name', isLeaf: 'leaf' },
      permission: {
        add: ['user_add'],
        edit: ['user_edit'],
        del: ['user_del'],
        down: ['user_down']
      },
      rules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          {
            min: 2,
            max: 20,
            message: '长度在 2 到 20 个字符',
            trigger: 'blur'
          }
        ],
        nickName: [
          { required: true, message: '请输入用户昵称', trigger: 'blur' },
          {
            min: 2,
            max: 20,
            message: '长度在 2 到 20 个字符',
            trigger: 'blur'
          }
        ],
        email: [
          { required: true, message: '请输入邮箱地址', trigger: 'blur' },
          { type: 'email', message: '请输入正确的邮箱地址', trigger: 'blur' }
        ],
        phone: [{ required: true, trigger: 'blur', validator: validPhone }]
      },
      Avatar: Avatar
    }
  },
  computed: {
    ...mapGetters(['user', 'baseApi'])
  },
  created() {
    this.crud.msg.add = '新增成功，默认密码：123456'
  },
  mounted: function() {
    const that = this
    window.onresize = function temp() {
      that.height = document.documentElement.clientHeight - 180 + 'px;'
    }
  },
  methods: {
    changeRole(value) {
      userRoles = []
      value.forEach(function(data, index) {
        const role = { id: data.id, name: data.name, permission: data.permission }
        userRoles.push(role)
      })
    },
    changeJob(value) {
      userJobs = []
      value.forEach(function(data, index) {
        const job = { id: data.id, name: data.name }
        userJobs.push(job)
      })
    },
    deleteTag(value) {
      userRoles.forEach(function(data, index) {
        if (data.id === value) {
          userRoles.splice(index, value)
        }
      })
    },
    // 新增与编辑前做的操作
    [CRUD.HOOK.afterToCU](crud, form) {
      this.getRoles()
      if (form.id == null) {
        this.getDepts()
      } else {
        this.getSupDepts(form.dept.id)
      }
      this.getRoleLevel()
      this.getJobs()
      this.getTenants()
      // form.enabled = form.enabled
    },
    // 新增前将多选的值设置为空
    [CRUD.HOOK.beforeToAdd]() {
      this.jobDatas = []
      this.roleDatas = []
    },
    // 初始化编辑时候的角色与岗位
    [CRUD.HOOK.beforeToEdit](crud, form) {
      this.jobDatas = []
      this.roleDatas = []
      userRoles = []
      userJobs = []
      const _this = this
      form.roles.forEach(function(role, index) {
        _this.roleDatas.push({ id: role.id, name: role.name })
        const rol = { id: role.id, name: role.name }
        userRoles.push(rol)
      })
      form.jobs.forEach(function(job, index) {
        _this.jobDatas.push({ id: job.id, name: job.name })
        const data = { id: job.id, name: job.name }
        userJobs.push(data)
      })
      if (form.tenantId === 0) {
        this.form.tenantId = null
      }
    },
    // 提交前做的操作
    [CRUD.HOOK.afterValidateCU](crud) {
      if (!crud.form.dept.id) {
        this.$message({
          message: '部门不能为空',
          type: 'warning',
          center: true
        })
        return false
      } else if (this.jobDatas.length === 0) {
        this.$message({
          message: '岗位不能为空',
          type: 'warning',
          center: true
        })
        return false
      } else if (this.roleDatas.length === 0) {
        this.$message({
          message: '角色不能为空',
          type: 'warning',
          center: true
        })
        return false
      }
      crud.form.roles = userRoles
      crud.form.jobs = userJobs
      crud.form.phone = crud.form.phone.toString()
      return true
    },
    // 获取左侧部门数据
    getDeptDatas(node, resolve) {
      const sort = 'sort asc'
      const params = { SortFields: sort }
      if (typeof node !== 'object') {
        if (node) {
          params['name'] = node
        }
      } else if (node.level !== 0) {
        params['parentId'] = node.data.id
      }
      setTimeout(() => {
        getDepts(params).then((res) => {
          if (resolve) {
            resolve(res.content)
          } else {
            this.deptDatas = res.content
          }
        })
      }, 100)
    },
    getDepts() {
      getDepts({ enabled: true }).then((res) => {
        this.depts = res.content.map(function(obj) {
          if (obj.hasChildren) {
            obj.children = null
          }
          return obj
        })
      })
    },
    getSupDepts(deptId) {
      getDeptSuperior(deptId).then((res) => {
        const date = res.content
        this.buildDepts(date)
        this.depts = date
      })
    },
    buildDepts(depts) {
      depts.forEach((data) => {
        if (data.children) {
          this.buildDepts(data.children)
        }
        if (data.hasChildren && !data.children) {
          data.children = null
        }
      })
    },
    // 获取弹窗内部门数据
    loadDepts({ action, parentNode, callback }) {
      if (action === LOAD_CHILDREN_OPTIONS) {
        getDepts({ enabled: true, parentId: parentNode.id }).then((res) => {
          parentNode.children = res.content.map(function(obj) {
            if (obj.hasChildren) {
              obj.children = null
            }
            return obj
          })
          setTimeout(() => {
            callback()
          }, 200)
        })
      }
    },
    // 切换部门
    handleNodeClick(data) {
      if (data.parentId === 0) {
        this.query.deptId = null
      } else {
        this.query.deptId = data.id
      }
      this.crud.toQuery()
    },
    // 改变状态
    changeEnabled(data, val) {
      this.$confirm(
        '此操作将 "' +
        this.dict.label.user_status[val] +
        '" ' +
        data.username +
        ', 是否继续？',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      )
        .then(() => {
          crudUser
            .edit(data)
            .then((res) => {
              this.crud.message(
                this.dict.label.user_status[val] + '成功',
                CRUD.NOTIFICATION_TYPE.SUCCESS
              )
            })
            .catch(() => {
              data.enabled = !data.enabled
            })
        })
        .catch(() => {
          data.enabled = !data.enabled
        })
    },
    // 获取弹窗内角色数据
    getRoles() {
      getAllRole()
        .then((res) => {
          this.roles = res
        })
        .catch(() => {
        })
    },
    // 获取弹窗内岗位数据
    getJobs() {
      getAllJob()
        .then((res) => {
          this.jobs = res.content
        })
        .catch(() => {
        })
    },
    // 获取权限级别
    getRoleLevel() {
      getLevel()
        .then((res) => {
          this.level = res.level
        })
        .catch(() => {
        })
    },
    getTenants() {
      getAllTenant()
        .then((res) => {
          this.tenants = res.content
        })
        .catch(() => {
        })
    },
    checkboxT(row, rowIndex) {
      return row.id !== this.user.id
    }
  }
}
</script>

<style rel='stylesheet/scss' lang='scss' scoped>
::v-deep .vue-treeselect__control,
::v-deep .vue-treeselect__placeholder,
::v-deep .vue-treeselect__single-value {
  height: 30px;
  line-height: 30px;
}

.tag {
  margin-right: 5px;
}

.avatar {
  width: 50px;
  height: 50px;
  border-radius: 50%;
}
</style>
