<template>
  <div class="statistics">
    <h2 class="page-title animate-fade-in">统计报表</h2>

    <!-- 时间范围选择器 -->
    <div class="time-selector animate-slide-down">
      <a-radio-group v-model:value="selectedPeriod" button-style="solid">
        <a-radio-button value="month">月度</a-radio-button>
        <a-radio-button value="quarter">季度</a-radio-button>
        <a-radio-button value="year">年度</a-radio-button>
      </a-radio-group>
      <a-button type="primary" class="export-btn" @click="handleExport">
        <DownloadOutlined /> 导出报表
      </a-button>
    </div>

    <!-- 借阅趋势图表 -->
    <a-row :gutter="[16, 16]" class="chart-row">
      <a-col :xs="24" :lg="16">
        <div class="card-container animate-fade-in" style="animation-delay: 0.1s">
          <div class="card-header">
            <h3 class="card-title">
              <LineChartOutlined /> 借阅趋势统计
            </h3>
          </div>
          <div class="card-body chart-container">
            <div class="chart-placeholder">
              <div class="chart-bars">
                <div v-for="(item, index) in borrowTrendData" :key="index" class="bar-item">
                  <div class="bar-label">{{ item.label }}</div>
                  <div class="bar-wrapper">
                    <div 
                      class="bar-fill" 
                      :style="{ 
                        height: `${(item.value / maxBorrowValue) * 100}%`,
                        backgroundColor: getBarColor(index)
                      }"
                    ></div>
                  </div>
                  <div class="bar-value">{{ item.value }}</div>
                </div>
              </div>
              <div class="chart-legend">
                <span class="legend-item">
                  <span class="legend-color" style="background: #1890ff"></span>
                  借阅数量
                </span>
              </div>
            </div>
          </div>
        </div>
      </a-col>
      <a-col :xs="24" :lg="8">
        <div class="card-container equal-height animate-fade-in" style="animation-delay: 0.2s">
          <div class="card-header">
            <h3 class="card-title">
              <PieChartOutlined /> 借阅分类占比
            </h3>
          </div>
          <div class="card-body chart-container">
            <div class="pie-chart-placeholder">
              <div class="pie-chart">
              <div 
                v-for="(item, index) in computedPieData" 
                :key="index" 
                class="pie-segment"
                :style="{
                  transform: `rotate(${item.startAngle}deg)`,
                  zIndex: computedPieData.length - index
                }"
              >
                <div 
                  class="pie-segment-fill"
                  :style="{
                    transform: `rotate(${item.angle}deg)`,
                    backgroundColor: getPieColor(index)
                  }"
                ></div>
              </div>
              <div class="pie-center"></div>
            </div>
              <div class="pie-legend">
                <div v-for="(item, index) in categoryPieData" :key="index" class="legend-item">
                  <span class="legend-color" :style="{background: getPieColor(index)}"></span>
                  <span class="legend-text">{{ item.name }}: {{ item.percentage }}%</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </a-col>
    </a-row>

    <!-- 图书热度排行榜 -->
    <div class="card-container animate-fade-in" style="margin-top: 16px; animation-delay: 0.3s">
      <div class="card-header">
        <h3 class="card-title">
          <FireOutlined /> 图书热度排行榜
        </h3>
      </div>
      <div class="card-body">
        <a-row :gutter="[16, 16]">
          <a-col v-for="(book, index) in hotBooks" :key="book.id" :xs="24" :sm="12" :lg="6">
            <div class="hot-book-card">
              <div class="rank-badge" :class="getRankClass(index)">
                {{ index + 1 }}
              </div>
              <div class="book-cover">
                <img :src="book.cover" :alt="book.title" />
              </div>
              <div class="book-info">
                <h4 class="book-title">{{ book.title }}</h4>
                <p class="book-author">{{ book.author }}</p>
                <div class="book-meta">
                  <span class="borrow-count">
                    <span class="icon">📖</span>
                    借阅 {{ book.borrowCount }} 次
                  </span>
                  <span class="rating">
                    <span class="icon">⭐</span>
                    {{ book.rating }}
                  </span>
                </div>
              </div>
            </div>
          </a-col>
        </a-row>
      </div>
    </div>

    <!-- 读者活跃度分析 -->
    <div class="card-container animate-fade-in" style="margin-top: 16px; animation-delay: 0.4s">
      <div class="card-header">
        <h3 class="card-title">
          <UserOutlined /> 读者活跃度分析
        </h3>
      </div>
      <div class="card-body">
        <a-row :gutter="[16, 16]">
          <a-col :xs="24" :lg="12">
            <div class="activity-card">
              <h4 class="section-title">最活跃读者</h4>
              <div class="reader-list">
                <div v-for="(reader, index) in activeReaders" :key="reader.id" class="reader-item">
                  <div class="reader-rank">{{ index + 1 }}</div>
                  <a-avatar :size="40" :src="reader.avatar" :style="{ backgroundColor: getAvatarColor(reader.id) }">
                    {{ reader.name.charAt(0) }}
                  </a-avatar>
                  <div class="reader-info">
                    <div class="reader-name">{{ reader.name }}</div>
                    <div class="reader-department">{{ reader.department }}</div>
                  </div>
                  <div class="reader-stats">
                    <span class="stat-value">{{ reader.borrowCount }}</span>
                    <span class="stat-label">借阅次数</span>
                  </div>
                </div>
              </div>
            </div>
          </a-col>
          <a-col :xs="24" :lg="12">
            <div class="activity-card">
              <h4 class="section-title">读者类型分布</h4>
              <div class="type-distribution">
                <div v-for="(type, index) in readerTypes" :key="index" class="type-item">
                  <div class="type-info">
                    <span class="type-name">{{ type.name }}</span>
                    <span class="type-count">{{ type.count }} 人</span>
                  </div>
                  <div class="type-bar">
                    <div 
                      class="type-bar-fill" 
                      :style="{ 
                        width: `${(type.count / maxReaderCount) * 100}%`,
                        backgroundColor: getTypeColor(index)
                      }"
                    ></div>
                  </div>
                </div>
              </div>
            </div>
          </a-col>
        </a-row>
      </div>
    </div>

    <!-- 统计概览卡片 -->
    <a-row :gutter="[16, 16]" class="stat-row" style="margin-top: 16px">
      <a-col :xs="24" :sm="12" :lg="6">
        <div class="stat-card animate-slide-up" style="animation-delay: 0.5s">
          <div class="stat-icon pulse-animation">
            <BookOutlined />
          </div>
          <div class="stat-info">
            <div class="stat-value count-up">{{ totalBooks }}</div>
            <div class="stat-label">图书总数</div>
          </div>
          <div class="stat-card-bg"></div>
        </div>
      </a-col>
      <a-col :xs="24" :sm="12" :lg="6">
        <div class="stat-card success animate-slide-up" style="animation-delay: 0.6s">
          <div class="stat-icon pulse-animation">
            <UserOutlined />
          </div>
          <div class="stat-info">
            <div class="stat-value count-up">{{ totalReaders }}</div>
            <div class="stat-label">读者总数</div>
          </div>
          <div class="stat-card-bg"></div>
        </div>
      </a-col>
      <a-col :xs="24" :sm="12" :lg="6">
        <div class="stat-card warning animate-slide-up" style="animation-delay: 0.7s">
          <div class="stat-icon pulse-animation">
            <SwapOutlined />
          </div>
          <div class="stat-info">
            <div class="stat-value count-up">{{ totalBorrowed }}</div>
            <div class="stat-label">借阅总量</div>
          </div>
          <div class="stat-card-bg"></div>
        </div>
      </a-col>
      <a-col :xs="24" :sm="12" :lg="6">
        <div class="stat-card error animate-slide-up" style="animation-delay: 0.8s">
          <div class="stat-icon pulse-animation">
            <LikeOutlined />
          </div>
          <div class="stat-info">
            <div class="stat-value count-up">{{ avgRating }}</div>
            <div class="stat-label">平均评分</div>
          </div>
          <div class="stat-card-bg"></div>
        </div>
      </a-col>
    </a-row>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { message } from 'ant-design-vue'
