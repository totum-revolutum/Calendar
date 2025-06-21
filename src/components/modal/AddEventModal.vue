<script setup>
import { reactive, watch, toRefs } from "vue";

const props = defineProps({
  visible: Boolean,
  defaultDate: String,
  editingEvent: Object,
  coords: {
    type: Object,
    default: () => ({ x: 0, y: 0 }),
  },
});

const emit = defineEmits(["save", "close", "delete"]);

const localEvent = reactive({
  title: "",
  date: "",
  time: "",
  note: "",
  color: "#3788d8",
});

watch(
  () => props.defaultDate,
  (val) => {
    localEvent.date = val || "";
  }
);

function resetForm() {
  localEvent.title = "";
  localEvent.date = props.defaultDate || "";
  localEvent.time = "";
  localEvent.note = "";
  localEvent.color = "#3788d8";
}

watch(
  () => props.visible,
  (val) => {
    if (val) {
      if (props.editingEvent) {
        Object.assign(localEvent, props.editingEvent);
      } else {
        resetForm();
      }
    }
  }
);

function saveEvent() {
  if (localEvent.title && localEvent.date) {
    const payload = { ...localEvent };
    console.log("payload", payload);
    if (props.editingEvent?.id) {
      console.log(" props.editingEvent", props.editingEvent);

      payload.id = props.editingEvent.id;
    }
    emit("save", payload);
  }
}
</script>

<template>
  <div v-if="visible" class="modal-overlay">
    <div
      class="modal-arrow"
      :style="{
        position: 'absolute',
        top: props.coords?.y + 10 + 'px',
        left: props.coords?.x + 'px',
        transform: 'translateX(-50%) rotate(45deg)',
      }"
    ></div>

    <div
      class="modal"
      :style="{
        position: 'absolute',
        top: props.coords?.y + 20 + 'px',
        left: props.coords?.x + 'px',
        transform: 'translateX(-50%)',
      }"
    >
      <button class="close-btn" @click="$emit('close')">✕</button>

      <label>event name</label>
      <div class="input-wrapper">
        <input type="text" v-model="localEvent.title" />
      </div>

      <label>event date</label>
      <div class="input-wrapper">
        <input type="date" v-model="localEvent.date" />
      </div>

      <label>event time</label>
      <div class="input-wrapper">
        <input type="time" v-model="localEvent.time" />
      </div>

      <label>notes</label>
      <div class="input-wrapper">
        <input type="text" v-model="localEvent.note" />
      </div>

      <label>Color</label>
      <div class="input-wrapper">
        <input type="color" v-model="localEvent.color" />
      </div>

      <div class="modal-actions">
        <button @click="$emit('close')" class="cancel-btn">Cancel</button>
        <button
          v-if="props.editingEvent?.id"
          @click="emit('delete', props.editingEvent.id)"
          class="delete-btn"
        >
          Delete
        </button>
        <button @click="saveEvent" class="save-btn">Save</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.3);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-arrow {
  width: 20px;
  height: 20px;
  background: black;
  z-index: 9;
}

.modal {
  z-index: 10;
  background: white;
  border-radius: 10px;
  padding: 20px;
  width: 280px;
  border: 1.5px solid black;
}

.modal label {
  display: block;
  margin-top: 10px;
  font-size: 14px;

  color: rgb(201, 199, 199);
}

.modal input {
  width: 100%;
  padding: 6px;
  margin-top: 4px;

  border: none;
}

.input-wrapper {
  position: relative;
  margin-bottom: 10px;
}

.input-wrapper::after {
  content: "";
  position: absolute;
  left: 0;
  bottom: 0;
  height: 2px;
  width: 100%;
  background: rgb(228, 228, 228);
}

.modal-actions {
  margin-top: 35px;
  display: flex;
  justify-content: space-between;
}

.cancel-btn,
.delete-btn {
  color: red;
  background: none;
  border: none;
  cursor: pointer;
}

.save-btn {
  background-color: white;
  color: rgb(121, 121, 121);
  padding: 6px 12px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.close-btn {
  position: absolute;
  top: 8px;
  right: 10px;
  background: none;
  color: rgb(228, 228, 228);
  border: 3px solid rgb(228, 228, 228);
  border-radius: 50%;

  font-weight: 900;
  font-size: 13px;
  padding: 1px 5px;
  cursor: pointer;
}
</style>
