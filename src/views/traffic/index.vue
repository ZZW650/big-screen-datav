<template>
  <div class="knowledge-graph">
    <div class="graph-header">
      <div class="header-content">
        <h3>西安多灾种灾害知识图谱</h3>
        <div class="legend">
          <span class="legend-item">
            <div class="legend-color disaster"></div>灾害类型
          </span>
          <span class="legend-item">
            <div class="legend-color event"></div>突发事件
          </span>
          <span class="legend-item">
            <div class="legend-color resource"></div>应急资源
          </span>
          <span class="legend-item">
            <div class="legend-color department"></div>责任部门
          </span>
        </div>
      </div>
    </div>

    <!-- 图谱容器 -->
    <div class="graph-container" ref="graphContainer">
      <!-- 连接线 -->
      <svg class="graph-links" :width="containerWidth" :height="containerHeight">
        <line
            v-for="link in computedLinks"
            :key="link.id"
            :x1="link.source.x"
            :y1="link.source.y"
            :x2="link.target.x"
            :y2="link.target.y"
            class="link"
            :class="{ 'link-highlighted': isLinkHighlighted(link) }"
        />
      </svg>

      <!-- 节点区域 -->
      <div class="graph-nodes">
        <!-- 第一列：灾害类型 -->
        <div class="node-column" :style="{ left: `${col1X}%` }">
          <div class="column-header">
            <h4 class="column-title">灾害类型</h4>
            <div class="column-line disaster"></div>
          </div>
          <div
              v-for="node in disasterNodes"
              :key="node.id"
              :class="['node', node.type, {
                'node-active': selectedNode?.id === node.id,
                'node-highlighted': isNodeHighlighted(node)
              }]"
              :style="{ top: `${node.y}px` }"
              @click="onNodeClick(node)"
          >
            <div class="node-content">
              <div class="node-icon">{{ getNodeIcon(node.type) }}</div>
              <div class="node-label">{{ node.label }}</div>
            </div>
          </div>
        </div>

        <!-- 第二列：突发事件 -->
        <div class="node-column" :style="{ left: `${col2X}%` }">
          <div class="column-header">
            <h4 class="column-title">突发事件</h4>
            <div class="column-line event"></div>
          </div>
          <div
              v-for="node in eventNodes"
              :key="node.id"
              :class="['node', node.type, {
                'node-active': selectedNode?.id === node.id,
                'node-highlighted': isNodeHighlighted(node)
              }]"
              :style="{ top: `${node.y}px` }"
              @click="onNodeClick(node)"
          >
            <div class="node-content">
              <div class="node-icon">{{ getNodeIcon(node.type) }}</div>
              <div class="node-label">{{ node.label }}</div>
            </div>
          </div>
        </div>

        <!-- 第三列：应急资源 -->
        <div class="node-column" :style="{ left: `${col3X}%` }">
          <div class="column-header">
            <h4 class="column-title">应急资源</h4>
            <div class="column-line resource"></div>
          </div>
          <div
              v-for="node in resourceNodes"
              :key="node.id"
              :class="['node', node.type, {
                'node-active': selectedNode?.id === node.id,
                'node-highlighted': isNodeHighlighted(node)
              }]"
              :style="{ top: `${node.y}px` }"
              @click="onNodeClick(node)"
          >
            <div class="node-content">
              <div class="node-icon">{{ getNodeIcon(node.type) }}</div>
              <div class="node-label">{{ node.label }}</div>
            </div>
          </div>
        </div>

        <!-- 第四列：责任部门 -->
        <div class="node-column" :style="{ left: `${col4X}%` }">
          <div class="column-header">
            <h4 class="column-title">责任部门</h4>
            <div class="column-line department"></div>
          </div>
          <div
              v-for="node in departmentNodes"
              :key="node.id"
              :class="['node', node.type, {
                'node-active': selectedNode?.id === node.id,
                'node-highlighted': isNodeHighlighted(node)
              }]"
              :style="{ top: `${node.y}px` }"
              @click="onNodeClick(node)"
          >
            <div class="node-content">
              <div class="node-icon">{{ getNodeIcon(node.type) }}</div>
              <div class="node-label">{{ node.label }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 详情面板 -->
    <div class="graph-info" v-if="selectedNode">
      <div class="info-header">
        <h4>{{ selectedNode.label }}</h4>
        <span class="info-tag" :class="`tag-${selectedNode.type}`">
          {{ getNodeTypeName(selectedNode.type) }}
        </span>
      </div>
      <div class="info-content">
        <p>{{ selectedNode.description }}</p>
        <div v-if="selectedNode.related" class="related-info">
          <strong>关联信息：</strong>
          <ul>
            <li v-for="item in selectedNode.related" :key="item">{{ item }}</li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'KnowledgeGraph',
  data() {
    return {
      containerWidth: 0,
      containerHeight: 0,
      selectedNode: null,
      highlightedNodes: new Set(),
      highlightedLinks: new Set(),
      // 节点数据保持不变
      disasterNodes: [
        { id: 1, type: 'disaster', label: '地震灾害',  description: '西安地处渭河地震带，防范6级以上地震灾害' },
        { id: 2, type: 'disaster', label: '气象灾害', description: '暴雨、干旱、高温、寒潮等极端天气事件' },
        { id: 3, type: 'disaster', label: '地质灾害',  description: '滑坡、泥石流、地面沉降等地质灾害' },
        { id: 4, type: 'disaster', label: '洪水灾害',  description: '渭河、浐河、灞河等河流洪水风险' }
      ],
      eventNodes: [
        { id: 5, type: 'event', label: '建筑物倒塌',  description: '地震导致的建筑物倒塌事件' },
        { id: 6, type: 'event', label: '城市内涝',  description: '暴雨引发的城市道路积水内涝' },
        { id: 7, type: 'event', label: '交通中断',  description: '灾害导致的公路、铁路交通瘫痪' },
        { id: 8, type: 'event', label: '电力故障',  description: '灾害引发的电力供应中断' },
        { id: 9, type: 'event', label: '通讯中断',  description: '通信基站损坏导致的通讯中断' },
        { id: 10, type: 'event', label: '人员伤亡',  description: '灾害造成的人员伤亡事件' }
      ],
      resourceNodes: [
        { id: 11, type: 'resource', label: '应急救援队',  description: '消防、武警等专业应急救援队伍' },
        { id: 12, type: 'resource', label: '医疗救护',  description: '医院、急救中心等医疗资源' },
        { id: 13, type: 'resource', label: '避难场所', description: '学校、体育馆等应急避难场所' },
        { id: 14, type: 'resource', label: '物资储备',  description: '食品、饮用水、帐篷等物资储备' },
        { id: 15, type: 'resource', label: '运输车辆',  description: '应急运输和救援车辆' },
        { id: 16, type: 'resource', label: '通信设备',  description: '卫星电话、对讲机等应急通信设备' }
      ],
      departmentNodes: [
        { id: 17, type: 'department', label: '应急管理局',  description: '西安市应急管理局，统筹协调' },
        { id: 18, type: 'department', label: '气象局',  description: '西安市气象局，预警发布' },
        { id: 19, type: 'department', label: '住建局',  description: '西安市住建局，建筑安全' },
        { id: 20, type: 'department', label: '交通局',  description: '西安市交通局，交通保障' },
        { id: 21, type: 'department', label: '卫健委',  description: '西安市卫健委，医疗救援' },
        { id: 22, type: 'department', label: '水务局',  description: '西安市水务局，防洪防汛' }
      ],
      linkRules: [
        // 保持原有的连接规则不变
        { id: 1, sourceId: 1, targetId: 5 },
        { id: 2, sourceId: 1, targetId: 7 },
        { id: 3, sourceId: 1, targetId: 8 },
        { id: 4, sourceId: 2, targetId: 6 },
        { id: 5, sourceId: 3, targetId: 7 },
        { id: 6, sourceId: 4, targetId: 6 },
        { id: 7, sourceId: 5, targetId: 11 },
        { id: 8, sourceId: 5, targetId: 12 },
        { id: 9, sourceId: 6, targetId: 13 },
        { id: 10, sourceId: 7, targetId: 15 },
        { id: 11, sourceId: 8, targetId: 16 },
        { id: 12, sourceId: 10, targetId: 12 },
        { id: 13, sourceId: 11, targetId: 17 },
        { id: 14, sourceId: 12, targetId: 21 },
        { id: 15, sourceId: 13, targetId: 17 },
        { id: 16, sourceId: 15, targetId: 20 },
        { id: 17, sourceId: 16, targetId: 20 },
      ]
    };
  },
  computed: {
    allNodes() {
      return [
        ...this.disasterNodes,
        ...this.eventNodes,
        ...this.resourceNodes,
        ...this.departmentNodes
      ];
    },
    computedLinks() {
      const colXMap = {
        disaster: this.col1X,
        event: this.col2X,
        resource: this.col3X,
        department: this.col4X
      };
      const getX = (type) => (this.containerWidth * colXMap[type]) / 100 ;

      return this.linkRules.map(rule => {
        const sourceNode = this.allNodes.find(n => n.id === rule.sourceId);
        const targetNode = this.allNodes.find(n => n.id === rule.targetId);
        return {
          ...rule,
          source: { x: getX(sourceNode.type), y: sourceNode.y },
          target: { x: getX(targetNode.type), y: targetNode.y }
        };
      });
    },
    col1X() { return 12; },
    col2X() { return 36; },
    col3X() { return 64; },
    col4X() { return 88; }
  },
  mounted() {
    this.initViewportSize();
    this.calculateNodeY();
    window.addEventListener('resize', this.handleResize);
  },
  beforeDestroy() {
    window.removeEventListener('resize', this.handleResize);
  },
  methods: {
    initViewportSize() {
      const parentEl = this.$refs.graphContainer?.parentElement;
      if (!parentEl) return;

      const headerEl = this.$el.querySelector('.graph-header');
      const infoEl = this.$el.querySelector('.graph-info');

      // 用clientHeight（不含边框）计算，避免边框挤压
      const headerHeight = headerEl ? headerEl.clientHeight : 0;
      const infoHeight = infoEl ? infoEl.clientHeight : 0;
      const parentHeight = parentEl.clientHeight;

      // 容器宽高 = 父容器宽高 - 头部和详情面板高度
      this.containerWidth = parentEl.clientWidth;
      this.containerHeight = parentHeight - headerHeight - infoHeight;

      const container = this.$refs.graphContainer;
      if (container) {
        container.style.width = `${this.containerWidth}px`;
        container.style.height = `${this.containerHeight}px`;
      }
    },
    handleResize() {
      this.initViewportSize();
      this.calculateNodeY();
    },
    calculateNodeY() {
      // 为每个节点组计算起始位置和间距，确保垂直对齐
      const nodeGroups = [
        this.disasterNodes,
        this.eventNodes,
        this.resourceNodes,
        this.departmentNodes
      ];

      nodeGroups.forEach(group => {
        // 计算组内节点数量
        const nodeCount = group.length;
        // 计算节点之间的间距
        const spacing = this.containerHeight / (nodeCount + 1);

        // 按顺序排列节点，确保垂直对齐
        group.forEach((node, index) => {
          // 从spacing开始，均匀分布
          node.y = (index + 1) * spacing - 25; // 25是节点高度的一半，用于垂直居中
        });
      });
    },
    getNodeIcon(type) {
      const icons = {
        disaster: '⚡',
        event: '🚨',
        resource: '🛡️',
        department: '🏢'
      };
      return icons[type] || '●';
    },
    getNodeTypeName(type) {
      const names = {
        disaster: '灾害类型',
        event: '突发事件',
        resource: '应急资源',
        department: '责任部门'
      };
      return names[type] || '未知类型';
    },
    onNodeClick(node) {
      this.selectedNode = node;
      this.updateHighlightedElements(node);
      setTimeout(() => {
        this.initViewportSize();
        this.calculateNodeY();
      }, 100);
    },
    updateHighlightedElements(selectedNode) {
      this.highlightedNodes.clear();
      this.highlightedLinks.clear();

      if (!selectedNode) return;

      // 添加当前选中节点
      this.highlightedNodes.add(selectedNode.id);

      // 查找相关连接并确保连接线正确高亮
      this.linkRules.forEach(link => {
        if (link.sourceId === selectedNode.id || link.targetId === selectedNode.id) {
          this.highlightedLinks.add(link.id);
          this.highlightedNodes.add(link.sourceId);
          this.highlightedNodes.add(link.targetId);
        }
      });

      // 强制重新计算布局
      this.$nextTick(() => {
        this.calculateNodeY();
      });
    },
    isNodeHighlighted(node) {
      return this.highlightedNodes.has(node.id);
    },
    isLinkHighlighted(link) {
      return this.highlightedLinks.has(link.id);
    }
  }
};
</script>