import {
  BookOutlined,
  UserOutlined,
  SwapOutlined,
  DownloadOutlined,
  LineChartOutlined,
  PieChartOutlined,
  FireOutlined,
  LikeOutlined
} from '@ant-design/icons-vue'

// 导入本地封面图片
import hlmCover from '@/views/img/hlm.webp'
import jsCover from '@/views/img/js.webp'
import sgyyCover from '@/views/img/sgyy.webp'
import vueCover from '@/views/img/vue.jpeg'
import sjCover from '@/views/img/sj.webp'
import jjxCover from '@/views/img/jjx.webp'
import xlxCover from '@/views/img/xlx.webp'
import sxCover from '@/views/img/sx.webp'

const DEFAULT_COVERS = [
  hlmCover,
  jsCover,
  sgyyCover,
  vueCover,
  sjCover,
  jjxCover,
  xlxCover,
  sxCover
]

// 响应式数据
const selectedPeriod = ref('month')

// 模拟数据
const borrowTrendData = ref([
  { label: '1月', value: 128 },
  { label: '2月', value: 145 },
  { label: '3月', value: 168 },
  { label: '4月', value: 156 },
  { label: '5月', value: 189 },
  { label: '6月', value: 215 },
  { label: '7月', value: 198 },
  { label: '8月', value: 225 },
  { label: '9月', value: 246 },
  { label: '10月', value: 278 },
  { label: '11月', value: 256 },
  { label: '12月', value: 298 }
])

