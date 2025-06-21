<script setup>
import { ref } from "vue";
import FullCalendar from "@fullcalendar/vue3";
import dayGridPlugin from "@fullcalendar/daygrid";
import timeGridPlugin from "@fullcalendar/timegrid";
import interactionPlugin from "@fullcalendar/interaction";
import listPlugin from "@fullcalendar/list";
import AddEventModal from "./modal/AddEventModal.vue";

const calendarRef = ref(null);
const currentView = ref("dayGridMonth");
const showModal = ref(false);
const selectedDate = ref("");
const editingEvent = ref(null);
const modalCoords = ref({ x: 0, y: 0 });

const calendarOptions = ref({
  plugins: [dayGridPlugin, timeGridPlugin, interactionPlugin, listPlugin],
  initialView: "dayGridMonth",
  headerToolbar: false,
  editable: true,
  height: 650,
  contentHeight: 650,
  dayMaxEventRows: true,
  expandRows: true,
  nowIndicator: true,
  eventDisplay: "block",
  slotMinTime: "08:00:00",
  slotMaxTime: "20:00:00",
  scrollTime: "08:00:00",
  views: {
    dayGridMonth: {
      displayEventTime: false,
      contentHeight: "auto",
    },
    timeGridWeek: {
      contentHeight: "auto",
    },
    timeGridDay: {
      contentHeight: "auto",
    },
  },
  eventTimeFormat: false,
  events: [],
  dateClick: handleDateClick,
  eventClick: handleEventClick,
  eventDrop(info) {
    const event = info.event;
    const updatedEvent = {
      id: event.id,
      title: event.title,
      date: event.startStr.split("T")[0],
      time: event.startStr.split("T")[1]?.slice(0, 5) || "",
      note: event.extendedProps.note || "",
      color: event.backgroundColor || "#3788d8",
    };
    saveEvent(updatedEvent);
  },
});

const changeView = (view) => {
  calendarRef.value.getApi().changeView(view);
  currentView.value = view;
};

const goToday = () => {
  calendarRef.value.getApi().today();
};

const goPrev = () => {
  calendarRef.value.getApi().prev();
};

const goNext = () => {
  calendarRef.value.getApi().next();
};

function handleDateClick(arg) {
  const [datePart, timePart] = arg.dateStr.split("T");

  selectedDate.value = datePart;

  if (timePart) {
    editingEvent.value = {
      title: "",
      date: datePart,
      time: timePart.slice(0, 5),
      note: "",
      color: "#3788d8",
    };
  } else {
    editingEvent.value = null;
  }

  modalCoords.value = { x: arg.jsEvent.pageX, y: arg.jsEvent.pageY };
  showModal.value = true;
}

function handleEventClick(arg) {
  const event = arg.event;
  editingEvent.value = {
    id: event.id,
    title: event.title,
    date: event.startStr.split("T")[0],
    time: event.startStr.split("T")[1]?.slice(0, 5) || "",
    note: event.extendedProps.note || "",
    color: event.backgroundColor || "#3788d8",
  };
  selectedDate.value = editingEvent.value.date;
  modalCoords.value = { x: arg.jsEvent.pageX, y: arg.jsEvent.pageY };
  showModal.value = true;
}

function saveEvent(eventData) {
  if (eventData.id) {
    const event = calendarRef.value.getApi().getEventById(eventData.id);
    if (event) {
      event.setProp("title", eventData.title);
      event.setStart(`${eventData.date}T${eventData.time}`);
      event.setProp("backgroundColor", eventData.color);
      event.setExtendedProp("note", eventData.note);
    }
  } else {
    calendarRef.value.getApi().addEvent({
      id: String(Date.now()),
      title: eventData.title,
      start: `${eventData.date}T${eventData.time}`,
      backgroundColor: eventData.color,
      note: eventData.note,
    });
  }
  showModal.value = false;
  editingEvent.value = null;
}

function deleteEvent(eventId) {
  const event = calendarRef.value.getApi().getEventById(eventId);
  if (event) {
    event.remove();
  }
  showModal.value = false;
  editingEvent.value = null;
}
</script>

