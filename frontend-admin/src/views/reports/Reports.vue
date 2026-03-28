<template>
  <div class="reports">
    <h2 class="page-title animate-fade-in">统计报表</h2>

    <a-row :gutter="[16, 16]" class="stat-row">
      <a-col :xs="24" :sm="12" :lg="6">
        <div class="stat-card animate-slide-up" style="animation-delay: 0.1s">
          <div class="stat-icon">
            <BarChartOutlined />
          </div>
          <div class="stat-info">
            <div class="stat-value">{{ totalBorrowCount }}</div>
            <div class="stat-label">年度借阅总量</div>
          </div>
          <div class="stat-card-bg"></div>
        </div>
      </a-col>
      <a-col :xs="24" :sm="12" :lg="6">
        <div class="stat-card success animate-slide-up" style="animation-delay: 0.2s">
          <div class="stat-icon">
            <BookOutlined />
          </div>
          <div class="stat-info">
            <div class="stat-value">{{ bookHotRanking.length }}</div>
            <div class="stat-label">热门图书</div>
          </div>
          <div class="stat-card-bg"></div>
        </div>
      </a-col>
      <a-col :xs="24" :sm="12" :lg="6">
        <div class="stat-card warning animate-slide-up" style="animation-delay: 0.3s">
          <div class="stat-icon">
            <UserOutlined />
          </div>
          <div class="stat-info">
            <div class="stat-value">{{ readerActivityData.length }}</div>
            <div class="stat-label">活跃读者</div>
          </div>
          <div class="stat-card-bg"></div>
        </div>
      </a-col>
      <a-col :xs="24" :sm="12" :lg="6">
        <div class="stat-card error animate-slide-up" style="animation-delay: 0.4s">
          <div class="stat-icon">
            <FileTextOutlined />
          </div>
          <div class="stat-info">
            <div class="stat-value">3</div>
            <div class="stat-label">报表类型</div>
          </div>
          <div class="stat-card-bg"></div>
        </div>
      </a-col>
    </a-row>

    <a-row :gutter="[16, 16]">
      <a-col :xs="24" :lg="16">
        <div class="card-container equal-height animate-fade-in" style="animation-delay: 0.5s">
          <div class="card-header">
            <h3 class="card-title">
              <BarChartOutlined /> 借阅统计报表
            </h3>
            <a-space>
              <a-radio-group v-model:value="reportPeriod" button-style="solid" size="small">
                <a-radio-button value="monthly">月度</a-radio-button>
                <a-radio-button value="quarterly">季度</a-radio-button>
                <a-radio-button value="yearly">年度</a-radio-button>
              </a-radio-group>
              <a-button type="primary" size="small" @click="exportBorrowReport">
                <DownloadOutlined /> 导出
              </a-button>
            </a-space>
          </div>
          <div class="card-body">
            <div class="chart-container">
              <div class="chart-bars">
                <div
                  v-for="(item, index) in currentBorrowData"
                  :key="getLabel(item)"
                  class="chart-bar-group"
                >
                  <div class="chart-bar-wrapper">
                    <div
                      class="chart-bar borrow-bar"
                      :style="{ height: `${(item.borrowCount / maxBorrowCount) * 100}%` }"
                    >
                      <span class="bar-value">{{ item.borrowCount }}</span>
                    </div>
                    <div
                      class="chart-bar return-bar"
                      :style="{ height: `${(item.returnCount / maxBorrowCount) * 100}%` }"
                    >
                      <span class="bar-value">{{ item.returnCount }}</span>
                    </div>
                  </div>
                  <div class="bar-label">{{ getLabel(item) }}</div>
                </div>
              </div>
              <div class="chart-legend">
                <div class="legend-item">
                  <span class="legend-color borrow-color"></span>
                  <span class="legend-text">借阅量</span>
                </div>
                <div class="legend-item">
                  <span class="legend-color return-color"></span>
                  <span class="legend-text">归还量</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </a-col>

      <a-col :xs="24" :lg="8">
        <div class="card-container equal-height animate-fade-in" style="animation-delay: 0.6s">
          <div class="card-header">
            <h3 class="card-title">
              <FireOutlined /> 图书热度排行榜
            </h3>
            <a-button type="link" size="small" @click="exportBookRanking">
              <DownloadOutlined /> 导出
            </a-button>
          </div>
          <div class="card-body ranking-body">
            <div class="ranking-list">
              <div
                v-for="(book, index) in bookHotRanking"
                :key="book.rank"
                class="ranking-item hover-highlight"
              >
                <div class="ranking-left">
                  <div
                    class="ranking-number"
                    :class="{
                      'top-1': book.rank === 1,
                      'top-2': book.rank === 2,
                      'top-3': book.rank === 3
                    }"
                  >
                    {{ book.rank }}
                  </div>
                  <div class="ranking-cover">
                    <img :src="book.cover" :alt="book.title" />
                  </div>
                  <div class="ranking-info">
                    <div class="ranking-title">{{ book.title }}</div>
                    <div class="ranking-author">{{ book.author }}</div>
                  </div>
                </div>
                <div class="ranking-right">
                  <span class="ranking-count">{{ book.borrowCount }}</span>
                  <span class="ranking-unit">次</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </a-col>
    </a-row>

    <div class="card-container animate-fade-in" style="margin-top: 16px; animation-delay: 0.7s">
      <div class="card-header">
        <h3 class="card-title">
          <TeamOutlined /> 读者活跃度分析
        </h3>
        <a-button type="link" size="small" @click="exportReaderActivity">
          <DownloadOutlined /> 导出
        </a-button>
      </div>
      <div class="card-body">
        <a-row :gutter="[16, 16]">
          <a-col
            v-for="(reader, index) in readerActivityData"
            :key="reader.rank"
            :xs="24"
            :sm="12"
            :lg="6"
          >
            <div class="reader-card hover-lift">
              <div class="reader-header">
                <div
                  class="reader-rank"
                  :class="{
                    'top-1': reader.rank === 1,
                    'top-2': reader.rank === 2,
                    'top-3': reader.rank === 3
                  }"
                >
                  {{ reader.rank }}
                </div>
                <a-avatar
                  :style="{ backgroundColor: getAvatarColor(reader.rank) }"
                  size="large"
                >
                  {{ reader.name.charAt(0) }}
                </a-avatar>
              </div>
              <div class="reader-info">
                <div class="reader-name">{{ reader.name }}</div>
                <div class="reader-department">{{ reader.department }}</div>
                <a-tag :color="reader.type === '教师' ? 'blue' : 'green'" size="small" class="reader-type">
                  {{ reader.type }}
                </a-tag>
              </div>
              <div class="reader-stats">
                <div class="stat-number">{{ reader.borrowCount }}</div>
                <div class="stat-label">借阅次数</div>
              </div>
            </div>
          </a-col>
        </a-row>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { message } from 'ant-design-vue'
