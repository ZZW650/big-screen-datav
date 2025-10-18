<template>
  <div>
    <div class="map-controls">
      <button @click="toggleDrawMode">
        {{ isDrawing ? '结束绘制' : '开始绘制圆' }}
      </button>
      <button @click="clearAllCircles">清除所有圆</button>
      <div v-if="isDrawing" class="radius-control">
        <label>半径(米):</label>
        <input
            type="number"
            v-model.number="circleRadius"
            min="100"
            step="100"
            placeholder="输入半径"
        >
      </div>
    </div>
    <div id="container"></div>
  </div>
</template>

<script setup>
import { onMounted, onUnmounted, ref } from "vue";
import AMapLoader from "@amap/amap-jsapi-loader";

let map = null;
let currentCircle = null;
const drawnCircles = [];
const isDrawing = ref(false);
const circleRadius = ref(1000); // 默认半径1000米

// 西安的经纬度坐标（GCJ-02坐标系）
const XI_AN_COORDS = [108.95, 34.27];

onMounted(() => {
  window._AMapSecurityConfig = {
    securityJsCode: "c8e118cd71b1ab650c73d86a6eaa7cba",
  };

  AMapLoader.load({
    key: "c8e118cd71b1ab650c73d86a6eaa7cba",
    version: "2.0",
    plugins: ["AMap.Scale"], // 移除了未使用的MouseTool插件
  })
      .then((AMap) => {
        // 初始化地图，中心点设为西安
        map = new AMap.Map("container", {
          viewMode: "3D",
          zoom: 12,
          center: XI_AN_COORDS,
          resizeEnable: true // 启用地图自适应容器大小
        });

        // 添加比例尺控件
        map.addControl(new AMap.Scale());

        // 地图点击事件处理函数
        const handleMapClick = (e) => {
          if (!isDrawing.value) return;

          // 关键修复：显式提取经纬度为数组格式 [lng, lat]
          const clickCoords = [e.lnglat.getLng(), e.lnglat.getLat()];

          // 调试用：打印点击坐标，可在控制台查看是否正确
          console.log("点击坐标:", clickCoords);

          // 如果已有临时圆，先移除
          if (currentCircle) {
            map.remove(currentCircle);
          }

          // 创建新圆（使用数组格式的坐标）
          currentCircle = new AMap.Circle({
            center: clickCoords, // 改用数组格式的坐标
            radius: circleRadius.value,
            strokeColor: "#3366FF",
            strokeOpacity: 1,
            strokeWeight: 2,
            fillColor: "#3366FF",
            fillOpacity: 0.3,
            zIndex: 10 // 确保圆在地图图层上方
          });

          // 添加到地图
          map.add(currentCircle); // 改用map.add()方法更稳妥
          // 保存圆实例
          drawnCircles.push(currentCircle);
        };

        // 绑定地图点击事件
        map.on('click', handleMapClick);

        // 组件卸载时清理事件
        onUnmounted(() => {
          map.off('click', handleMapClick);
        });
      })
      .catch((e) => {
        console.error("地图加载失败:", e);
      });
});

// 切换绘制模式
const toggleDrawMode = () => {// eslint-disable-line no-unused-vars
  isDrawing.value = !isDrawing.value;
  if (!isDrawing.value) {
    currentCircle = null;
  }
};

// 清除所有绘制的圆
const clearAllCircles = () => {// eslint-disable-line no-unused-vars
  if (map && drawnCircles.length > 0) {
    map.remove(drawnCircles);
    drawnCircles.length = 0;
    currentCircle = null;
  }
};

onUnmounted(() => {
  if (map) {
    map.destroy();
    map = null;
  }
});
</script>

<style scoped>
#container {
  width: 100%;
  height: 800px;
  position: relative; /* 确保地图容器是相对定位，避免控件定位异常 */
}

.map-controls {
  position: absolute;
  top: 135px;
  left: 20px;
  z-index: 100;
  background: white;
  padding: 10px;
  border-radius: 4px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  display: flex;
  gap: 10px;
  align-items: center;
}

button {
  padding: 6px 12px;
  background-color: #1677ff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
}

button:hover {
  background-color: #0f62d9;
}

.radius-control {
  display: flex;
  align-items: center;
  gap: 5px;
}

input {
  width: 100px;
  padding: 4px 6px;
  border: 1px solid #ddd;
  border-radius: 4px;
}
</style>