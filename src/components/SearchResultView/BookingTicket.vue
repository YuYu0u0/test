<template>
  <!-- 主容器 -->
  <div class="ticket-container">
    <!-- 左側主要區塊 -->
    <div class="left-panel">
      <!-- 1. 航班號 & 航程時間 -->
      <div class="info-row flight-info f5">
        <div class="flight-number">機號 {{ flight.flightNumber }}・機型 {{ flight.aircraftType }}</div>
        <div class="flight-duration">飛行時間: {{ flight.duration }}</div>
      </div>

      <!-- 2. 起飛 & 降落時間 -->
      <div class="info-row time-info">
        <div class="time-block departure-time">
          <span class="date f5">{{ flight.departureDate }}</span>
          <span class="time">{{ flight.departureTime }}</span>
        </div>
        <div class="time-block arrival-time">
          <span class="date f5">{{ flight.arrivalDate }}</span>
          <span class="time">{{ flight.arrivalTime }}</span>
        </div>
      </div>

      <!-- 3. 機場代碼 & 時間軸 -->
      <div class="info-row timeline-info">
        <span class="airport-code f3">{{ flight.originCode }}</span>
        <div class="timeline-line"></div>
        <span class="airport-code f3">{{ flight.destinationCode }}</span>
      </div>
    </div>

    <!-- 右側主要區塊 -->
    <div class="right-panel">
      <!-- 艙等與價格 -->
      <div class="price-section">
        <div class="price-content">
          <div class="class-name">{{ cabinClassDisplay }}</div>
          <div class="price-display">
            <span class="currency f5">TWD</span>
            <span v-if="flight.isSpecialOffer" class="special-offer-tag">特價</span>
            <span class="amount">{{ ticketPrice.toLocaleString() }}</span>
            <span class="suffix f5">起</span>
          </div>
        </div>
      </div>
      <div class="select-seat-section">
        <!-- 選位按鈕 -->
        <router-link :to="{ name: selectSeatRouteName }" @click="selectFlight">
          <button class="select-seat-button">選位</button>
        </router-link>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue';
import { useOrderStore } from '@/stores/order';

const orderStore = useOrderStore();

const props = defineProps({
  flight: {
    type: Object,
    required: true,
    default: () => ({
      id: '',
      flightNumber: '',
      aircraftType: '',
      duration: '',
      departureDate: '',
      departureTime: '',
      arrivalDate: '',
      arrivalTime: '',
      originCode: '',
      destinationCode: '',
      ticketPrice: 0, // 價格從 flight prop 傳入
      isSpecialOffer: false, // 特價標註從 flight prop 傳入
    }),
  },
  cabinClass: {
    type: String,
    required: true,
  },
});

const cabinClassDisplayMap = {
  economy: '經濟艙',
  premiumEconomy: '豪華經濟艙',
  first: '頭等艙',
};

const ticketPrice = computed(() => {
  return props.flight.ticketPrice;
});

const cabinClassDisplay = computed(() => {
  return cabinClassDisplayMap[props.cabinClass] || '未知艙等';
});

const selectSeatRouteName = computed(() => {
  // 將 camelCase 轉換為 kebab-case
  const kebabCaseCabinClass = props.cabinClass.replace(/([a-z0-9]|(?=[A-Z]))([A-Z])/g, '$1-$2').toLowerCase();
  return `select-seat-${kebabCaseCabinClass}`;
});

const selectFlight = () => {
  orderStore.setSelectedFlight({ ...props.flight, cabinClass: props.cabinClass });
};
</script>

<style scoped>
/* 主容器 (修改為 Grid，且等寬) */
.ticket-container {
  width: 100%;
  background-color: var(--color-neutral-form-bg-light-orange);
  display: grid;
  grid-template-columns: 1fr 1fr;
  /* 修改為 1fr 1fr，使左右區塊等寬 */
  box-sizing: border-box;
  border-radius: 8px;
  /* 圓角 */
  overflow: hidden;
  /* 確保圓角生效 */
}

/* 左側區塊 */
.left-panel {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 20px;
  border-right: 1px solid #794425;
  /* 主要垂直分隔線 */
}

.info-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

/* 1. 航班號 & 航程時間 */
.flight-info {
  color: #555;
}

/* 2. 起飛 & 降落時間 */
.time-block .date {
  color: #666;
  display: block;
}

.time-block .time {
  font-size: 28px;
  font-weight: bold;
  color: #333;
}

.arrival-time {
  text-align: right;
}

/* 3. 機場代碼 & 時間軸 */
.timeline-info {
  padding: 0 5px;
}

.airport-code {
  font-weight: bold;
  color: #444;
}

.timeline-line {
  flex: 1;
  height: 1px;
  background-color: #ccc;
  margin: 0 15px;
}

/* 右側區塊 (修改為置中內容) */
.right-panel {
  display: flex;
  align-items: center;
  /* 修改為 center，使內部項目置中 */
  gap: 20px;
  /* 增加項目間的距離 */
}

/* 價格區塊 */
.price-section {
  width: 50%;
  display: flex;
  justify-content: end;
  height: 100%;
  padding: 0 20px;
  /* 調整內距 */
  border-right: 1px solid #ab8e7d;
  /* 右側較淡的垂直分隔線 */
}

.price-content {
  display: flex;
  flex-direction: column;
  justify-content: space-around;
}

.select-seat-section {
  display: flex;
  flex: 1;
  align-items: center;
  justify-content: center;
}

.class-name {
  font-size: 16px;
  font-weight: bold;
  color: #333;
  margin-bottom: 5px;
}

.price-display {
  display: flex;
  align-items: baseline;
  border-bottom: 3px solid #794425;
}

.price-display .currency,
.price-display .suffix {
  font-weight: bold;
  color: #666;
}

.price-display .amount {
  font-size: 32px;
  font-weight: bold;
  color: #d9534f;
  margin: 0 4px;
}

/* 選位按鈕 */
.select-seat-button {
  width: 100px;
  height: 44px;
  background-color: #794425;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  transition: opacity 0.2s;
}

.select-seat-button:hover {
  opacity: 0.9;
}

.special-offer-tag {
  background-color: #d9534f;
  color: white;
  padding: 2px 6px;
  border-radius: 4px;
  font-size: 0.8em;
  margin-left: 8px;
  white-space: nowrap;
}
</style>