const categoryPieData = ref([
  { name: '文学小说', value: 35, percentage: 35, angle: 126 },
  { name: '科技编程', value: 25, percentage: 25, angle: 90 },
  { name: '历史传记', value: 18, percentage: 18, angle: 64.8 },
  { name: '经济管理', value: 12, percentage: 12, angle: 43.2 },
  { name: '艺术设计', value: 10, percentage: 10, angle: 36 }
])

// 计算饼图起始角度
const computedPieData = computed(() => {
  let startAngle = 0
  return categoryPieData.value.map(item => {
    const currentAngle = startAngle
    startAngle += item.angle
    return {
      ...item,
      startAngle: currentAngle
    }
  })
})

const hotBooks = ref([
  { id: 1, title: '红楼梦', author: '曹雪芹', borrowCount: 156, rating: 4.9, cover: DEFAULT_COVERS[0] },
  { id: 2, title: 'JavaScript高级编程', author: 'Nicholas C. Zakas', borrowCount: 142, rating: 4.8, cover: DEFAULT_COVERS[1] },
  { id: 3, title: '三国演义', author: '罗贯中', borrowCount: 138, rating: 4.7, cover: DEFAULT_COVERS[2] },
  { id: 4, title: 'Vue.js实战', author: '梁灏', borrowCount: 125, rating: 4.6, cover: DEFAULT_COVERS[3] },
  { id: 5, title: '史记', author: '司马迁', borrowCount: 118, rating: 4.8, cover: DEFAULT_COVERS[4] },
  { id: 6, title: '经济学原理', author: '曼昆', borrowCount: 105, rating: 4.5, cover: DEFAULT_COVERS[5] },
  { id: 7, title: '西游记', author: '吴承恩', borrowCount: 98, rating: 4.7, cover: DEFAULT_COVERS[6] },
  { id: 8, title: '设计心理学', author: '唐纳德·诺曼', borrowCount: 89, rating: 4.6, cover: DEFAULT_COVERS[7] }
])

const activeReaders = ref([
  { id: 1, name: '张三', department: '计算机学院', borrowCount: 45, avatar: '' },
  { id: 2, name: '李四', department: '文学院', borrowCount: 38, avatar: '' },
  { id: 3, name: '王五', department: '历史学院', borrowCount: 32, avatar: '' },
  { id: 4, name: '赵六', department: '经济学院', borrowCount: 28, avatar: '' },
  { id: 5, name: '钱七', department: '艺术学院', borrowCount: 25, avatar: '' }
])

const readerTypes = ref([
  { name: '本科生', count: 256 },
  { name: '研究生', count: 128 },
  { name: '教师', count: 45 },
  { name: '职工', count: 32 }
])

// 计算属性
const maxBorrowValue = computed(() => {
  return Math.max(...borrowTrendData.value.map(item => item.value))
})

const maxReaderCount = computed(() => {
  return Math.max(...readerTypes.value.map(item => item.count))
})

const totalBooks = computed(() => 1258)
const totalReaders = computed(() => 461)
const totalBorrowed = computed(() => 2589)
const avgRating = computed(() => 4.7)