<style scoped>
/* 重置和基础样式 */
.knowledge-graph {
  height: 126vh;
  min-height: 100vh; /* 确保内容不足时仍占满屏幕 */
  display: flex;
  flex-direction: column;
  overflow: hidden;
  background: #f8fafc;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  margin: 0; /* 清除默认margin */
  padding: 0; /* 清除默认padding */
}

.graph-header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  z-index: 100;
}

.header-content {
  padding: 16px 24px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  max-width: 1400px;
  margin: 0 auto;
  width: 100%;
}

.graph-header h3 {
  margin: 0;
  font-size: 20px;
  font-weight: 600;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
}

/* 图例样式优化 */
.legend {
  display: flex;
  gap: 20px;
  background: rgba(255, 255, 255, 0.15);
  padding: 8px 16px;
  border-radius: 20px;
  backdrop-filter: blur(10px);
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 13px;
  font-weight: 500;
}

.legend-color {
  width: 12px;
  height: 12px;
  border-radius: 3px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
}

.legend-color.disaster { background: linear-gradient(135deg, #ef4444, #dc2626); }
.legend-color.event { background: linear-gradient(135deg, #f59e0b, #d97706); }
.legend-color.resource { background: linear-gradient(135deg, #22c55e, #16a34a); }
.legend-color.department { background: linear-gradient(135deg, #3b82f6, #2563eb); }

/* 图谱容器样式优化 */
.graph-container {
  flex: 1; /* 占满父容器剩余空间 */
  width: 100%; /* 确保宽度100% */
  position: relative;
  overflow: auto;
  background:
      radial-gradient(circle at 20% 80%, rgba(239, 68, 68, 0.03) 0%, transparent 50%),
      radial-gradient(circle at 80% 20%, rgba(59, 130, 246, 0.03) 0%, transparent 50%),
      linear-gradient(135deg, #f8fafc 0%, #f1f5f9 100%);
  margin: 0;
  padding: 0;
}

/* 节点区域样式 */
.graph-nodes {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

/* 节点列样式优化 */
.node-column {
  position: absolute;
  top: 0;
  width: 140px;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 30px 0;
  box-sizing: border-box;
}

.column-header {
  width: 100%;
  text-align: center;
  margin-bottom: 40px;
}

.column-title {
  margin: 0 0 8px 0;
  padding: 8px 12px;
  background: white;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 600;
  color: #1e293b;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  border: 1px solid;
  border-color: inherit;
}

.column-line {
  height: 3px;
  width: 60%;
  margin: 0 auto;
  border-radius: 2px;
  opacity: 0.6;
}

.node-column:nth-child(1) .column-title {
  border-color: #ef4444;
  color: #ef4444;
}
.node-column:nth-child(1) .column-line { background: #ef4444; }

.node-column:nth-child(2) .column-title {
  border-color: #f59e0b;
  color: #f59e0b;
}
.node-column:nth-child(2) .column-line { background: #f59e0b; }

.node-column:nth-child(3) .column-title {
  border-color: #22c55e;
  color: #22c55e;
}
.node-column:nth-child(3) .column-line { background: #22c55e; }

.node-column:nth-child(4) .column-title {
  border-color: #3b82f6;
  color: #3b82f6;
}
.node-column:nth-child(4) .column-line { background: #3b82f6; }

/* 节点样式优化 */
.node {
  position: absolute;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  transform: translateX(-50%);
  margin-bottom: 12px;
  width: 100%;
  z-index: 2;
}

.node:hover {
  transform: translateX(-50%) scale(1.08);
  z-index: 10;
}

.node-highlighted {
  transform: translateX(-50%) scale(1.05);
  z-index: 5;
}

.node-active .node-content {
  box-shadow:
      0 0 0 3px rgba(59, 130, 246, 0.3),
      0 8px 24px rgba(0, 0, 0, 0.15),
      0 4px 12px rgba(0, 0, 0, 0.1);
  transform: translateY(-2px);
}

.node-content {
  padding: 12px 8px;
  border-radius: 12px;
  text-align: center;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.08);
  width: 100%;
  box-sizing: border-box;
  background: white;
  border: 2px solid transparent;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.node-content::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 3px;
  background: inherit;
  opacity: 0.8;
}

.node.disaster .node-content {
  border-color: #ef4444;
  color: #b91c1c;
  background: linear-gradient(135deg, #fef2f2, #fecaca);
}
.node.event .node-content {
  border-color: #f59e0b;
  color: #92400e;
  background: linear-gradient(135deg, #fffbeb, #fed7aa);
}
.node.resource .node-content {
  border-color: #22c55e;
  color: #166534;
  background: linear-gradient(135deg, #f0fdf4, #bbf7d0);
}
.node.department .node-content {
  border-color: #3b82f6;
  color: #1e40af;
  background: linear-gradient(135deg, #eff6ff, #dbeafe);
}

.node-icon {
  font-size: 20px;
  margin-bottom: 6px;
  filter: drop-shadow(0 1px 2px rgba(0, 0, 0, 0.1));
}

.node-label {
  font-size: 13px;
  font-weight: 600;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  line-height: 1.3;
}

/* 连接线样式优化 */
.graph-links {
  position: absolute;
  top: 0;
  left: 0;
  pointer-events: none;
  z-index: 1;
}

.link {
  stroke: #cbd5e1;
  stroke-width: 2;
  stroke-dasharray: 5, 3;
  transition: all 0.3s ease;
}

.link-highlighted {
  stroke-width: 3;
  stroke-dasharray: none;
  filter: drop-shadow(0 1px 2px rgba(0, 0, 0, 0.2));
}

.link-highlighted:nth-child(6n+1) { stroke: #ef4444; }
.link-highlighted:nth-child(6n+2) { stroke: #f59e0b; }
.link-highlighted:nth-child(6n+3) { stroke: #22c55e; }
.link-highlighted:nth-child(6n+4) { stroke: #3b82f6; }
.link-highlighted:nth-child(6n+5) { stroke: #8b5cf6; }
.link-highlighted:nth-child(6n) { stroke: #ec4899; }

/* 详情面板样式优化 */
.graph-info {
  padding: 20px 24px;
  border-top: 1px solid #e2e8f0;
  background: white;
  max-height: 200px;
  overflow-y: auto;
  box-shadow: 0 -4px 12px rgba(0, 0, 0, 0.05);
  box-sizing: border-box;
}

.info-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.info-header h4 {
  margin: 0;
  color: #1e293b;
  font-size: 16px;
  font-weight: 600;
}

.info-tag {
  font-size: 12px;
  padding: 4px 12px;
  border-radius: 12px;
  color: white;
  font-weight: 500;
  text-shadow: 0 1px 1px rgba(0, 0, 0, 0.2);
}

.tag-disaster { background: linear-gradient(135deg, #ef4444, #dc2626); }
.tag-event { background: linear-gradient(135deg, #f59e0b, #d97706); }
.tag-resource { background: linear-gradient(135deg, #22c55e, #16a34a); }
.tag-department { background: linear-gradient(135deg, #3b82f6, #2563eb); }

.info-content {
  font-size: 14px;
  color: #475569;
  line-height: 1.6;
}

.info-content p {
  margin: 0 0 12px;
}

.related-info {
  margin-top: 12px;
  padding-top: 12px;
  border-top: 1px solid #f1f5f9;
}

.related-info strong {
  color: #334155;
  font-size: 14px;
  display: block;
  margin-bottom: 6px;
}

.related-info ul {
  margin: 0;
  padding-left: 20px;
}

.related-info li {
  margin: 4px 0;
  font-size: 13px;
  color: #64748b;
}

/* 滚动条美化 */
.graph-container::-webkit-scrollbar,
.graph-info::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

.graph-container::-webkit-scrollbar-thumb,
.graph-info::-webkit-scrollbar-thumb {
  background-color: #cbd5e1;
  border-radius: 4px;
}

.graph-container::-webkit-scrollbar-thumb:hover,
.graph-info::-webkit-scrollbar-thumb:hover {
  background-color: #94a3b8;
}

.graph-container::-webkit-scrollbar-track,
.graph-info::-webkit-scrollbar-track {
  background-color: #f1f5f9;
  border-radius: 4px;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .header-content {
    flex-direction: column;
    gap: 12px;
    text-align: center;
  }

  .legend {
    flex-wrap: wrap;
    justify-content: center;
  }

  .node-column {
    width: 120px;
    padding: 20px 0;
  }

  .column-title {
    font-size: 12px;
    padding: 6px 8px;
  }
}

/* 动画效果 */
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

.node-content {
  animation: fadeIn 0.4s ease-out;
}

.graph-info {
  animation: fadeIn 0.3s ease-out;
}
</style>