import {
  BarChartOutlined,
  BookOutlined,
  UserOutlined,
  FileTextOutlined,
  FireOutlined,
  TeamOutlined,
  DownloadOutlined
} from '@ant-design/icons-vue'
import {
  monthlyBorrowData,
  quarterlyBorrowData,
  yearlyBorrowData,
  bookHotRanking,
  readerActivityData
} from '@/data/mockData'

const reportPeriod = ref('monthly')

const currentBorrowData = computed(() => {
  switch (reportPeriod.value) {
    case 'monthly':
      return monthlyBorrowData
    case 'quarterly':
      return quarterlyBorrowData
    case 'yearly':
      return yearlyBorrowData
    default:
      return monthlyBorrowData
  }
})

const maxBorrowCount = computed(() => {
  const counts = currentBorrowData.value.flatMap(item => [item.borrowCount, item.returnCount])
  return Math.max(...counts, 1)
})

const totalBorrowCount = computed(() => {
  return yearlyBorrowData.find(d => d.year === '2024')?.borrowCount || 0
})

function getLabel(item) {
  return item.month || item.quarter || item.year || ''
}

function getAvatarColor(id) {
  const colors = ['#1890ff', '#52c41a', '#faad14', '#722ed1', '#eb2f96', '#13c2c2', '#fa541c', '#2f54eb']
  return colors[(id - 1) % colors.length]
}

