<template>
  <div class="app-container">
    <div class="filter-container">
      <el-select
        v-model="listQuery.tenantId"
        placeholder="租户（必填）"
        style="width: 220px"
        filterable
        class="filter-item"
        @change="tenantSelectList()"
      >
        <el-option
          v-for="item in tenantOptions"
          :key="item.key"
          :label="item.display_name"
          :value="item.key"
        />
      </el-select>
      <el-select
        v-model="listQuery.itemId"
        placeholder="项目（必填）"
        style="width: 220px"
        filterable
        class="filter-item"
        @change="itemSelectList()"
      >
        <el-option
          v-for="item in itemOptions"
          :key="item.key"
          :label="item.display_name"
          :value="item.key"
        />
      </el-select>
      <el-button
        v-waves
        class="filter-item"
        type="primary"
        icon="el-icon-search"
        @click="fetchData"
      >
        搜索
      </el-button>
    </div>
    <el-table
      v-loading="listLoading"
      :data="list"
      element-loading-text="Loading"
      border
      fit
      max-height="714"
      highlight-current-row
    >
      <el-table-column label="序号" fixed width="95">
        <template slot-scope="scope">{{ scope.$index + 1 }}</template>
      </el-table-column>
      <el-table-column label="实例标识" width="260">
        <template slot-scope="scope">
          <el-link type="primary" :underline="false">{{ scope.row.identify }}</el-link>
        </template>
      </el-table-column>
      <el-table-column label="Active" width="120">
        <template slot-scope="scope">
          <el-tag :type="scope.row.active | statusFilter">
            {{ scope.row.active }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column label="核心线程" width="120">
        <template slot-scope="scope">
          <el-link type="success" :underline="false">{{ scope.row.coreSize }}</el-link>
        </template>
      </el-table-column>
      <el-table-column label="最大线程" width="120">
        <template slot-scope="scope">
          <el-link type="danger" :underline="false">{{ scope.row.maximumSize }}</el-link>
        </template>
      </el-table-column>
      <el-table-column label="队列类型" width="260">
        <template slot-scope="scope">{{ scope.row.queueType }}</template>
      </el-table-column>
      <el-table-column label="队列容量" width="160">
        <template slot-scope="scope">{{ scope.row.queueCapacity }}</template>
      </el-table-column>
      <el-table-column label="拒绝策略" width="260">
        <template slot-scope="scope">{{ scope.row.rejectedName }}</template>
      </el-table-column>
      <el-table-column label="空闲回收" width="100">
        <template slot-scope="scope">{{ scope.row.keepAliveTime }}</template>
      </el-table-column>
      <el-table-column
        label="操作"
        width="90"
        align="center"
        fixed="right"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="{ row }">
          <el-button type="text" size="small" @click="handleInfo(row)"> 查看 </el-button>
          <el-button type="text" :disabled="isEditDisabled" size="small" @click="handleUpdate(row)">
            编辑
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-dialog
      :title="textMap[dialogStatus]"
      :visible.sync="instanceDialogFormVisible"
      width="1000px"
    >
      <template>
        <el-descriptions class="margin-top" title="负载信息" :column="3" :size="size" border>
          <el-descriptions-item>
            <template slot="label"> 当前负载 </template>
            {{ runTimeTemp.currentLoad }}
          </el-descriptions-item>
          <el-descriptions-item>
            <template slot="label"> 峰值负载 </template>
            {{ runTimeTemp.peakLoad }}
          </el-descriptions-item>
          <el-descriptions-item>
            <template slot="label"> 剩余内存 </template>
            {{ runTimeTemp.freeMemory }}
          </el-descriptions-item>
          <el-descriptions-item>
            <template slot="label"> 内存占比 </template>
            {{ runTimeTemp.memoryProportion }}
          </el-descriptions-item>
        </el-descriptions>
        <br />
        <br />
        <el-descriptions class="margin-top" title="线程信息" :column="3" :size="size" border>
          <el-descriptions-item>
            <template slot="label"> 核心线程 </template>
            {{ runTimeTemp.coreSize }}
          </el-descriptions-item>
          <el-descriptions-item>
            <template slot="label"> 当前线程 </template>
            {{ runTimeTemp.poolSize }}
          </el-descriptions-item>
          <el-descriptions-item>
            <template slot="label"> 最大线程 </template>
            {{ runTimeTemp.maximumSize }}
          </el-descriptions-item>
          <el-descriptions-item>
            <template slot="label"> 活跃线程 </template>
            {{ runTimeTemp.activeSize }}
          </el-descriptions-item>
          <el-descriptions-item>
            <template slot="label"> 同存最大线程 </template>
            {{ runTimeTemp.largestPoolSize }}
          </el-descriptions-item>
        </el-descriptions>
        <br />
        <br />
        <el-descriptions class="margin-top" title="队列信息" :column="3" :size="size" border>
          <el-descriptions-item>
            <template slot="label"> 队列容量 </template>
            {{ runTimeTemp.queueCapacity }}
          </el-descriptions-item>
          <el-descriptions-item>
            <template slot="label"> 队列元素 </template>
            {{ runTimeTemp.queueSize }}
          </el-descriptions-item>
          <el-descriptions-item>
            <template slot="label"> 队列剩余容量 </template>
            {{ runTimeTemp.queueRemainingCapacity }}
          </el-descriptions-item>
          <el-descriptions-item>
            <template slot="label"> 阻塞队列 </template>
            {{ runTimeTemp.queueType }}
          </el-descriptions-item>
        </el-descriptions>
        <br />
        <br />
        <el-descriptions class="margin-top" title="其它信息" :column="3" :size="size" border>
          <el-descriptions-item>
            <template slot="label"> 任务总量 </template>
            {{ runTimeTemp.completedTaskCount }}
          </el-descriptions-item>
          <el-descriptions-item>
            <template slot="label"> 最后更新时间 </template>
            {{ runTimeTemp.clientLastRefreshTime }}
          </el-descriptions-item>
          <el-descriptions-item>
            <template slot="label"> 拒绝策略 </template>
            {{ runTimeTemp.rejectedName }}
          </el-descriptions-item>
        </el-descriptions>
      </template>
      <div slot="footer" class="dialog-footer">
        <el-button @click="instanceDialogFormVisible = false">
          <i class="el-icon-close" />
          关 闭
        </el-button>
        <el-button type="primary" @click="handleInfo()">
          <i class="el-icon-refresh-right" />
          刷 新
        </el-button>
      </div>
    </el-dialog>
    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
      <el-form
        ref="dataForm"
        :rules="rules"
        :model="temp"
        label-position="left"
        label-width="110px"
      >
        <el-form-item label="核心线程" prop="coreSize">
          <template>
            <el-input-number
              v-model="temp.coreSize"
              controls-position="right"
              :min="1"
              :max="9999"
            ></el-input-number>
          </template>
        </el-form-item>
        <el-form-item label="最大线程" prop="maximumSize">
          <template>
            <el-input-number
              v-model="temp.maximumSize"
              controls-position="right"
              :min="1"
              :max="9999"
            ></el-input-number>
          </template>
        </el-form-item>
        <el-form-item label="全部修改" prop="allUpdate">
          <el-switch v-model="temp.allUpdate"></el-switch>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false"> 取消 </el-button>
        <el-button type="primary" @click="updateData()"> 确认 </el-button>
      </div>
    </el-dialog>
    <el-dialog :visible.sync="dialogPluginVisible" title="Reading statistics">
      <el-table :data="pluginData" border fit highlight-current-row style="width: 100%">
        <el-table-column prop="key" label="Channel" />
        <el-table-column prop="pv" label="Pv" />
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="dialogPvVisible = false">Confirm</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import * as itemApi from '@/api/hippo4j-item';
import * as tenantApi from '@/api/hippo4j-tenant';
import * as threadPoolApi from '@/api/hippo4j-threadPool';
import waves from '@/directive/waves';

export default {
  name: 'JobProject',
  directives: { waves },
  filters: {
    statusFilter(status) {
      const statusMap = {
        DEV: 'info',
        TEST: 'success',
        UAT: 'warning',
        PROD: 'danger',
      };
      return statusMap[status];
    },
  },
  data() {
    return {
      isRejectShow: false, // 是否显示spi拒绝策略
      list: [],
      listLoading: false,
      total: 0,
      listQuery: {
        current: 1,
        size: 10,
        tpId: '',
        itemId: '',
      },
      pluginTypeOptions: ['reader', 'writer'],
      dialogPluginVisible: false,
      pluginData: [],
      dialogFormVisible: false,
      isEditDisabled: false,
      runTimeTemp: {},
      tenantOptions: [],
      instanceDialogFormVisible: false,
      threadPoolOptions: [],
      itemOptions: [],
      itemTempOptions: [],
      queueTypeOptions: [
        { key: 1, display_name: 'ArrayBlockingQueue' },
        { key: 2, display_name: 'LinkedBlockingQueue' },
        { key: 3, display_name: 'LinkedBlockingDeque' },
        { key: 4, display_name: 'SynchronousQueue' },
        { key: 5, display_name: 'LinkedTransferQueue' },
        { key: 6, display_name: 'PriorityBlockingQueue' },
        { key: 9, display_name: 'ResizableLinkedBlockingQueue (动态修改队列大小)' },
      ],
      rejectedOptions: [
        { key: 1, display_name: 'CallerRunsPolicy' },
        { key: 2, display_name: 'AbortPolicy' },
        { key: 3, display_name: 'DiscardPolicy' },
        { key: 4, display_name: 'DiscardOldestPolicy' },
        { key: 5, display_name: 'RunsOldestTaskPolicy' },
        { key: 6, display_name: 'SyncPutQueuePolicy' },
        { key: 99, display_name: 'CustomRejectedPolicy（自定义 SPI 策略）' },
      ],
      alarmTypes: [
        { key: 1, display_name: '报警' },
        { key: 0, display_name: '不报警' },
      ],
      allowCoreThreadTimeOutTypes: [
        { key: 1, display_name: '超时' },
        { key: 0, display_name: '不超时' },
      ],
      size: 500,
      dialogStatus: '',
      textMap: {
        update: 'Edit',
        create: 'Create',
        info: 'Info',
      },
      rules: {
        coreSize: [{ required: true, message: 'this is required', trigger: 'blur' }],
        maximumSize: [{ required: true, message: 'this is required', trigger: 'blur' }],
      },
      temp: {
        id: undefined,
        tenantId: '',
        itemId: '',
        rejectedType: null,
        allUpdate: '1',
        customRejectedType: null,
      },
      visible: true,
    };
  },
  created() {
    // 初始化租户、项目
    this.initSelect();
  },
  mounted() {
    this.isEditDisabled = this.$cookie.get('userName') !== 'admin';
  },
  methods: {
    onInput() {
      this.$forceUpdate();
    },
    fetchData() {
      if (!this.listQuery.tenantId) {
        this.$message.warning('租户不允许为空');
        return;
      }
      if (!this.listQuery.itemId) {
        this.$message.warning('项目不允许为空');
        return;
      }
      this.listLoading = true;
      threadPoolApi.listClient(this.listQuery).then((response) => {
        if (response == null) {
          this.listLoading = false;
          return;
        }
        const tempResp = response;
        const tempList = [];
        for (let i = 0; i < tempResp.length; i++) {
          const tempData = {};
          const tempResp0 = response[i];
          tempData.identify = tempResp0.identify;
          tempData.active = tempResp0.active;
          tempData.clientAddress = tempResp0.clientAddress;
          tempData.coreSize = tempResp0.coreSize;
          tempData.maximumSize = tempResp0.maximumSize;
          tempData.queueType = tempResp0.queueType;
          tempData.queueCapacity = tempResp0.queueCapacity;
          tempData.rejectedName = tempResp0.rejectedName;
          tempData.keepAliveTime = tempResp0.keepAliveTime;
          tempList.push(tempData);
        }
        this.list = tempList;
        this.listLoading = false;
      });
    },
    changeAlarm(row) {
      threadPoolApi.alarmEnable(row).then(() => {
        this.fetchData();
        this.$notify({
          title: 'Success',
          message: 'Update Successfully',
          type: 'success',
          duration: 2000,
        });
      });
    },
    initSelect() {
      tenantApi.list({ size: this.size }).then((response) => {
        const { records } = response;
        for (let i = 0; i < records.length; i++) {
          this.tenantOptions.push({
            key: records[i].tenantId,
            display_name: records[i].tenantId + ' ' + records[i].tenantName,
          });
        }
      });
    },
    resetTemp() {
      this.isRejectShow = false;
      this.temp = {
        id: undefined,
        tenantId: '',
        itemId: '',
        rejectedType: null,
        customRejectedType: null,
      };
    },
    handleCreate() {
      this.resetTemp();
      this.dialogStatus = 'create';
      this.dialogFormVisible = true;
      this.isRejectShow = false;
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate();
      });
    },
    createData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          if (this.isRejectShow) {
            if (this.temp.customRejectedType == null) {
              this.temp.rejectedType = 2;
            } else {
              this.temp.rejectedType = this.temp.customRejectedType;
            }
          }
          threadPoolApi.created(this.temp).then(() => {
            this.fetchData();
            this.dialogFormVisible = false;
            this.$notify({
              title: 'Success',
              message: 'Created Successfully',
              type: 'success',
              duration: 2000,
            });
          });
        }
      });
    },
    handleUpdate(row) {
      this.temp = Object.assign({}, row); // copy obj
      let rejectedType = this.temp.rejectedType;
      const typeList = [1, 2, 3, 4, 5, 6];
      if (!typeList.includes(rejectedType)) {
        this.isRejectShow = true;
        this.temp.customRejectedType = this.temp.rejectedType;
        this.temp.rejectedType = 99;
      } else {
        this.isRejectShow = false;
      }
      this.dialogStatus = 'update';
      this.dialogFormVisible = true;

      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate();
      });
    },
    updateData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          if (parseInt(this.temp.maximumSize) < parseInt(this.temp.coreSize)) {
            this.$message({
              message: '最大线程必须大于等于核心线程',
              type: 'warning',
            });
            return;
          }
          const clientAddressList = [];
          const tempData = {
            corePoolSize: this.temp.coreSize,
            maximumPoolSize: this.temp.maximumSize,
            keepAliveTime: this.temp.keepAliveTime,
            clientAddressList: clientAddressList,
          };
          if (!this.temp.allUpdate) {
            clientAddressList[0] = this.temp.clientAddress;
          } else {
            for (let i = 0; i < this.list.length; i++) {
              if (this.list[i] != null) {
                clientAddressList[i] = this.list[i].clientAddress;
              }
            }
          }
          this.updateExecute(tempData);
        }
      });
    },

    updateExecute(updateData) {
      threadPoolApi
        .webUpdatePool(updateData)
        .then((response) => {
          this.dialogFormVisible = false;
          this.$notify({
            title: 'Success',
            message: 'Update Successfully',
            type: 'success',
            duration: 2000,
          });
          this.fetchData();
        })
        .catch((error) => {
          console.log(error);
          this.$message.error('查询失败，请尝试刷新页面');
        });
    },

    openDelConfirm(name) {
      return this.$confirm(`此操作将删除 ${name}, 是否继续?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      });
    },
    selectQueueType(value) {
      if (value === 4) {
        this.temp.capacity = 0;
      } else if (value === 5) {
        this.temp.capacity = 2147483647;
      }
    },

    tenantSelectList() {
      this.listQuery.itemId = null;
      this.listQuery.tpId = null;

      this.temp.itemId = null;

      this.itemOptions = [];
      this.itemTempOptions = [];
      this.threadPoolOptions = [];
      const tenantId = { tenantId: this.listQuery.tenantId, size: this.size };
      itemApi.list(tenantId).then((response) => {
        const { records } = response;
        for (let i = 0; i < records.length; i++) {
          this.itemOptions.push({
            key: records[i].itemId,
            display_name: records[i].itemId + ' ' + records[i].itemName,
          });
        }
      });
    },

    tenantTempSelectList() {
      this.itemTempOptions = [];
      if (this.temp.itemId) {
        this.temp.itemId = null;
      }
      const tenantId = { tenantId: this.temp.tenantId, size: this.size };
      itemApi.list(tenantId).then((response) => {
        const { records } = response;
        for (let i = 0; i < records && records.length; i++) {
          this.itemTempOptions.push({
            key: records[i].itemId,
            display_name: records[i].itemId + ' ' + records[i].itemName,
          });
        }
      });
    },

    itemSelectList() {
      this.listQuery.tpId = null;

      this.threadPoolOptions = [];
      const itemId = { itemId: this.listQuery.itemId, size: this.size };
      threadPoolApi.list(itemId).then((response) => {
        const { records } = response;
        for (let i = 0; i < records && records.length; i++) {
          this.threadPoolOptions.push({
            key: records[i].tpId,
            display_name: records[i].tpId,
          });
        }
      });
    },

    selectRejectedType(value) {
      if (value == 99) {
        this.isRejectShow = true;
      } else {
        this.isRejectShow = false;
      }
    },

    handleInfo(row) {
      this.instanceDialogFormVisible = true;
      this.dialogStatus = 'info';

      if (typeof row == 'undefined' || row == null) {
        row = this.tempRow;
      } else {
        this.tempRow = {
          clientAddress: row.clientAddress,
        };
      }

      this.refresh(row);
    },

    refresh(row) {
      let clientAddressStr = '';
      const clientAddress = row.clientAddress;
      let clientBasePath = row.clientBasePath;
      if (clientBasePath != null) {
        clientAddressStr = clientAddress + clientBasePath;
      } else {
        clientAddressStr = clientAddress;
      }
      threadPoolApi
        .webBaseState({ clientAddress: clientAddressStr })
        .then((response) => {
          if (response != null) {
            this.runTimeTemp = response;
          }
        })
        .catch((error) => {
          console.log(error);
          this.$message.error('查询失败，请尝试刷新页面');
        });
    },
  },
};
</script>