// 方法
function getBarColor(index) {
  const colors = ['#1890ff', '#52c41a', '#faad14', '#722ed1', '#eb2f96', '#13c2c2', '#fa541c', '#2f54eb']
  return colors[index % colors.length]
}

function getPieColor(index) {
  const colors = ['#1890ff', '#52c41a', '#faad14', '#722ed1', '#eb2f96', '#13c2c2', '#fa541c', '#2f54eb']
  return colors[index % colors.length]
}

function getTypeColor(index) {
  const colors = ['#1890ff', '#52c41a', '#faad14', '#722ed1']
  return colors[index % colors.length]
}

function getRankClass(index) {
  if (index === 0) return 'first'
  if (index === 1) return 'second'
  if (index === 2) return 'third'
  return 'normal'
}

function getAvatarColor(id) {
  const colors = ['#1890ff', '#52c41a', '#faad14', '#722ed1', '#eb2f96', '#13c2c2']
  return colors[(id - 1) % colors.length]
}

function handleExport() {
  message.success('报表导出功能正在开发中...')
}
</script>

<style lang="less" scoped>
// ========================================
// 动画定义
// ========================================
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes slideDown {
  from {
    opacity: 0;
    transform: translateY(-20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.1); }
}

@keyframes progressGrow {
  from { width: 0; }
}

// ========================================
// 动画类
// ========================================
.animate-fade-in {
  animation: fadeIn 0.5s ease-out both;
}

.animate-slide-up {
  animation: slideUp 0.5s ease-out both;
}

.animate-slide-down {
  animation: slideDown 0.5s ease-out both;
}

// ========================================
// 主样式
// ========================================
.statistics {
  .page-title {
    font-size: 20px;
    font-weight: 600;
    color: #1a1a1a;
    margin-bottom: 24px;
    position: relative;
    display: inline-block;

    &::after {
      content: '';
      position: absolute;
      bottom: -4px;
      left: 0;
      width: 0;
      height: 3px;
      background: linear-gradient(90deg, #1890ff, #40a9ff);
      border-radius: 2px;
      animation: expandWidth 0.8s ease-out 0.3s forwards;
    }
  }
}

@keyframes expandWidth {
  to { width: 100%; }
}

.time-selector {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
  background: #fff;
  padding: 16px 20px;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);

  .export-btn {
    transition: all 0.3s ease;
    &:hover {
      transform: translateY(-2px);
      box-shadow: 0 4px 12px rgba(24, 144, 255, 0.4);
    }
  }
}

.chart-row {
  margin-bottom: 16px;
}

.card-container {
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);
  transition: all 0.3s ease;
  border: 1px solid #f0f0f0;

  &:hover {
    box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
    border-color: #e0e0e0;
  }

  &.equal-height {
    height: 100%;
    min-height: 340px;
    display: flex;
    flex-direction: column;
  }

  .card-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 16px 20px;
    border-bottom: 1px solid #f0f0f0;
    margin-bottom: 16px;

    .card-title {
      font-size: 16px;
      font-weight: 600;
      color: #1a1a1a;
      margin: 0;
      display: flex;
      align-items: center;
      gap: 8px;
    }
  }

  .card-body {
    padding: 16px 20px;
    flex: 1;
    overflow: auto;
  }
}

