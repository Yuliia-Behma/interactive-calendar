<script setup lang="ts">
import { ref, onMounted } from "vue";
const now = new Date();
let currentMonth = ref(now.getMonth());
let currentYear = ref(now.getFullYear());

const selectedDate = ref<Date | null>(null);

const prevBtn = ref<HTMLElement | null>(null);
const nextBtn = ref<HTMLElement | null>(null);
const monthElem = ref<HTMLElement | null>(null);
let dateNumberElements: HTMLElement[] = [];

interface Event {
  id: number;
  title: string;
  date: Date;
  time: string;
}

const events = ref<Event[]>([]);
let eventIdCounter = 0;

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
      el.classList.add("today");
    } else {
      el.classList.remove("today");
    }
    // Очищаємо події
    el.innerHTML =
      dateNumber > 0 && dateNumber <= numDaysInMonth ? dateNumberAsString : "";
    // Додаємо події до днів
    if (dateNumber > 0 && dateNumber <= numDaysInMonth) {
      const dayDate = new Date(year, monthIndex, dateNumber);
      const dayEvents = events.value.filter(
        (event) => event.date.toDateString() === dayDate.toDateString()
      );
      dayEvents.forEach((event) => {
        const eventElem = document.createElement("div");
        eventElem.className = "event";
        eventElem.draggable = true;
        eventElem.innerText = `∘${event.time}-${event.title}`;
        eventElem.dataset.id = event.id.toString();
        el.appendChild(eventElem);
      });
    }

    // Додаємо обробник кліку
    el.addEventListener("click", () => {
      selectedDate.value =
        dateNumber > 0 && dateNumber <= numDaysInMonth
          ? new Date(year, monthIndex, dateNumber)
          : null;
      renderMonth(currentMonth.value, currentYear.value);
    });

    // Виділяємо вибрану дату
    if (
      selectedDate.value &&
      selectedDate.value.getDate() === dateNumber &&
      selectedDate.value.getMonth() === monthIndex &&
      selectedDate.value.getFullYear() === year
    ) {
      el.classList.add("selected");
    } else {
      el.classList.remove("selected");
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

const createEvent = (title: string, date: Date, time: string) => {
  events.value.push({ id: eventIdCounter++, title, date, time });
  renderMonth(currentMonth.value, currentYear.value);
};

const editEvent = (
  id: number,
  newTitle: string,
  newDate: Date,
  newTime: string
) => {
  const event = events.value.find((event) => event.id === id);
  if (event) {
    event.title = newTitle;
    event.date = newDate;
    event.time = newTime;
    renderMonth(currentMonth.value, currentYear.value);
  }
};

const deleteEvent = (id: number) => {
  events.value = events.value.filter((event) => event.id !== id);
  renderMonth(currentMonth.value, currentYear.value);
};

const handleDragStart = (event: DragEvent) => {
  const target = event.target as HTMLElement;
  event.dataTransfer?.setData("text/plain", target.dataset.id || "");
};

const handleDrop = (event: DragEvent, targetDate: Date) => {
  const id = parseInt(event.dataTransfer?.getData("text/plain") || "");
  const eventObj = events.value.find((event) => event.id === id);
  if (eventObj) {
    eventObj.date = targetDate;
    renderMonth(currentMonth.value, currentYear.value);
  }
};

onMounted(() => {
  dateNumberElements = Array.from(
    document.querySelectorAll(".date-box")
  ) as HTMLElement[];
  renderMonth(currentMonth.value, currentYear.value);

  dateNumberElements.forEach((el) => {
    el.addEventListener("dragover", (e) => e.preventDefault());
    el.addEventListener("drop", (e) => {
      const dateNumber = parseInt(el.innerText);
      const targetDate = new Date(
        currentYear.value,
        currentMonth.value,
        dateNumber
      );
      handleDrop(e as DragEvent, targetDate);
    });
  });
});

const newEventTitle = ref("");
const newEventTime = ref("");

const addEvent = () => {
  if (selectedDate.value && newEventTitle.value && newEventTime.value) {
    createEvent(newEventTitle.value, selectedDate.value, newEventTime.value);
    newEventTitle.value = "";
    newEventTime.value = "";
  } else {
    alert("Please select a date, enter a valid title, and time");
  }
};

// Отримуємо події для вибраної дати
const eventsForSelectedDate = computed(() => {
  if (!selectedDate.value) return [];
  return events.value.filter(
    (event) => event.date.toDateString() === selectedDate.value?.toDateString()
  );
});

const editEventHandler = (event: Event) => {
  if (selectedDate.value !== null) {
    editEvent(
      event.id,
      prompt("New Title", event.title) || event.title,
      selectedDate.value,
      prompt("New Time", event.time) || event.time
    );
  }
};
</script>

<template>
  <div class="general-wraper">
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

      <div
        class="date-box"
        v-for="n in 42"
        :key="n"
        @dragstart="handleDragStart"
      ></div>
    </div>

    <div class="add-event">
      <h3>Add New Event</h3>
      <div class="inputBlock">
        <input
          class="titleInput"
          type="text"
          v-model="newEventTitle"
          placeholder="Event Title"
        />
        <input
          class="timeInput"
          type="time"
          v-model="newEventTime"
          placeholder="Event Time"
        />
      </div>
      <button @click="addEvent">Add Event</button>
    </div>
    <div v-if="selectedDate" class="selected-date">
      <h2 class="currentDate">
        {{ selectedDate.getDate() }}.{{
          monthIndexToName[selectedDate.getMonth()]
        }}.{{ selectedDate.getFullYear() }}
      </h2>
      <div
        v-for="event in eventsForSelectedDate"
        :key="event.id"
        class="event-details"
      >
        <div>{{ event.time }} - {{ event.title }}</div>
        <button class="edit" @click="editEventHandler(event)">Edit</button>
        <button class="delete" @click="deleteEvent(event.id)">Delete</button>
      </div>
    </div>
    <div v-else>
      <p>Select a date to view or add events</p>
    </div>
  </div>
</template>

<style lang="scss">
.general-wraper {
  display: grid;
  width: 100%;
  grid-template-columns: repeat(2, 1fr);
  grid-template-rows: 170px auto;
  gap: 10px;
  box-sizing: border-box;
}

.wraperCalendar {
  display: grid;
  grid-template-columns: repeat(7, minmax(30px, 1fr));
  grid-row: span 2;
  gap: 15px;
  font-family: var(--font);
  border: 1px solid var(--border);
  border-radius: 6px;
  padding: 20px 25px 15px 25px;
  width: 100%;
  box-shadow: var(--light-box-shadow);
  background-color: #fff;
  box-sizing: border-box;
}

.wraperCalendar > div {
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: start;
  justify-content: start;
  min-height: 50px;
  padding: 3px;
}

.add-event,
.selected-date {
  font-family: var(--font);
  border: 1px solid var(--border);
  border-radius: 6px;
  width: 100%;
  box-shadow: var(--light-box-shadow);
  background-color: #fff;
  box-sizing: border-box;
  padding: 15px;
}

.add-event h3 {
  font-family: var(--font-alt);
  font-weight: 700;
  color: var(--dark);
  margin: 10px 0;
}

.add-event button {
  padding: 10px 25px;
  background-color: var(--success);
  color: var(--primary--light-color);
  font-weight: 500;
  margin-top: 20px;
  border-radius: var(--radius-rounded);
  outline: none;
  box-shadow: var(--light-box-shadow);
}

.add-event button:hover {
  box-shadow: 0 2px 10px #05d69e, 0 1px 5px #000000a2;
}

.currentDate {
  font-size: 18px;
  color: var(--dark-text);
  margin: 10px;
}

.date-box {
  box-shadow: var(--light-box-shadow);
  border-radius: 6px;
  overflow: hidden;
  font-size: 10px;
}

.date-box.selected {
  border: 2px solid var(--info);
}

.month-year {
  grid-column: span 5;
  text-transform: uppercase;
  line-height: 44px;
  justify-self: center;
  align-self: center;
}

.day-name {
  font-weight: bold;
  font-size: 16px;
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

.inputBlock {
  display: flex;
  width: 100%;
  flex-direction: row;
}

.titleInput {
  flex-grow: 1;
  padding: 8px 16px;
  border-top-left-radius: 6px;
  border-bottom-left-radius: 6px;
  outline: none;
  border: 1px solid var(--muted-grey);
}

.timeInput {
  outline: none;
  border: 1px solid var(--muted-grey);
  border-top-right-radius: 6px;
  border-bottom-right-radius: 6px;
}

.today {
  background-color: var(--success--light-color);
  border: 2px solid #05d69e7c;
}

.selected-date h2 {
  margin: 0;
}

.event-details {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 25px;
  border-radius: var(--radius-rounded);
  box-shadow: var(--light-box-shadow);
  margin-bottom: 10px;
}

.event-details div {
  flex-grow: 1;
}

.delete {
  background-color: var(--danger);
}

.edit {
  padding: 5px 15px;
  border: 1px solid var(--warning);
  border-radius: var(--radius-rounded);
  margin-right: 15px;
  color: var(--warning);
  font-weight: 700;
}
</style>