function exportBorrowReport() {
  const data = currentBorrowData.value
  const headers = reportPeriod.value === 'monthly'
    ? ['月份', '借阅量', '归还量']
    : reportPeriod.value === 'quarterly'
    ? ['季度', '借阅量', '归还量']
    : ['年份', '借阅量', '归还量']
  
  const rows = data.map(item => [
    getLabel(item),
    item.borrowCount,
    item.returnCount
  ])
  
  exportToCSV(headers, rows, `借阅报表_${reportPeriod.value}_${new Date().toISOString().slice(0, 10)}.csv`)
}

function exportBookRanking() {
  const headers = ['排名', '书名', '作者', '借阅次数']
  const rows = bookHotRanking.map(book => [
    book.rank,
    book.title,
    book.author,
    book.borrowCount
  ])
  
  exportToCSV(headers, rows, `图书热度排行榜_${new Date().toISOString().slice(0, 10)}.csv`)
}

function exportReaderActivity() {
  const headers = ['排名', '姓名', '部门', '类型', '借阅次数']
  const rows = readerActivityData.map(reader => [
    reader.rank,
    reader.name,
    reader.department,
    reader.type,
    reader.borrowCount
  ])
  
  exportToCSV(headers, rows, `读者活跃度分析_${new Date().toISOString().slice(0, 10)}.csv`)
}

function exportToCSV(headers, rows, filename) {
  const csvContent = [
    headers.join(','),
    ...rows.map(row => row.join(','))
  ].join('\n')
  
  const blob = new Blob(['\ufeff' + csvContent], { type: 'text/csv;charset=utf-8;' })
  const link = document.createElement('a')
  const url = URL.createObjectURL(blob)
  link.setAttribute('href', url)
  link.setAttribute('download', filename)
  link.style.visibility = 'hidden'
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
  URL.revokeObjectURL(url)
  
  message.success('导出成功！')
}
</script>

<style lang="less" scoped>
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

.animate-fade-in {
  animation: fadeIn 0.5s ease-out both;
}

.animate-slide-up {
  animation: slideUp 0.5s ease-out both;
}

.hover-lift {
  transition: all 0.3s ease;

  &:hover {
    transform: translateY(-4px);
    box-shadow: 0 8px 24px rgba(0, 0, 0, 0.12);
  }
}

.hover-highlight {
  transition: background-color 0.3s ease;

  &:hover {
    background-color: #fafafa;
  }
}

