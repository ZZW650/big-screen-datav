<template>
  <div class="disaster-container">
    <!-- 顶部导航栏 -->
    <header class="navbar">
      <div class="logo">
        <i class="icon-water"></i>
      </div>
      <div class="nav-actions">
        <button class="btn btn-outline" @click="refreshData">
          <i class="icon-refresh"></i> 刷新
        </button>
        <div class="time-info">
          <span>数据更新时间: </span>
          <span>{{ updateTime }}</span>
        </div>
      </div>
    </header>

    <!-- 主内容区 -->
    <main class="main-content">
      <!-- 左侧筛选面板 -->
      <aside class="filter-panel">
        <div class="panel-title">
          <h2>筛选条件</h2>
        </div>

        <div class="filter-group">
          <label class="filter-label">承载体类型</label>
          <select v-model="selectedType" class="filter-select" @change="filterData">
            <option value="all">全部类型</option>
            <option value="building">建筑物</option>
            <option value="road">道路设施</option>
            <option value="water">水利设施</option>
            <option value="power">电力设施</option>
            <option value="communication">通信设施</option>
          </select>
        </div>

        <div class="filter-group">
          <label class="filter-label">受灾程度</label>
          <div class="checkbox-group">
            <label class="checkbox-item">
              <input type="checkbox" value="light" v-model="damageLevels"> 轻微
            </label>
            <label class="checkbox-item">
              <input type="checkbox" value="moderate" v-model="damageLevels"> 中度
            </label>
            <label class="checkbox-item">
              <input type="checkbox" value="severe" v-model="damageLevels"> 严重
            </label>
            <label class="checkbox-item">
              <input type="checkbox" value="destroyed" v-model="damageLevels"> 损毁
            </label>
          </div>
        </div>

        <div class="filter-group">
          <label class="filter-label">区域筛选</label>
          <select v-model="selectedDistrict" class="filter-select" @change="filterData">
            <option value="all">全部区域</option>
            <option value="beilin">碑林区</option>
            <option value="xincheng">新城区</option>
            <option value="lianhu">莲湖区</option>
            <option value="baqiao">灞桥区</option>
            <option value="雁塔">雁塔区</option>
            <option value="weiyang">未央区</option>
            <option value="chanba">浐灞生态区</option>
          </select>
        </div>

        <div class="filter-actions">
          <button class="btn btn-primary" @click="filterData">应用筛选</button>
          <button class="btn btn-outline" @click="resetFilters">重置</button>
        </div>

        <div class="stats-card">
          <h3>统计概览</h3>
          <div class="stat-item">
            <span class="stat-label">总计承载体</span>
            <span class="stat-value">{{ totalCount }}</span>
          </div>
          <div class="stat-item">
            <span class="stat-label">受影响</span>
            <span class="stat-value affected">{{ affectedCount }}</span>
          </div>
          <div class="stat-item">
            <span class="stat-label">严重受损</span>
            <span class="stat-value severe">{{ severeCount }}</span>
          </div>
          <div class="stat-chart">
            <canvas id="damageChart"></canvas>
          </div>
        </div>
      </aside>

      <!-- 右侧内容区 -->
      <section class="content-panel">
        <div class="panel-header">
          <h2>承载体信息列表</h2>
          <div class="search-bar">
            <input
                type="text"
                placeholder="搜索承载体名称或编号..."
                v-model="searchQuery"
                @input="handleSearch"
            >
            <button class="btn btn-primary" @click="openAddForm">
              <i class="icon-plus"></i> 新增信息
            </button>
          </div>
        </div>

        <!-- 数据表格 -->
        <div class="data-table-container">
          <table class="data-table">
            <thead>
            <tr>
              <th>编号</th>
              <th>名称</th>
              <th>类型</th>
              <th>所在区域</th>
              <th>受灾程度</th>
              <th>受损情况描述</th>
              <th>上报时间</th>
              <th>操作</th>
            </tr>
            </thead>
            <tbody>
            <tr v-for="item in filteredData" :key="item.id" class="table-row">
              <td>{{ item.id }}</td>
              <td>{{ item.name }}</td>
              <td>
                <span :class="`type-tag ${item.type}`">{{ getTypeName(item.type) }}</span>
              </td>
              <td>{{ getDistrictName(item.district) }}</td>
              <td>
                  <span :class="`damage-tag ${item.damageLevel}`">
                    {{ getDamageLevelName(item.damageLevel) }}
                  </span>
              </td>
              <td class="description-cell">{{ item.description }}</td>
              <td>{{ formatTime(item.reportTime) }}</td>
              <td class="action-buttons">
                <button class="btn btn-icon" @click="viewDetails(item)" title="查看详情">
                  <i class="icon-eye"></i>
                  查看
                </button>
                <button class="btn btn-icon" @click="editItem(item)" title="编辑">
                  <i class="icon-edit"></i>
                  编辑
                </button>
                <button class="btn btn-icon danger" @click="deleteItem(item.id)" title="删除">
                  <i class="icon-trash"></i>
                  删除
                </button>
              </td>
            </tr>
            <tr v-if="filteredData.length === 0" class="no-data">
              <td colspan="8">暂无符合条件的数据</td>
            </tr>
            </tbody>
          </table>
        </div>

        <!-- 分页 -->
        <div class="pagination" v-if="filteredData.length > 0">
          <button class="btn btn-outline" @click="changePage(currentPage - 1)" :disabled="currentPage === 1">上一页</button>
          <span class="page-info">第 {{ currentPage }} / {{ totalPages }} 页</span>
          <button class="btn btn-outline" @click="changePage(currentPage + 1)" :disabled="currentPage === totalPages">下一页</button>
          <div class="page-size">
            <span>每页显示:</span>
            <select v-model="pageSize" @change="resetPage">
              <option value="10">10条</option>
              <option value="20">20条</option>
              <option value="50">50条</option>
            </select>
          </div>
        </div>
      </section>
    </main>

    <!-- 详情弹窗 -->
    <div class="modal" v-if="showDetails">
      <div class="modal-overlay" @click="closeDetails"></div>
      <div class="modal-content">
        <div class="modal-header">
          <h3>承载体详情</h3>
          <button class="modal-close" @click="closeDetails">×</button>
        </div>
        <div class="modal-body" v-if="currentItem">
          <div class="detail-row">
            <span class="detail-label">编号:</span>
            <span class="detail-value">{{ currentItem.id }}</span>
          </div>
          <div class="detail-row">
            <span class="detail-label">名称:</span>
            <span class="detail-value">{{ currentItem.name }}</span>
          </div>
          <div class="detail-row">
            <span class="detail-label">类型:</span>
            <span class="detail-value">{{ getTypeName(currentItem.type) }}</span>
          </div>
          <div class="detail-row">
            <span class="detail-label">所在区域:</span>
            <span class="detail-value">{{ getDistrictName(currentItem.district) }}</span>
          </div>
          <div class="detail-row">
            <span class="detail-label">受灾程度:</span>
            <span class="detail-value">
              <span :class="`damage-tag ${currentItem.damageLevel}`">
                {{ getDamageLevelName(currentItem.damageLevel) }}
              </span>
            </span>
          </div>
          <div class="detail-row">
            <span class="detail-label">详细地址:</span>
            <span class="detail-value">{{ currentItem.address }}</span>
          </div>
          <div class="detail-row">
            <span class="detail-label">受损情况:</span>
            <span class="detail-value">{{ currentItem.description }}</span>
          </div>
          <div class="detail-row">
            <span class="detail-label">上报人:</span>
            <span class="detail-value">{{ currentItem.reporter }}</span>
          </div>
          <div class="detail-row">
            <span class="detail-label">上报时间:</span>
            <span class="detail-value">{{ formatTime(currentItem.reportTime) }}</span>
          </div>
          <div class="detail-row images-row">
            <span class="detail-label">现场图片:</span>
            <div class="images-container">
              <img v-for="(img, idx) in currentItem.images" :key="idx" :src="img" class="damage-image" alt="受灾现场图片">
            </div>
          </div>
        </div>
        <div class="modal-footer">
          <button class="btn btn-outline" @click="closeDetails">关闭</button>
        </div>
      </div>
    </div>

    <!-- 新增/编辑表单弹窗 -->
    <div class="modal" v-if="showForm">
      <div class="modal-overlay" @click="closeForm"></div>
      <div class="modal-content form-modal">
        <div class="modal-header">
          <h3>{{ isEditing ? '编辑承载体信息' : '新增承载体信息' }}</h3>
          <button class="modal-close" @click="closeForm">×</button>
        </div>
        <div class="modal-body">
          <form class="data-form" @submit.prevent="saveItem">
            <div class="form-group">
              <label class="form-label">名称 <span class="required">*</span></label>
              <input type="text" v-model="formData.name" class="form-control" required>
            </div>
            <div class="form-group">
              <label class="form-label">类型 <span class="required">*</span></label>
              <select v-model="formData.type" class="form-control" required>
                <option value="">请选择类型</option>
                <option value="building">建筑物</option>
                <option value="road">道路设施</option>
                <option value="water">水利设施</option>
                <option value="power">电力设施</option>
                <option value="communication">通信设施</option>
              </select>
            </div>
            <div class="form-group">
              <label class="form-label">所在区域 <span class="required">*</span></label>
              <select v-model="formData.district" class="form-control" required>
                <option value="">请选择区域</option>
                <option value="beilin">碑林区</option>
                <option value="xincheng">新城区</option>
                <option value="lianhu">莲湖区</option>
                <option value="baqiao">灞桥区</option>
                <option value="雁塔">雁塔区</option>
                <option value="weiyang">未央区</option>
                <option value="chanba">浐灞生态区</option>
              </select>
            </div>
            <div class="form-group">
              <label class="form-label">详细地址 <span class="required">*</span></label>
              <input type="text" v-model="formData.address" class="form-control" required>
            </div>
            <div class="form-group">
              <label class="form-label">受灾程度 <span class="required">*</span></label>
              <select v-model="formData.damageLevel" class="form-control" required>
                <option value="">请选择受灾程度</option>
                <option value="light">轻微</option>
                <option value="moderate">中度</option>
                <option value="severe">严重</option>
                <option value="destroyed">损毁</option>
              </select>
            </div>
            <div class="form-group">
              <label class="form-label">受损情况描述 <span class="required">*</span></label>
              <textarea v-model="formData.description" class="form-control" rows="4" required></textarea>
            </div>
            <div class="form-group">
              <label class="form-label">上报人 <span class="required">*</span></label>
              <input type="text" v-model="formData.reporter" class="form-control" required>
            </div>
            <div class="form-group">
              <label class="form-label">现场图片</label>
              <input type="file" multiple accept="image/*" class="form-control" @change="handleImageUpload">
            </div>
          </form>
        </div>
        <div class="modal-footer">
          <button class="btn btn-outline" @click="closeForm">取消</button>
          <button class="btn btn-primary" @click="saveItem">保存</button>
        </div>
      </div>
    </div>

    <!-- 确认删除弹窗 -->
    <div class="modal" v-if="showDeleteConfirm">
      <div class="modal-overlay" @click="cancelDelete"></div>
      <div class="modal-content confirm-modal">
        <div class="modal-header">
          <h3>确认删除</h3>
          <button class="modal-close" @click="cancelDelete">×</button>
        </div>
        <div class="modal-body">
          <p>您确定要删除这条信息吗？此操作不可撤销。</p>
        </div>
        <div class="modal-footer">
          <button class="btn btn-outline" @click="cancelDelete">取消</button>
          <button class="btn btn-danger" @click="confirmDelete">确认删除</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, reactive, onMounted, computed } from 'vue';