<template>
  <div class="container">
    <h1 class="title">Calendar</h1>

    <div class="calendar-wrapper">
      <div class="calendar-box">
        <div class="calendar-header">
          <div class="nav-buttons">
            <button class="btn" @click="goToday">Today</button>
            <button class="btn" @click="goPrev">Back</button>
            <button class="btn" @click="goNext">Next</button>
          </div>
          <div class="current-title">
            {{ calendarRef?.getApi()?.view?.title || "Loading..." }}
          </div>
          <div class="view-buttons">
            <button
              class="btn"
              :class="{ active: currentView === 'dayGridMonth' }"
              @click="changeView('dayGridMonth')"
            >
              Month
            </button>
            <button
              class="btn"
              :class="{ active: currentView === 'timeGridWeek' }"
              @click="changeView('timeGridWeek')"
            >
              Week
            </button>
            <button
              class="btn"
              :class="{ active: currentView === 'timeGridDay' }"
              @click="changeView('timeGridDay')"
            >
              Day
            </button>
            <button
              class="btn"
              :class="{ active: currentView === 'listWeek' }"
              @click="changeView('listWeek')"
            >
              Agenda
            </button>
          </div>
        </div>
        <FullCalendar ref="calendarRef" :options="calendarOptions" />

        <AddEventModal
          :visible="showModal"
          :defaultDate="selectedDate"
          :coords="modalCoords"
          :editingEvent="editingEvent"
          @save="saveEvent"
          @delete="deleteEvent"
          @close="showModal = false"
        />
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
  padding: 24px;
  background-color: #f8f9fc;
  box-sizing: border-box;
  height: 100%;
}

.title {
  font-size: 20px;
  padding-inline: 20%;
  font-weight: 600;
  margin-bottom: 30px;
  text-align: left;
}

.calendar-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
}

.calendar-box {
  width: 50%;
  height: 700px;
  background: white;
  padding: 16px;
  border-radius: 8px;
  box-shadow: 0 1px 5px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.calendar-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.nav-buttons,
.view-buttons {
  display: flex;
}

.btn {
  padding: 6px 12px;
  font-size: 14px;
  background-color: white;
  border: 0.5px solid rgb(168, 168, 168);
  border-radius: 2px;
  color: rgb(97, 97, 97);
  cursor: pointer;
}

.btn:hover {
  background-color: #f0f0f0;
}

.btn.active {
  color: #3788d8;
}

.current-title {
  font-size: 16px;
  font-weight: 500;
}

:deep(.fc-col-header-cell) {
  background-color: #f3f4f6;
  color: grey;
  padding-block: 6px;
  font-size: 13px;
}

:deep(.fc-timegrid) {
  overflow-y: auto;
  max-height: 600px;
}

:deep(.fc-daygrid-event) {
  padding: 4px 6px;
  line-height: 1.3;
}

:deep(.fc-event-title) {
  padding-block: 2px;
  overflow: hidden !important;
}

:deep(.fc-day-today) {
  background-color: #f0f0f0 !important;
}

:deep(.fc .fc-timegrid-now-indicator-line) {
  border-color: #7bb4ff !important;
  border-width: 1.5px !important;
}

:deep(.fc-timegrid-slots) {
  height: auto !important;
}

:deep(.fc-timegrid-now-indicator-arrow) {
  border: none !important;
  width: 0 !important;
  height: 0 !important;
  margin-left: 49px !important;
  position: relative !important;
}

:deep(.fc-timegrid-now-indicator-arrow::before) {
  content: "" !important;
  position: absolute !important;
  left: 0 !important;
  width: 0 !important;
  height: 0 !important;
  border-left: 6px solid #7bb4ff !important;
  border-top: 6px solid transparent !important;
  border-bottom: 6px solid transparent !important;
}

:deep(.fc-timeGridWeek-view) .fc-timegrid-body,
:deep(.fc-timeGridDay-view) .fc-timegrid-body {
  overflow-y: auto !important;
  height: calc(100% - 60px) !important;
  overflow-x: hidden;
}

:deep(.fc-scroller) {
  scrollbar-color: #c1c1c1 #f1f1f1;
}

:deep(.fc-event-time) {
  display: none !important;
}

:deep(.fc-event) {
  border: none !important;
}
</style>