.chart-container {
  .chart-placeholder {
    height: 280px;
    display: flex;
    align-items: flex-end;
    justify-content: space-around;
    padding: 20px;
    position: relative;

    .chart-bars {
      display: flex;
      align-items: flex-end;
      justify-content: space-around;
      width: 100%;
      height: 220px;
      border-left: 2px solid #e8e8e8;
      border-bottom: 2px solid #e8e8e8;
      padding: 0 20px 20px 20px;

      .bar-item {
        display: flex;
        flex-direction: column;
        align-items: center;
        width: 8%;

        .bar-label {
          font-size: 12px;
          color: #999;
          margin-bottom: 8px;
        }

        .bar-wrapper {
          width: 100%;
          height: 180px;
          display: flex;
          align-items: flex-end;
          justify-content: center;

          .bar-fill {
            width: 80%;
            border-radius: 4px 4px 0 0;
            transition: all 0.5s ease;
            animation: progressGrow 1s ease-out both;
          }
        }

        .bar-value {
          font-size: 12px;
          color: #666;
          margin-top: 8px;
          font-weight: 500;
        }
      }
    }

    .chart-legend {
      position: absolute;
      top: 10px;
      right: 20px;
      display: flex;
      gap: 16px;

      .legend-item {
        display: flex;
        align-items: center;
        gap: 6px;
        font-size: 12px;
        color: #666;

        .legend-color {
          width: 12px;
          height: 12px;
          border-radius: 2px;
        }
      }
    }
  }

  .pie-chart-placeholder {
    height: 280px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 20px;

    .pie-chart {
      width: 150px;
      height: 150px;
      border-radius: 50%;
      position: relative;
      margin-bottom: 20px;

      .pie-segment {
        position: absolute;
        width: 100%;
        height: 100%;
        border-radius: 50%;
        clip: rect(0, 150px, 150px, 75px);

        .pie-segment-fill {
          position: absolute;
          width: 100%;
          height: 100%;
          border-radius: 50%;
          background-color: inherit;
          clip: rect(0, 75px, 150px, 0);
          transform-origin: 50% 50%;
        }
      }

      .pie-center {
        position: absolute;
        top: 50%;
        left: 50%;
        width: 60%;
        height: 60%;
        background-color: #fff;
        border-radius: 50%;
        transform: translate(-50%, -50%);
      }
    }

    .pie-legend {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 8px;
      padding: 10px;
      max-width: 100%;

      .legend-item {
        display: flex;
        align-items: center;
        gap: 6px;
        font-size: 11px;
        min-width: 0;

        .legend-color {
          width: 10px;
          height: 10px;
          border-radius: 2px;
          flex-shrink: 0;
        }

        .legend-text {
          color: #666;
          white-space: nowrap;
          overflow: hidden;
          text-overflow: ellipsis;
          min-width: 0;
        }
      }
    }
  }
}