import Chart from 'chart.js/auto';


export default {
  setup() {
    // 时间状态
    const updateTime = ref('2023-07-20 14:30:00');

    // 筛选条件
    const selectedType = ref('all');
    const damageLevels = ref(['light', 'moderate', 'severe', 'destroyed']);
    const selectedDistrict = ref('all');
    const searchQuery = ref('');

    // 分页状态
    const currentPage = ref(1);
    const pageSize = ref(10);

    // 弹窗状态
    const showDetails = ref(false);
    const showForm = ref(false);
    const showDeleteConfirm = ref(false);
    const isEditing = ref(false);
    const currentItem = ref(null);
    const itemToDelete = ref(null);

    // 表单数据
    const formData = reactive({
      name: '',
      type: '',
      district: '',
      address: '',
      damageLevel: '',
      description: '',
      reporter: '',
      images: []
    });

    // 模拟承载体数据
    const carrierData = ref([
      {
        id: 'XC-20230720-001',
        name: '西安站北广场',
        type: 'building',
        district: 'xincheng',
        address: '西安市新城区环城北路44号',
        damageLevel: 'severe',
        description: '广场积水约50cm，地下通道被淹，部分设施损坏',
        reporter: '张明',
        reportTime: 1689832200000,
      },
      {
        id: 'XC-20230720-002',
        name: '南二环路西段',
        type: 'road',
        district: '雁塔',
        address: '西安市雁塔区南二环路西段',
        damageLevel: 'moderate',
        description: '道路积水约30cm，交通拥堵，部分路段塌陷',
        reporter: '李华',
        reportTime: 1689832500000,
      },
      {
        id: 'XC-20230720-003',
        name: '浐河大桥',
        type: 'road',
        district: 'chanba',
        address: '西安市浐灞生态区浐河大桥',
        damageLevel: 'severe',
        description: '桥墩被洪水冲击，出现裂缝，已禁止通行',
        reporter: '王强',
        reportTime: 1689832800000,
      },
      {
        id: 'XC-20230720-004',
        name: '西安国际会展中心',
        type: 'building',
        district: 'chanba',
        address: '西安市浐灞生态区会展一路',
        damageLevel: 'light',
        description: '地下室轻微渗水，已及时处理',
        reporter: '赵丽',
        reportTime: 1689833100000,
        images: []
      },
      {
        id: 'XC-20230720-005',
        name: '西郊变电站',
        type: 'power',
        district: 'lianhu',
        address: '西安市莲湖区西郊工业区',
        damageLevel: 'destroyed',
        description: '变电站被淹没，设备损毁，导致周边大面积停电',
        reporter: '刘军',
        reportTime: 1689833400000,
      },
      {
        id: 'XC-20230720-006',
        name: '大雁塔地下停车场',
        type: 'building',
        district: '雁塔',
        address: '西安市雁塔区大雁塔北广场地下',
        damageLevel: 'severe',
        description: '停车场完全被淹没，约200辆车辆受影响',
        reporter: '陈明',
        reportTime: 1689833700000,
      },
      {
        id: 'XC-20230720-007',
        name: '北石桥污水处理厂',
        type: 'water',
        district: 'weiyang',
        address: '西安市未央区北石桥村',
        damageLevel: 'moderate',
        description: '部分处理设施被淹，处理能力下降约40%',
        reporter: '赵刚',
        reportTime: 1689834000000,
        images: []
      },
      {
        id: 'XC-20230720-008',
        name: '西安邮电大学长安校区',
        type: 'building',
        district: '雁塔',
        address: '西安市雁塔区长安南路',
        damageLevel: 'light',
        description: '校园部分路段积水，教学楼无大碍',
        reporter: '孙丽',
        reportTime: 1689834300000,
      },
      {
        id: 'XC-20230720-009',
        name: '西郊通信基站',
        type: 'communication',
        district: 'lianhu',
        address: '西安市莲湖区丰镐西路',
        damageLevel: 'severe',
        description: '基站设备进水损坏，周边5平方公里信号中断',
        reporter: '吴杰',
        reportTime: 1689834600000,
        images: []
      },
      {
        id: 'XC-20230720-010',
        name: '护城河安定门段',
        type: 'water',
        district: 'lianhu',
        address: '西安市莲湖区安定门附近',
        damageLevel: 'moderate',
        description: '河堤出现小型溃口，已临时加固',
        reporter: '郑涛',
        reportTime: 1689834900000,
      },
      {
        id: 'XC-20230720-011',
        name: '纺织城地铁站',
        type: 'building',
        district: 'baqiao',
        address: '西安市灞桥区纺织城正街',
        damageLevel: 'light',
        description: '入口处有积水，已设置警示牌并排水',
        reporter: '钱明',
        reportTime: 1689835200000,
        images: []
      },
      {
        id: 'XC-20230720-012',
        name: '西安北站',
        type: 'building',
        district: 'weiyang',
        address: '西安市未央区元朔路',
        damageLevel: 'moderate',
        description: '地下换乘通道积水，部分列车晚点',
        reporter: '周红',
        reportTime: 1689835500000,
      }
    ]);

    // 筛选后的数据
    const filteredData = computed(() => {
      let result = [...carrierData.value];

      // 类型筛选
      if (selectedType.value !== 'all') {
        result = result.filter(item => item.type === selectedType.value);
      }

      // 受灾程度筛选
      if (damageLevels.value.length > 0) {
        result = result.filter(item => damageLevels.value.includes(item.damageLevel));
      }

      // 区域筛选
      if (selectedDistrict.value !== 'all') {
        result = result.filter(item => item.district === selectedDistrict.value);
      }

      // 搜索筛选
      if (searchQuery.value) {
        const query = searchQuery.value.toLowerCase();
        result = result.filter(item =>
            item.name.toLowerCase().includes(query) ||
            item.id.toLowerCase().includes(query)
        );
      }

      // 分页处理
      const startIndex = (currentPage.value - 1) * pageSize.value;
      return result.slice(startIndex, startIndex + pageSize.value);
    });

    // 总页数
    const totalPages = computed(() => {
      let totalFiltered = [...carrierData.value];

      if (selectedType.value !== 'all') {
        totalFiltered = totalFiltered.filter(item => item.type === selectedType.value);
      }

      if (damageLevels.value.length > 0) {
        totalFiltered = totalFiltered.filter(item => damageLevels.value.includes(item.damageLevel));
      }

      if (selectedDistrict.value !== 'all') {
        totalFiltered = totalFiltered.filter(item => item.district === selectedDistrict.value);
      }

      if (searchQuery.value) {
        const query = searchQuery.value.toLowerCase();
        totalFiltered = totalFiltered.filter(item =>
            item.name.toLowerCase().includes(query) ||
            item.id.toLowerCase().includes(query)
        );
      }

      return Math.ceil(totalFiltered.length / pageSize.value);
    });

    // 统计数据
    const totalCount = computed(() => carrierData.value.length);

    const affectedCount = computed(() => {
      return carrierData.value.filter(item =>
          item.damageLevel === 'moderate' ||
          item.damageLevel === 'severe' ||
          item.damageLevel === 'destroyed'
      ).length;
    });

    const severeCount = computed(() => {
      return carrierData.value.filter(item =>
          item.damageLevel === 'severe' ||
          item.damageLevel === 'destroyed'
      ).length;
    });

    // 图表初始化
    let damageChart = null;
    const initChart = () => {
      const ctx = document.getElementById('damageChart').getContext('2d');

      // 统计各受灾程度的数量
      const lightCount = carrierData.value.filter(item => item.damageLevel === 'light').length;
      const moderateCount = carrierData.value.filter(item => item.damageLevel === 'moderate').length;
      const severeCount = carrierData.value.filter(item => item.damageLevel === 'severe').length;
      const destroyedCount = carrierData.value.filter(item => item.damageLevel === 'destroyed').length;

      if (damageChart) {
        damageChart.destroy();
      }

      damageChart = new Chart(ctx, {
        type: 'doughnut',
        data: {
          labels: ['轻微', '中度', '严重', '损毁'],
          datasets: [{
            data: [lightCount, moderateCount, severeCount, destroyedCount],
            backgroundColor: [
              '#4CAF50',
              '#FFC107',
              '#FF9800',
              '#F44336'
            ],
            borderWidth: 1
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: {
            legend: {
              position: 'bottom',
              labels: {
                boxWidth: 12,
                font: {
                  size: 11
                }
              }
            }
          }
        }
      });
    };

    // 生命周期钩子
    onMounted(() => {
      initChart();
    });

    // 方法
    const filterData = () => {
      currentPage.value = 1;
      initChart();
    };

    const resetFilters = () => {
      selectedType.value = 'all';
      damageLevels.value = ['light', 'moderate', 'severe', 'destroyed'];
      selectedDistrict.value = 'all';
      searchQuery.value = '';
      currentPage.value = 1;
      initChart();
    };

    const handleSearch = () => {
      currentPage.value = 1;
    };

    const changePage = (page) => {
      if (page >= 1 && page <= totalPages.value) {
        currentPage.value = page;
        // 滚动到表格顶部
        const tableContainer = document.querySelector('.data-table-container');
        if (tableContainer) {
          tableContainer.scrollTop = 0;
        }
      }
    };

    const resetPage = () => {
      currentPage.value = 1;
    };

    const viewDetails = (item) => {
      currentItem.value = { ...item };
      showDetails.value = true;
    };

    const closeDetails = () => {
      showDetails.value = false;
      currentItem.value = null;
    };

    const openAddForm = () => {
      isEditing.value = false;
      // 重置表单
      Object.keys(formData).forEach(key => {
        formData[key] = key === 'images' ? [] : '';
      });
      showForm.value = true;
    };

    const editItem = (item) => {
      isEditing.value = true;
      // 填充表单
      Object.keys(formData).forEach(key => {
        formData[key] = item[key] || (key === 'images' ? [] : '');
      });
      currentItem.value = { ...item };
      showForm.value = true;
    };

    const closeForm = () => {
      showForm.value = false;
    };

    const saveItem = () => {
      if (!formData.name || !formData.type || !formData.district || !formData.address || !formData.damageLevel || !formData.description || !formData.reporter) {
        alert('请填写所有必填字段');
        return;
      }

      if (isEditing.value && currentItem.value) {
        // 更新现有项
        const index = carrierData.value.findIndex(item => item.id === currentItem.value.id);
        if (index !== -1) {
          carrierData.value[index] = {
            ...carrierData.value[index],
            ...formData,
            reportTime: Date.now() // 更新上报时间
          };
        }
      } else {
        // 添加新项
        const newId = `XC-${new Date().getFullYear()}${(new Date().getMonth() + 1).toString().padStart(2, '0')}${new Date().getDate().toString().padStart(2, '0')}-${(carrierData.value.length + 1).toString().padStart(3, '0')}`;
        carrierData.value.push({
          id: newId,
          ...formData,
          reportTime: Date.now()
        });
      }

      showForm.value = false;
      updateTime.value = new Date().toLocaleString();
      initChart();
    };

    const deleteItem = (id) => {
      itemToDelete.value = id;
      showDeleteConfirm.value = true;
    };

    const cancelDelete = () => {
      showDeleteConfirm.value = false;
      itemToDelete.value = null;
    };

    const confirmDelete = () => {
      if (itemToDelete.value) {
        carrierData.value = carrierData.value.filter(item => item.id !== itemToDelete.value);
        showDeleteConfirm.value = false;
        itemToDelete.value = null;
        updateTime.value = new Date().toLocaleString();
        initChart();
      }
    };

    const exportData = () => {
      alert('数据导出功能已触发，实际项目中会生成Excel或CSV文件');
    };

    const refreshData = () => {
      updateTime.value = new Date().toLocaleString();
      // 模拟刷新效果
      const table = document.querySelector('.data-table');
      table.classList.add('refreshing');
      setTimeout(() => {
        table.classList.remove('refreshing');
      }, 800);
      initChart();
    };

    const handleImageUpload = (e) => {
      // 实际项目中会上传图片并获取URL，这里仅做模拟
      const files = e.target.files;
      if (files && files.length) {
        for (let i = 0; i < files.length; i++) {
          // 用随机图片URL模拟
          const randomId = 1050 + Math.floor(Math.random() * 20);
          formData.images.push(`https://picsum.photos/id/${randomId}/400/300`);
        }
      }
    };

    // 格式化时间
    const formatTime = (timestamp) => {
      const date = new Date(timestamp);
      return date.toLocaleString('zh-CN', {
        year: 'numeric',
        month: '2-digit',
        day: '2-digit',
        hour: '2-digit',
        minute: '2-digit'
      });
    };

    // 获取类型名称
    const getTypeName = (type) => {
      const typeMap = {
        'building': '建筑物',
        'road': '道路设施',
        'water': '水利设施',
        'power': '电力设施',
        'communication': '通信设施'
      };
      return typeMap[type] || type;
    };

    // 获取区域名称
    const getDistrictName = (district) => {
      const districtMap = {
        'beilin': '碑林区',
        'xincheng': '新城区',
        'lianhu': '莲湖区',
        'baqiao': '灞桥区',
        '雁塔': '雁塔区',
        'weiyang': '未央区',
        'chanba': '浐灞生态区'
      };
      return districtMap[district] || district;
    };

    // 获取受灾程度名称
    const getDamageLevelName = (level) => {
      const levelMap = {
        'light': '轻微',
        'moderate': '中度',
        'severe': '严重',
        'destroyed': '损毁'
      };
      return levelMap[level] || level;
    };

    return {
      updateTime,
      selectedType,
      damageLevels,
      selectedDistrict,
      searchQuery,
      currentPage,
      pageSize,
      showDetails,
      showForm,
      showDeleteConfirm,
      isEditing,
      currentItem,
      formData,
      filteredData,
      totalPages,
      totalCount,
      affectedCount,
      severeCount,
      filterData,
      resetFilters,
      handleSearch,
      changePage,
      resetPage,
      viewDetails,
      closeDetails,
      openAddForm,
      editItem,
      closeForm,
      saveItem,
      deleteItem,
      cancelDelete,
      confirmDelete,
      exportData,
      refreshData,
      handleImageUpload,
      formatTime,
      getTypeName,
      getDistrictName,
      getDamageLevelName
    };
  }
};
</script>

<style lang="scss" scoped>
// 全局样式变量
$primary-color: #165DFF;
$primary-light: #E8F3FF;
$success-color: #00B42A;
$warning-color: #FF7D00;
$danger-color: #F53F3F;
$light-gray: #F2F3F5;
$medium-gray: #C9CDD4;
$dark-gray: #4E5969;
$text-color: #1D2129;
$border-radius: 4px;
$shadow: 0 2px 10px rgba(0, 0, 0, 0.08);

.disaster-container {
  min-height: 100vh;
  background-color: #F7F8FA;
  color: $text-color;
  font-family: 'Inter', system-ui, sans-serif;
}

// 导航栏样式
.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 20px;
  height: 60px;
  background-color: #fff;
  box-shadow: $shadow;
  position: sticky;
  top: 0;
  z-index: 100;

  .logo {
    display: flex;
    align-items: center;
    gap: 10px;

    .icon-water {
      font-size: 24px;
      color: $primary-color;
    }

    h1 {
      font-size: 18px;
      font-weight: 600;
      margin: 0;
    }
  }

  .nav-actions {
    display: flex;
    align-items: center;
    gap: 15px;

    .time-info {
      color: $dark-gray;
      font-size: 14px;
    }
  }
}

// 主内容区样式
.main-content {
  display: flex;
  padding: 20px;
  gap: 20px;
  max-width: 1600px;
  margin: 0 auto;
}

// 筛选面板样式
.filter-panel {
  width: 300px;
  background-color: #fff;
  border-radius: $border-radius;
  box-shadow: $shadow;
  padding: 20px;
  height: fit-content;

  .panel-title {
    margin-bottom: 20px;
    h2 {
      font-size: 16px;
      font-weight: 600;
      margin: 0;
      padding-bottom: 8px;
      border-bottom: 1px solid $light-gray;
    }
  }

  .filter-group {
    margin-bottom: 20px;

    .filter-label {
      display: block;
      margin-bottom: 8px;
      font-weight: 500;
      font-size: 14px;
    }

    .filter-select {
      width: 100%;
      padding: 8px 12px;
      border: 1px solid $medium-gray;
      border-radius: $border-radius;
      font-size: 14px;
      appearance: none;
      background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='12' viewBox='0 0 24 24' fill='none' stroke='%234E5969' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpolyline points='6 9 12 15 18 9'%3E%3C/polyline%3E%3C/svg%3E");
      background-repeat: no-repeat;
      background-position: right 12px center;
    }

    .checkbox-group {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;

      .checkbox-item {
        display: flex;
        align-items: center;
        gap: 5px;
        font-size: 14px;
        cursor: pointer;
      }
    }
  }

  .filter-actions {
    display: flex;
    gap: 10px;
    margin-bottom: 30px;
  }

  .stats-card {
    background-color: $light-gray;
    border-radius: $border-radius;
    padding: 15px;

    h3 {
      font-size: 15px;
      margin-top: 0;
      margin-bottom: 15px;
    }

    .stat-item {
      display: flex;
      justify-content: space-between;
      margin-bottom: 10px;
      font-size: 14px;

      .stat-label {
        color: $dark-gray;
      }

      .stat-value {
        font-weight: 600;

        &.affected {
          color: $warning-color;
        }

        &.severe {
          color: $danger-color;
        }
      }
    }

    .stat-chart {
      height: 180px;
      margin-top: 15px;
    }
  }
}

// 内容面板样式
.content-panel {
  flex: 1;
  background-color: #fff;
  border-radius: $border-radius;
  box-shadow: $shadow;
  padding: 20px;

  .panel-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;

    h2 {
      font-size: 18px;
      font-weight: 600;
      margin: 0;
    }

    .search-bar {
      display: flex;
      gap: 10px;
      align-items: center;
      width: 500px;

      input {
        flex: 1;
        padding: 8px 12px;
        border: 1px solid $medium-gray;
        border-radius: $border-radius;
        font-size: 14px;

        &::placeholder {
          color: $medium-gray;
        }
      }
    }
  }

  // 表格样式
  .data-table-container {
    overflow-x: auto;
    margin-bottom: 20px;

    .data-table {
      width: 100%;
      border-collapse: collapse;
      min-width: 1000px;

      &.refreshing {
        animation: refresh 0.8s ease-in-out;
      }

      thead {
        background-color: $light-gray;

        th {
          padding: 12px 15px;
          text-align: left;
          font-weight: 600;
          font-size: 14px;
          border-bottom: 1px solid $medium-gray;
        }
      }

      tbody {
        .table-row {
          transition: background-color 0.2s;

          &:hover {
            background-color: $primary-light;
          }

          td {
            padding: 12px 15px;
            font-size: 14px;
            border-bottom: 1px solid $light-gray;

            &.description-cell {
              max-width: 200px;
              white-space: nowrap;
              overflow: hidden;
              text-overflow: ellipsis;
            }

            &.action-buttons {
              display: flex;
              gap: 5px;
            }
          }
        }

        .no-data {
          text-align: center;
          color: $dark-gray;
          height: 100px;
        }
      }
    }
  }

  // 分页样式
  .pagination {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 15px;
    margin-top: 20px;

    .page-info {
      font-size: 14px;
      color: $dark-gray;
    }

    .page-size {
      display: flex;
      align-items: center;
      gap: 5px;
      font-size: 14px;

      select {
        padding: 4px 8px;
        border: 1px solid $medium-gray;
        border-radius: $border-radius;
        font-size: 13px;
      }
    }
  }
}

