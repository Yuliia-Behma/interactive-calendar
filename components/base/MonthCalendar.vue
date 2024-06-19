<script setup lang="ts">
import { ref, onMounted } from "vue";
const now = new Date();
let currentMonth = ref(now.getMonth());
let currentYear = ref(now.getFullYear());

const prevBtn = ref<HTMLElement | null>(null);
const nextBtn = ref<HTMLElement | null>(null);
const monthElem = ref<HTMLElement | null>(null);
let dateNumberElements: HTMLElement[] = [];

const monthIndexToName: { [key: number]: string } = {
  0: "January",
  1: "February",
  2: "March",
  3: "April",
  4: "May",
  5: "June",
  6: "July",
  7: "August",
  8: "September",
  9: "October",
  10: "November",
  11: "December",
};

const renderMonth = (monthIndex: number, year: number) => {
  const numDaysInMonth = new Date(year, monthIndex + 1, 0).getDate();
  const firstDate = new Date(year, monthIndex);
  const firstDay = firstDate.getDay();

  dateNumberElements.forEach((el, i) => {
    const dateNumber = i + 1 - firstDay;
    const dateNumberAsString = dateNumber.toString();
    el.innerText =
      dateNumber > 0 && dateNumber <= numDaysInMonth ? dateNumberAsString : "";

    const today = new Date();
    if (
      today.getMonth() == monthIndex &&
      today.getFullYear() == year &&
      today.getDate() == i + 1 - firstDay
    ) {
      el.classList.add('today');
    } else {
      el.classList.remove('today');
    }
  });
};

const prevBtnClick = () => {
  if (currentMonth.value === 0) {
    currentMonth.value = 11;
    currentYear.value--;
  } else {
    currentMonth.value--;
  }
  renderMonth(currentMonth.value, currentYear.value);
};

const nextBtnClick = () => {
  if (currentMonth.value === 11) {
    currentMonth.value = 0;
    currentYear.value++;
  } else {
    currentMonth.value++;
  }
  renderMonth(currentMonth.value, currentYear.value);
};

onMounted(() => {
  dateNumberElements = Array.from(
    document.querySelectorAll(".date-box")
  ) as HTMLElement[];
  renderMonth(currentMonth.value, currentYear.value);
});
</script>

<template>
  <div class="wraperCalendar">
    <button ref="prevBtn" class="prevBtn" @click="prevBtnClick">
      <svg
        aria-hidden="true"
        xmlns="http://www.w3.org/2000/svg"
        width="24"
        height="24"
        fill="none"
        viewBox="0 0 24 24"
      >
        <path
          stroke="#0398e2"
          stroke-linecap="round"
          stroke-linejoin="round"
          stroke-width="2"
          d="m15 19-7-7 7-7"
        />
      </svg>
    </button>

    <div ref="monthElem" class="month-year">
      {{ monthIndexToName[currentMonth] }} - {{ currentYear }}
    </div>

    <button ref="nextBtn" class="nextBtn" @click="nextBtnClick">
      <svg
        aria-hidden="true"
        xmlns="http://www.w3.org/2000/svg"
        width="24"
        height="24"
        fill="none"
        viewBox="0 0 24 24"
      >
        <path
          stroke="#0398e2"
          stroke-linecap="round"
          stroke-linejoin="round"
          stroke-width="2"
          d="m9 5 7 7-7 7"
        />
      </svg>
    </button>

    <div class="day-name">Sun</div>
    <div class="day-name">Mon</div>
    <div class="day-name">Tue</div>
    <div class="day-name">Wed</div>
    <div class="day-name">Thu</div>
    <div class="day-name">Fri</div>
    <div class="day-name">Sat</div>

    <div class="date-box" v-for="n in 42" :key="n" @dragstart="handleDragStart"></div>
  </div>
</template>

<style lang="scss">
.wraperCalendar {
  display: grid;
  grid-template-columns: repeat(7, minmax(30px, 1fr));
  gap: 15px;
  font-family: var(--font);
  border: 1px solid var(--border);
  border-radius: 6px;
  padding: 20px 25px 15px 25px;
  max-width: 350px;
  box-shadow: var(--light-box-shadow);
  background-color: #fff;
}

.wraperCalendar > div {
  text-align: center;
  display: flex;
  align-items: center;
  justify-content: center;
}

.date-box {
  aspect-ratio: 1/1;
}

.month-year {
  grid-column: span 5;
  text-transform: uppercase;
  line-height: 24px;
}

.day-name {
  font-weight: bold;
  font-size: 10px;
}

button {
  border: none;
  padding: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #fff;
  outline: none;
}

.today {
  background-color: var(--success);
  border-radius: 50%;
}
</style>