.hot-book-card {
  background: #fff;
  border-radius: 12px;
  overflow: hidden;
  border: 1px solid #f0f0f0;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  transition: all 0.3s ease;
  position: relative;

  &:hover {
    box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
    border-color: #1890ff;
    transform: translateY(-4px);
  }

  .rank-badge {
    position: absolute;
    top: 10px;
    left: 10px;
    width: 32px;
    height: 32px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: bold;
    color: #fff;
    z-index: 1;

    &.first {
      background: linear-gradient(135deg, #ffd700, #ffb800);
      box-shadow: 0 2px 8px rgba(255, 215, 0, 0.4);
    }

    &.second {
      background: linear-gradient(135deg, #c0c0c0, #a8a8a8);
      box-shadow: 0 2px 8px rgba(192, 192, 192, 0.4);
    }

    &.third {
      background: linear-gradient(135deg, #cd7f32, #b87333);
      box-shadow: 0 2px 8px rgba(205, 127, 50, 0.4);
    }

    &.normal {
      background: #1890ff;
      box-shadow: 0 2px 8px rgba(24, 144, 255, 0.3);
    }
  }

  .book-cover {
    height: 180px;
    overflow: hidden;
    background: #f5f5f5;

    img {
      width: 100%;
      height: 100%;
      object-fit: cover;
    }
  }

  .book-info {
    padding: 16px;

    .book-title {
      font-size: 14px;
      font-weight: 600;
      color: #1a1a1a;
      margin: 0 0 8px 0;
      display: -webkit-box;
      -webkit-line-clamp: 2;
      -webkit-box-orient: vertical;
      overflow: hidden;
      line-height: 1.5;
      min-height: 42px;
      word-break: break-all;
    }

    .book-author {
      font-size: 13px;
      color: #666;
      margin: 0 0 12px 0;
    }

    .book-meta {
      display: flex;
      justify-content: space-between;
      align-items: center;

      .borrow-count, .rating {
        display: flex;
        align-items: center;
        gap: 4px;
        font-size: 12px;
        color: #999;

        .icon {
          font-size: 14px;
        }
      }

      .rating {
        color: #faad14;
        font-weight: 500;
      }
    }
  }
}

.activity-card {
  .section-title {
    font-size: 14px;
    font-weight: 600;
    color: #1a1a1a;
    margin-bottom: 16px;
    padding-bottom: 8px;
    border-bottom: 2px solid #f0f0f0;
  }

  .reader-list {
    .reader-item {
      display: flex;
      align-items: center;
      padding: 12px 0;
      border-bottom: 1px solid #f5f5f5;
      transition: all 0.3s ease;

      &:last-child {
        border-bottom: none;
      }

      &:hover {
        background-color: #fafafa;
        border-radius: 8px;
        padding-left: 8px;
        padding-right: 8px;
      }

      .reader-rank {
        width: 24px;
        height: 24px;
        border-radius: 50%;
        background: #1890ff;
        color: #fff;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 12px;
        font-weight: bold;
        margin-right: 12px;
      }

      .reader-info {
        flex: 1;
        margin-left: 12px;

        .reader-name {
          font-weight: 500;
          color: #1a1a1a;
          font-size: 14px;
        }

        .reader-department {
          font-size: 12px;
          color: #999;
        }
      }

      .reader-stats {
        text-align: right;

        .stat-value {
          font-size: 18px;
          font-weight: 600;
          color: #1890ff;
        }

        .stat-label {
          font-size: 12px;
          color: #999;
        }
      }
    }
  }

  .type-distribution {
    .type-item {
      padding: 12px 0;

      .type-info {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 8px;

        .type-name {
          font-weight: 500;
          color: #1a1a1a;
        }

        .type-count {
          font-size: 14px;
          color: #666;
        }
      }

      .type-bar {
        height: 8px;
        background: #f0f0f0;
        border-radius: 4px;
        overflow: hidden;

        .type-bar-fill {
          height: 100%;
          border-radius: 4px;
          transition: all 0.5s ease;
          animation: progressGrow 1s ease-out both;
        }
      }
    }
  }
}

.stat-row {
  .stat-card {
    background: linear-gradient(135deg, #1890ff 0%, #40a9ff 100%);
    border-radius: 12px;
    padding: 24px;
    display: flex;
    align-items: center;
    color: #fff;
    box-shadow: 0 2px 8px rgba(24, 144, 255, 0.15);
    position: relative;
    overflow: hidden;
    cursor: pointer;
    transition: all 0.3s ease;

    &:hover {
      transform: translateY(-6px) scale(1.02);
      box-shadow: 0 12px 32px rgba(24, 144, 255, 0.4);
    }

    &.success {
      background: linear-gradient(135deg, #52c41a 0%, #73d13d 100%);
      box-shadow: 0 2px 8px rgba(82, 196, 26, 0.15);

      &:hover {
        box-shadow: 0 12px 32px rgba(82, 196, 26, 0.4);
      }
    }

    &.warning {
      background: linear-gradient(135deg, #faad14 0%, #ffc53d 100%);
      box-shadow: 0 2px 8px rgba(250, 173, 20, 0.15);

      &:hover {
        box-shadow: 0 12px 32px rgba(250, 173, 20, 0.4);
      }
    }

    &.error {
      background: linear-gradient(135deg, #ff4d4f 0%, #ff7875 100%);
      box-shadow: 0 2px 8px rgba(255, 77, 79, 0.15);

      &:hover {
        box-shadow: 0 12px 32px rgba(255, 77, 79, 0.4);
      }
    }

    .stat-card-bg {
      position: absolute;
      right: -20px;
      bottom: -20px;
      width: 120px;
      height: 120px;
      background: rgba(255, 255, 255, 0.1);
      border-radius: 50%;
      transition: transform 0.3s ease;
    }

    &:hover .stat-card-bg {
      transform: scale(1.1);
    }

    .stat-icon {
      font-size: 40px;
      opacity: 0.9;
      margin-right: 20px;
      z-index: 1;
      transition: transform 0.3s ease;
    }

    &:hover .stat-icon {
      transform: scale(1.1);
    }

    .stat-info {
      z-index: 1;

      .stat-value {
        font-size: 32px;
        font-weight: 700;
        line-height: 1.2;
      }

      .stat-label {
        font-size: 14px;
        opacity: 0.9;
        margin-top: 4px;
      }
    }
  }
}

.pulse-animation {
  animation: pulse 2s ease-in-out infinite;
}
</style>