// 标签样式
.type-tag {
  display: inline-block;
  padding: 3px 8px;
  border-radius: 4px;
  font-size: 12px;
  font-weight: 500;

  &.building {
    background-color: #E8F3FF;
    color: $primary-color;
  }

  &.road {
    background-color: #FFF7E8;
    color: #FF7D00;
  }

  &.water {
    background-color: #E8F7FF;
    color: #0FC6C2;
  }

  &.power {
    background-color: #FDECEC;
    color: #F53F3F;
  }

  &.communication {
    background-color: #F3E8FF;
    color: #722ED1;
  }
}

.damage-tag {
  display: inline-block;
  padding: 3px 8px;
  border-radius: 4px;
  font-size: 12px;
  font-weight: 500;

  &.light {
    background-color: #E8FFEA;
    color: #00B42A;
  }

  &.moderate {
    background-color: #FFF7E8;
    color: #FF7D00;
  }

  &.severe {
    background-color: #FFE8E8;
    color: #F53F3F;
  }

  &.destroyed {
    background-color: #F2E8E8;
    color: #B71C1C;
  }
}

// 按钮样式
.btn {
  padding: 8px 16px;
  border-radius: $border-radius;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
  border: none;
  display: inline-flex;
  align-items: center;
  gap: 5px;

  &.btn-primary {
    background-color: $primary-color;
    color: white;

    &:hover {
      background-color: #0E42D2;
    }
  }

  &.btn-outline {
    background-color: transparent;
    color: $primary-color;
    border: 1px solid $primary-color;

    &:hover {
      background-color: $primary-light;
    }

    &:disabled {
      color: $medium-gray;
      border-color: $medium-gray;
      cursor: not-allowed;
      background-color: transparent;
    }
  }

  &.btn-danger {
    background-color: $danger-color;
    color: white;

    &:hover {
      background-color: #E02020;
    }
  }

  &.btn-icon {
    padding: 6px;
    width: 32px;
    height: 32px;
    justify-content: center;

    &.danger {
      color: $danger-color;

      &:hover {
        background-color: #FEE;
      }
    }
  }
}

