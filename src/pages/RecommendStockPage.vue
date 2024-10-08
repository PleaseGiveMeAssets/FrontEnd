<template>
  <div class="recommendations-page">
    <!-- 월, 일 선택 박스 -->
    <div class="unit-selector">
      <div
        :class="selectedUnit === 'day' ? 'selected-unit' : 'unit'"
        @click="selectedUnit = 'day'"
      >
        일
      </div>
      <div
        :class="selectedUnit === 'month' ? 'selected-unit' : 'unit'"
        @click="selectedUnit = 'month'"
      >
        월
      </div>
    </div>

    <!-- 날짜 선택 부분 -->
    <div class="date-picker">
      <!-- 일 선택기 -->
      <input v-if="selectedUnit === 'day'" v-model="currentDate" type="date" />
      <!-- 월 선택기 -->
      <input
        v-if="selectedUnit === 'month'"
        v-model="currentMonth"
        type="month"
      />
    </div>

    <!-- 종목 리스트 -->
    <div class="recommendations">
      <div
        v-for="(stock, sIndex) in recommendStock"
        :key="sIndex"
        class="stock-group"
      >
        <div
          v-for="(rs, rsIndex) in stock.dailyRecommendStockDTOList"
          :key="rsIndex"
          class="stock-item"
        >
          <div class="stock-day">{{ rs.day }}일</div>
          <!-- 날짜 부분 -->
          <div class="stock-details">
            <div class="stock-info">
              <div class="stock-name">{{ stock.stockName }}</div>
              <div class="short-code">{{ stock.shortCode }}</div>
            </div>
            <div class="stock-price-info">
              <div class="stock-price">{{ rs.price }}원</div>
              <div
                :class="
                  rs.changeAmount > 0 ? 'up' : rs.changeAmount < 0 ? 'down' : ''
                "
              >
                {{ rs.changeAmount > 0 ? "▲" : "▼"
                }}{{ Math.abs(rs.changeAmount) }}원
              </div>
            </div>
            <div
              :class="
                rs.changeAmountRate > 0
                  ? 'stock-change-rate up'
                  : rs.changeAmountRate < 0
                    ? 'stock-change-rate down'
                    : 'stock-change-rate'
              "
            >
              {{ rs.changeAmountRate }}%
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import axios from "axios";
import { onMounted, reactive, ref, watch } from "vue";

// 기본 URL과 종목 추천 데이터 설정
const BASE = `${import.meta.env.VITE_API_URL}/dailyrecommend`;
const recommendStock = reactive([]);
const token = localStorage.getItem("token");

// 날짜 정보와 선택 단위
const currentDate = ref(new Date().toISOString().slice(0, 10)); // 기본 현재 날짜
const currentMonth = ref(new Date().toISOString().slice(0, 7)); // 기본 현재 월
const selectedUnit = ref("day"); // 기본 선택 단위는 '일'

// 종목 추천 데이터 불러오기
const createRecommendStock = async (date) => {
  try {
    const response = await axios.get(BASE + `/${date}`, {
      headers: { Authorization: `Bearer ${token}` },
    });
    Object.assign(recommendStock, response.data); // 데이터 갱신
  } catch (err) {
    console.error("createRecommendStock 에러: ", err.message);
  }
};

// 선택된 단위에 따라 API 호출
watch(selectedUnit, (newUnit) => {
  createRecommendStock(
    newUnit === "day" ? currentDate.value : currentMonth.value,
  );
});

// 초기 API 호출
onMounted(() => createRecommendStock(currentDate.value));
</script>

<style scoped>
.recommendations-page {
  padding-top: 100px;
  font-family: Arial, sans-serif;
}

.unit-selector {
  display: flex;
  justify-content: space-between; /* 좌우 끝에 배치 */
  align-items: center;
  width: 100%; /* 전체 화면을 차지하게 설정 */
  padding: 0 20px; /* 좌우 여백 추가 */
  margin-bottom: 20px;
}

.unit,
.selected-unit {
  font-size: 18px;
  padding: 10px 0; /* 상하 여백만 추가 */
  width: 50%; /* 좌우 버튼이 동일한 넓이를 차지하게 설정 */
  text-align: center; /* 가운데 정렬 */
  cursor: pointer;
  border-bottom: 2px solid transparent;
  transition: all 0.3s ease;
}

.selected-unit {
  color: black;
  font-weight: bold;
  border-bottom: 2px solid #6e2ff4; /* 선택된 항목 강조 */
}

.unit {
  color: gray;
}

.date-picker {
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
}

.stock-group {
  margin-bottom: 20px;
}

.stock-item {
  display: flex;
  flex-direction: column; /* 날짜는 위에, 나머지는 아래로 정렬 */
  margin-bottom: 10px;
}

.stock-day {
  font-size: 16px;
  font-weight: bold;
  color: #333;
  padding-top: 20px;
  margin-bottom: 5px;
}

.stock-details {
  display: flex;
  justify-content: space-between; /* 나머지 정보는 수평으로 정렬 */
  width: 100%;
}

.stock-info {
  flex: 1;
  text-align: left;
}

.stock-price-info {
  flex: 1;
  text-align: right;
  margin-right: 20px;
}

.stock-price {
  color: #333;
  font-weight: bold;
}

.up {
  color: red;
}

.down {
  color: blue;
}

.stock-change-rate {
  font-weight: bold;
  padding: 10px;
  border-radius: 8px;
  background-color: gray;
  color: white;
}

.stock-change-rate.up {
  background-color: red;
}

.stock-change-rate.down {
  background-color: blue;
}

.short-code {
  color: #888;
  font-size: 14px;
  margin-left: auto;
}

.stock-name {
  font-weight: bold;
}
</style>