.reports {
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

.stat-row {
  margin-bottom: 16px;
}

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
  height: 280px;
  display: flex;
  flex-direction: column;

  .chart-bars {
    flex: 1;
    display: flex;
    align-items: flex-end;
    justify-content: space-around;
    padding-bottom: 20px;
    gap: 8px;
  }

  .chart-bar-group {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    height: 100%;
  }

  .chart-bar-wrapper {
    flex: 1;
    display: flex;
    align-items: flex-end;
    justify-content: center;
    gap: 4px;
    width: 100%;
  }

  .chart-bar {
    width: 40%;
    border-radius: 4px 4px 0 0;
    position: relative;
    transition: all 0.3s ease;

    &:hover {
      filter: brightness(1.1);
    }

    .bar-value {
      position: absolute;
      top: -20px;
      left: 50%;
      transform: translateX(-50%);
      font-size: 11px;
      color: #666;
      white-space: nowrap;
    }
  }

  .borrow-bar {
    background: linear-gradient(180deg, #1890ff 0%, #40a9ff 100%);
  }

  .return-bar {
    background: linear-gradient(180deg, #52c41a 0%, #73d13d 100%);
  }

  .bar-label {
    font-size: 12px;
    color: #666;
    margin-top: 8px;
    text-align: center;
  }

  .chart-legend {
    display: flex;
    justify-content: center;
    gap: 24px;
    padding-top: 16px;
    border-top: 1px solid #f0f0f0;

    .legend-item {
      display: flex;
      align-items: center;
      gap: 8px;
    }

    .legend-color {
      width: 16px;
      height: 16px;
      border-radius: 4px;
    }

    .borrow-color {
      background: #1890ff;
    }

    .return-color {
      background: #52c41a;
    }

    .legend-text {
      font-size: 12px;
      color: #666;
    }
  }
}

.ranking-body {
  padding: 8px 20px !important;
}

.ranking-list {
  .ranking-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 12px 0;
    border-bottom: 1px solid #f5f5f5;

    &:last-child {
      border-bottom: none;
    }

    .ranking-left {
      display: flex;
      align-items: center;
      gap: 12px;
      flex: 1;
      min-width: 0;
    }

    .ranking-number {
      width: 28px;
      height: 28px;
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 14px;
      font-weight: 600;
      background: #f0f0f0;
      color: #999;
      flex-shrink: 0;

      &.top-1 {
        background: linear-gradient(135deg, #ffd700 0%, #ffed4e 100%);
        color: #8b6914;
      }

      &.top-2 {
        background: linear-gradient(135deg, #c0c0c0 0%, #e8e8e8 100%);
        color: #666;
      }

      &.top-3 {
        background: linear-gradient(135deg, #cd7f32 0%, #e8a862 100%);
        color: #fff;
      }
    }

    .ranking-cover {
      width: 40px;
      height: 56px;
      border-radius: 4px;
      overflow: hidden;
      flex-shrink: 0;
      background: #f5f5f5;

      img {
        width: 100%;
        height: 100%;
        object-fit: cover;
      }
    }

    .ranking-info {
      flex: 1;
      min-width: 0;

      .ranking-title {
        font-size: 14px;
        font-weight: 500;
        color: #1a1a1a;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
      }

      .ranking-author {
        font-size: 12px;
        color: #999;
        margin-top: 2px;
      }
    }

    .ranking-right {
      display: flex;
      align-items: baseline;
      gap: 2px;
      flex-shrink: 0;

      .ranking-count {
        font-size: 18px;
        font-weight: 600;
        color: #1890ff;
      }

      .ranking-unit {
        font-size: 12px;
        color: #999;
      }
    }
  }
}

.reader-card {
  background: linear-gradient(135deg, #fafbfc 0%, #f5f7fa 100%);
  border-radius: 12px;
  padding: 20px;
  text-align: center;
  border: 1px solid #e8e8e8;

  .reader-header {
    position: relative;
    margin-bottom: 12px;
    display: flex;
    justify-content: center;

    .reader-rank {
      position: absolute;
      top: -8px;
      left: -8px;
      width: 28px;
      height: 28px;
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 14px;
      font-weight: 600;
      background: #f0f0f0;
      color: #999;
      z-index: 1;

      &.top-1 {
        background: linear-gradient(135deg, #ffd700 0%, #ffed4e 100%);
        color: #8b6914;
      }

      &.top-2 {
        background: linear-gradient(135deg, #c0c0c0 0%, #e8e8e8 100%);
        color: #666;
      }

      &.top-3 {
        background: linear-gradient(135deg, #cd7f32 0%, #e8a862 100%);
        color: #fff;
      }
    }
  }

  .reader-info {
    margin-bottom: 16px;

    .reader-name {
      font-size: 16px;
      font-weight: 600;
      color: #1a1a1a;
      margin-bottom: 4px;
    }

    .reader-department {
      font-size: 12px;
      color: #999;
      margin-bottom: 8px;
    }

    .reader-type {
      margin: 0;
    }
  }

  .reader-stats {
    .stat-number {
      font-size: 28px;
      font-weight: 700;
      color: #1890ff;
      line-height: 1.2;
    }

    .stat-label {
      font-size: 12px;
      color: #999;
      margin-top: 2px;
    }
  }
}
</style>