// 弹窗样式
.modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 1000;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;

  .modal-overlay {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: rgba(0, 0, 0, 0.5);
    backdrop-filter: blur(2px);
    cursor: pointer;
  }

  .modal-content {
    position: relative;
    background-color: white;
    border-radius: $border-radius;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
    width: 100%;
    max-width: 600px;
    max-height: 90vh;
    display: flex;
    flex-direction: column;

    &.form-modal {
      max-width: 700px;
    }

    &.confirm-modal {
      max-width: 400px;
    }

    .modal-header {
      padding: 16px 20px;
      border-bottom: 1px solid $light-gray;
      display: flex;
      justify-content: space-between;
      align-items: center;

      h3 {
        margin: 0;
        font-size: 18px;
        font-weight: 600;
      }

      .modal-close {
        background: none;
        border: none;
        font-size: 20px;
        cursor: pointer;
        color: $dark-gray;
        width: 30px;
        height: 30px;
        display: flex;
        align-items: center;
        justify-content: center;
        border-radius: 50%;
        transition: background-color 0.2s;

        &:hover {
          background-color: $light-gray;
        }
      }
    }

    .modal-body {
      padding: 20px;
      overflow-y: auto;
      flex: 1;

      .detail-row {
        display: flex;
        margin-bottom: 15px;
        font-size: 14px;

        &.images-row {
          flex-direction: column;

          .images-container {
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
            margin-top: 8px;

            .damage-image {
              width: 120px;
              height: 90px;
              object-fit: cover;
              border-radius: 4px;
              cursor: pointer;
              transition: transform 0.2s;

              &:hover {
                transform: scale(1.05);
              }
            }
          }
        }

        .detail-label {
          width: 100px;
          font-weight: 500;
          color: $dark-gray;
        }

        .detail-value {
          flex: 1;
        }
      }
    }

    .modal-footer {
      padding: 16px 20px;
      border-top: 1px solid $light-gray;
      display: flex;
      justify-content: flex-end;
      gap: 10px;
    }
  }
}

