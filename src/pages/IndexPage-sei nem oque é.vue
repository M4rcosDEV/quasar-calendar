<template>
  <q-page class="q-pa-md bg-grey-2">
    <div class="column full-height bg-white shadow-2 border-radius-lg overflow-hidden">
      
      <div class="row items-center q-pa-md border-bottom bg-white">
        <div class="column">
          <div class="text-h6 text-weight-bolder">Quadro de Equipa</div>
          <div class="text-caption text-grey-7">{{ formattedDate }}</div>
        </div>
        <q-space />
        <div class="row q-gutter-sm">
          <q-select 
            v-model="filterStatus" 
            :options="['Todos', 'Confirmados', 'Pendentes']" 
            dense outlined label="Filtrar Status" 
            style="min-width: 150px"
          />
          <q-btn-group unelevated>
            <q-btn color="primary" icon="chevron_left" @click="onPrev" />
            <q-btn color="primary" label="Hoje" @click="onToday" />
            <q-btn color="primary" icon="chevron_right" @click="onNext" />
          </q-btn-group>
        </div>
      </div>

      <q-calendar-day
        ref="calendar"
        v-model="selectedDate"
        view="day-resource"
        :resources="resources"
        locale="pt-BR"
        :interval-start="8"
        :interval-count="14"
        :interval-height="70"
        :hour24-format="true"
        bordered
        class="col"
      >
        <template #resource-label="{ scope }">
          <div class="column items-center q-pa-sm full-width">
            <q-avatar size="50px" class="shadow-2 q-mb-xs">
              <img :src="scope.resource.avatar">
              <q-badge floating rounded :color="scope.resource.online ? 'positive' : 'grey-5'" />
            </q-avatar>
            <div class="text-weight-bold text-dark text-center" style="font-size: 13px">
              {{ scope.resource.name }}
            </div>
            <div class="text-caption text-grey-6" style="font-size: 10px">
              {{ scope.resource.role }}
            </div>
          </div>
        </template>

        <template #day-resource-body="{ scope }">
          <template v-for="ev in getEvents(scope.timestamp.date, scope.resource.id)" :key="ev.id">
            <div
              class="resource-event shadow-1"
              :style="getEventStyle(ev)"
              @click="openDetail(ev.appointment)"
            >
              <div v-if="ev.duration < 40" class="row items-center full-height q-px-sm">
                <span class="text-weight-bold q-mr-xs">{{ ev.appointment.startTime }}</span>
                <span class="ellipsis">{{ ev.appointment.clientName }}</span>
              </div>

              <div v-else class="column q-pa-sm full-height">
                <div class="row items-center justify-between no-wrap">
                  <div class="text-weight-bolder ellipsis" style="font-size: 12px">{{ ev.appointment.clientName }}</div>
                  <q-icon :name="STATUS_ICONS[ev.appointment.status]" size="14px" />
                </div>
                <div class="text-caption ellipsis opacity-80" style="font-size: 11px">{{ ev.appointment.service }}</div>
                <q-space />
                <div class="row items-center justify-between text-italic" style="font-size: 10px">
                  <span>{{ ev.appointment.startTime }}</span>
                  <q-icon name="more_horiz" />
                </div>
              </div>

              <q-tooltip class="bg-dark shadow-5">
                {{ ev.appointment.clientName }} - {{ ev.appointment.service }}
                <br>Status: {{ ev.appointment.status.toUpperCase() }}
              </q-tooltip>
            </div>
          </template>
        </template>
      </q-calendar-day>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import { QCalendarDay } from '@quasar/quasar-ui-qcalendar';

// ─── Definição de Recursos (Equipa) ──────────────────────────────────────────
const resources = ref([
  { id: 1, name: 'Lucas Barber', role: 'Sênior', avatar: 'https://cdn.quasar.dev/img/avatar1.jpg', online: true },
  { id: 2, name: 'Pedro Fade', role: 'Barbeiro', avatar: 'https://cdn.quasar.dev/img/avatar3.jpg', online: true },
  { id: 3, name: 'Ana Stylist', role: 'Colorista', avatar: 'https://cdn.quasar.dev/img/avatar2.jpg', online: false },
  { id: 4, name: 'Julia Nails', role: 'Manicure', avatar: 'https://cdn.quasar.dev/img/avatar4.jpg', online: true }
]);

const STATUS_ICONS = { confirmado: 'check_circle', pendente: 'history', concluido: 'done_all' };
const selectedDate = ref('2026-04-20');
const filterStatus = ref('Todos');

// ─── Dados Fake ──────────────────────────────────────────────────────────────
const appointments = ref([
  { id: 1, staffId: 1, clientName: 'Carlos Silva', startTime: '08:30', endTime: '09:30', service: 'Corte + Barba', status: 'confirmado' },
  { id: 2, staffId: 1, clientName: 'André M.', startTime: '10:00', endTime: '10:30', service: 'Barba', status: 'pendente' },
  { id: 3, staffId: 2, clientName: 'Roberto Alves', startTime: '09:00', endTime: '10:00', service: 'Corte Fade', status: 'confirmado' },
  { id: 4, staffId: 3, clientName: 'Mariana Costa', startTime: '09:00', endTime: '12:00', service: 'Luzes + Hidratação', status: 'confirmado' }
]);

// ─── Lógica de Layout ────────────────────────────────────────────────────────
function getEvents(date: string, resourceId: number) {
  return appointments.value
    .filter(a => a.date === date && a.staffId === resourceId)
    .map(a => {
      const duration = calculateDuration(a.startTime, a.endTime);
      return {
        id: a.id,
        appointment: a,
        duration,
        top: calculateTop(a.startTime),
        height: Math.max(duration * (70/60), 25) // 70px por hora
      };
    });
}

function calculateTop(time: string) {
  const [h, m] = time.split(':').map(Number);
  return ((h - 8) * 60 + m) * (70/60);
}

function calculateDuration(start: string, end: string) {
  const [sh, sm] = start.split(':').map(Number);
  const [eh, em] = end.split(':').map(Number);
  return (eh * 60 + em) - (sh * 60 + sm);
}

function getEventStyle(ev: any) {
  return {
    position: 'absolute',
    top: `${ev.top}px`,
    height: `${ev.height}px`,
    left: '4px',
    right: '4px',
    backgroundColor: '#34495e',
    color: 'white',
    borderRadius: '8px',
    borderLeft: '5px solid #3b82f6',
    zIndex: 10,
    cursor: 'pointer',
    overflow: 'hidden'
  };
}

const formattedDate = computed(() => 'Segunda, 20 de Abril');
</script>

<style scoped>
.border-bottom { border-bottom: 1px solid #e0e0e0; }
.border-radius-lg { border-radius: 16px; }
.resource-event { transition: transform 0.1s ease; }
.resource-event:hover { transform: scale(1.02); z-index: 20 !important; box-shadow: 0 4px 10px rgba(0,0,0,0.3); }
</style>