// 表单样式
.data-form {
  .form-group {
    margin-bottom: 16px;

    .form-label {
      display: block;
      margin-bottom: 8px;
      font-weight: 500;
      font-size: 14px;

      .required {
        color: $danger-color;
      }
    }

    .form-control {
      width: 100%;
      padding: 10px 12px;
      border: 1px solid $medium-gray;
      border-radius: $border-radius;
      font-size: 14px;
      font-family: inherit;

      &:focus {
        outline: none;
        border-color: $primary-color;
        box-shadow: 0 0 0 2px rgba(22, 93, 255, 0.2);
      }

      &[type="file"] {
        padding: 8px;
        border: 1px dashed $medium-gray;
        background-color: $light-gray;
      }

      &textarea {
        resize: vertical;
      }
    }
  }
}

// 动画
@keyframes refresh {
  0%, 100% {
    opacity: 1;
  }
  50% {
    opacity: 0.6;
  }
}

// 响应式设计
@media (max-width: 1200px) {
  .main-content {
    flex-direction: column;
  }

  .filter-panel {
    width: 100%;
  }

  .content-panel .panel-header .search-bar {
    width: auto;
    flex: 1;
    min-width: 200px;
  }
}

@media (max-width: 768px) {
  .navbar {
    flex-direction: column;
    height: auto;
    padding: 10px 20px;
    gap: 10px;
  }

  .content-panel .panel-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 10px;
  }

  .content-panel .panel-header .search-bar {
    width: 100%;
  }

  .pagination {
    flex-wrap: wrap;
  }
}
</style>