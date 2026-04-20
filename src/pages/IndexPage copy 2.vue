<template>
  <q-page class="row">
    <!-- <div v-if="drawerOpen" class="col-auto q-pa-md bg-grey-1" style="width: 270px; border-right: 1px solid #e0e0e0;">
      <q-scroll-area class="fit">
        <div class="text-subtitle2 text-weight-bold text-grey-8 q-mb-sm">Filtrar por Status</div>
        <q-list dense>
          <q-item
            v-for="(cfg, st) in STATUS_CONFIG"
            :key="st"
            clickable
            v-ripple
            :active="activeFilters.includes(st as AppointmentStatus)"
            active-class="bg-blue-1"
            @click="toggleFilter(st as AppointmentStatus)"
            class="rounded-borders q-mb-xs"
          >
            <q-item-section avatar>
              <q-icon :name="cfg.icon" :color="cfg.color" size="20px" />
            </q-item-section>
            <q-item-section>
              <q-item-label>{{ cfg.label }}</q-item-label>
            </q-item-section>
            <q-item-section side>
              <q-badge :color="cfg.color" :label="countByStatus(st as AppointmentStatus)" />
            </q-item-section>
          </q-item>
        </q-list>

        <q-separator class="q-my-md" />

        <div class="text-subtitle2 text-weight-bold text-grey-8 q-mb-sm">Resumo da Semana</div>
        <q-card flat bordered class="bg-white">
          <q-card-section class="q-pa-sm">
            <div
              v-for="(cfg, st) in STATUS_CONFIG"
              :key="st"
              class="row items-center justify-between q-mb-xs"
            >
              <div class="row items-center">
                <q-icon :name="cfg.icon" :color="cfg.color" size="16px" class="q-mr-xs" />
                <span class="text-caption text-grey-7">{{ cfg.label }}</span>
              </div>
              <q-badge :color="cfg.color" :label="countByStatus(st as AppointmentStatus)" />
            </div>
            <q-separator class="q-my-xs" />
            <div class="row items-center justify-between">
              <span class="text-caption text-weight-bold text-grey-8">Total</span>
              <span class="text-caption text-weight-bold">{{ filteredAppointments.length }}</span>
            </div>
          </q-card-section>
        </q-card>

        <q-separator class="q-my-md" />
        <q-btn
          outline color="primary" label="Limpar Filtros" icon="filter_list_off"
          class="full-width" size="sm"
          @click="clearFilters"
          :disable="activeFilters.length === 0"
        />
      </q-scroll-area>
    </div> -->

    <div class="col column">
      <!-- <div class="row items-center q-pa-sm bg-white shadow-1">
        <q-btn flat dense round icon="menu" @click="drawerOpen = !drawerOpen" class="q-mr-sm" />
        <q-icon name="event" size="28px" class="q-mr-sm" />
        <div class="text-h6 text-weight-bold">Agenda Semanal</div>
        <q-space />
        <q-btn flat round dense icon="today" @click="goToToday"><q-tooltip>Ir para hoje</q-tooltip></q-btn>
        <q-btn flat round dense icon="add" @click="showNewDialog = true"><q-tooltip>Novo Agendamento</q-tooltip></q-btn>
        
        <div class="row items-center q-ml-md">
          <q-btn flat round dense icon="chevron_left" @click="prevWeek" />
          <div class="text-subtitle1 text-weight-medium q-mx-sm">{{ weekRangeLabel }}</div>
          <q-btn flat round dense icon="chevron_right" @click="nextWeek" />
        </div>
      </div> -->

      <div class="col q-pa-sm">
        <q-calendar-day
          v-model="selectedDate"
          view="week"
          locale="pt-BR"
          :weekdays="[1, 2, 3, 4, 5, 6, 0]"
          :interval-start="8"
          :interval-count="14"
          :interval-height="60"
          :hour24-format="true"
          no-active-date
          bordered
          class="scheduler-calendar col"
        >
          <template #day-body="{ scope }">
            <template v-for="ev in getEventsForDate(scope.timestamp.date)" :key="ev.id">
              <div class="event-block" :style="getEventStyle(ev)" @click.stop="openDetail(ev.appointment)">
                <div class="event-content">
                  <div class="event-service">{{ ev.appointment.service }}</div>
                  <div class="event-client"><q-icon name="person" size="11px" /> {{ ev.appointment.clientName }}</div>
                  <div class="event-time" v-if="ev.duration >= 45"><q-icon name="schedule" size="11px" /> {{ ev.appointment.startTime }} – {{ ev.appointment.endTime }}</div>
                  <div v-if="ev.duration >= 60" style="margin-top: 1px">
                    <q-badge :color="STATUS_CONFIG[ev.appointment.status].color" :label="STATUS_CONFIG[ev.appointment.status].label" :text-color="STATUS_CONFIG[ev.appointment.status].textColor" style="font-size: 9px" />
                  </div>
                </div>
              </div>
            </template>
          </template>

          <template #head-day-label="{ scope }">
            <div class="column items-center q-py-xs">
              <div class="text-h6 text-weight-bold" :class="isToday(scope.timestamp.date) ? 'today-circle' : 'text-grey-8'">{{ scope.timestamp.day }}</div>
              <q-badge v-if="getCount(scope.timestamp.date) > 0" color="primary" :label="getCount(scope.timestamp.date)" style="font-size: 9px" />
            </div>
          </template>
        </q-calendar-day>
      </div>
    </div>

    <q-dialog v-model="detailDialog" :maximized="$q.screen.lt.sm">
        </q-dialog>

    <q-dialog v-model="showNewDialog" :maximized="$q.screen.lt.sm">
        </q-dialog>
  </q-page>
</template>

<script setup lang="ts">
import { ref, computed, reactive } from "vue";
import { useQuasar } from "quasar";
import { QCalendarDay } from "@quasar/quasar-ui-qcalendar";

// ─── Tipos ────────────────────────────────────────────────────────────────────

type AppointmentStatus = "confirmado" | "pendente" | "cancelado" | "concluido";

interface Appointment {
  id: number;
  clientId: number;
  clientName: string;
  clientPhone: string;
  status: AppointmentStatus;
  date: string;
  startTime: string;
  endTime: string;
  service: string;
  notes: string;
}

interface CalendarEvent {
  id: number;
  duration: number;
  bgcolor: string;
  appointment: Appointment;
}

interface StatusConfig {
  label: string;
  color: string;
  icon: string;
  textColor: string;
}

// ─── Configuração de Status ───────────────────────────────────────────────────

const STATUS_CONFIG: Record<AppointmentStatus, StatusConfig> = {
  confirmado: { label: "Confirmado",  color: "positive",    icon: "check_circle", textColor: "white" },
  pendente:   { label: "Pendente",    color: "warning",     icon: "schedule",     textColor: "dark"  },
  cancelado:  { label: "Cancelado",   color: "negative",    icon: "cancel",       textColor: "white" },
  concluido:  { label: "Concluído",   color: "blue-grey-6", icon: "done_all",     textColor: "white" },
};

// ─── Cores por Serviço ────────────────────────────────────────────────────────

const SERVICE_COLORS: Record<string, string> = {
  "Corte + Escova":         "#1565C0",
  "Barba":                  "#0277BD",
  "Coloração + Hidratação": "#7B1FA2",
  "Corte Masculino":        "#00695C",
  "Manicure + Pedicure":    "#C62828",
  "Corte + Barba":          "#1B5E20",
  "Progressiva":            "#E65100",
  "Barba Modelagem":        "#004D40",
  "Escova + Penteado":      "#880E4F",
  "Corte Degradê":          "#01579B",
  "Manutenção Coloração":   "#4A148C",
  "Sobrancelha + Cílios":   "#BF360C",
};

function getServiceColor(service: string): string {
  return SERVICE_COLORS[service] ?? "#546E7A";
}

// ─── Dados Iniciais ───────────────────────────────────────────────────────────

const INITIAL_APPOINTMENTS: Appointment[] = [
  { id: 1,  clientId: 1,  clientName: "Ana Silva",         clientPhone: "(11) 98765-4321", status: "confirmado", date: "2026-04-20", startTime: "09:00", endTime: "10:00", service: "Corte + Escova",         notes: "Cliente prefere corte em camadas" },
  { id: 2,  clientId: 2,  clientName: "Carlos Mendes",     clientPhone: "(11) 91234-5678", status: "pendente",   date: "2026-04-20", startTime: "10:30", endTime: "11:00", service: "Barba",                   notes: "" },
  { id: 3,  clientId: 3,  clientName: "Mariana Costa",     clientPhone: "(21) 99876-5432", status: "confirmado", date: "2026-04-21", startTime: "14:00", endTime: "16:30", service: "Coloração + Hidratação",  notes: "Loiro mel, trazer referência" },
  { id: 4,  clientId: 4,  clientName: "Roberto Alves",     clientPhone: "(11) 97654-3210", status: "cancelado",  date: "2026-04-21", startTime: "09:00", endTime: "09:45", service: "Corte Masculino",         notes: "Cancelado pelo cliente" },
  { id: 5,  clientId: 5,  clientName: "Fernanda Lima",     clientPhone: "(31) 98888-7777", status: "concluido",  date: "2026-04-22", startTime: "11:00", endTime: "12:30", service: "Manicure + Pedicure",     notes: "Esmalte vermelho clássico" },
  { id: 6,  clientId: 6,  clientName: "Paulo Santos",      clientPhone: "(11) 96666-5555", status: "confirmado", date: "2026-04-22", startTime: "14:00", endTime: "15:00", service: "Corte + Barba",           notes: "" },
  { id: 7,  clientId: 7,  clientName: "Juliana Rocha",     clientPhone: "(21) 95555-4444", status: "pendente",   date: "2026-04-23", startTime: "09:30", endTime: "11:00", service: "Progressiva",             notes: "Cabelo muito volumoso" },
  { id: 8,  clientId: 8,  clientName: "Diego Ferreira",    clientPhone: "(11) 94444-3333", status: "confirmado", date: "2026-04-23", startTime: "13:00", endTime: "13:30", service: "Barba Modelagem",         notes: "" },
  { id: 9,  clientId: 9,  clientName: "Beatriz Oliveira",  clientPhone: "(41) 93333-2222", status: "confirmado", date: "2026-04-24", startTime: "10:00", endTime: "11:30", service: "Escova + Penteado",       notes: "Para evento no final de semana" },
  { id: 10, clientId: 10, clientName: "Thiago Nascimento", clientPhone: "(11) 92222-1111", status: "pendente",   date: "2026-04-24", startTime: "15:00", endTime: "15:45", service: "Corte Degradê",           notes: "Primeira visita" },
  { id: 11, clientId: 3,  clientName: "Mariana Costa",     clientPhone: "(21) 99876-5432", status: "confirmado", date: "2026-04-25", startTime: "09:00", endTime: "10:00", service: "Manutenção Coloração",    notes: "Retoque das raízes" },
  { id: 12, clientId: 11, clientName: "Luciana Vieira",    clientPhone: "(85) 91111-0000", status: "concluido",  date: "2026-04-25", startTime: "11:00", endTime: "12:00", service: "Sobrancelha + Cílios",    notes: "Design de sobrancelha árabe" },
];

// ─── Estado ───────────────────────────────────────────────────────────────────

const $q = useQuasar();

const drawerOpen    = ref($q.screen.gt.sm);
const selectedDate = ref<string | undefined>("2026-04-21");
const detailDialog  = ref(false);
const showNewDialog = ref(false);
const selected      = ref<Appointment | null>(null);
const appointments  = ref<Appointment[]>([...INITIAL_APPOINTMENTS]);
const activeFilters = ref<AppointmentStatus[]>([]);

const form = reactive({
  clientName: "",
  clientPhone: "",
  service: "",
  date: "2026-04-21",
  startTime: "09:00",
  endTime: "10:00",
  status: "pendente" as AppointmentStatus,
  notes: "",
});

// ─── Computed ─────────────────────────────────────────────────────────────────

// const statusOptions = computed(() =>
//   Object.entries(STATUS_CONFIG).map(([value, cfg]) => ({ label: cfg.label, value }))
// );

// const calendarStyle = computed(() => ({
//   height: $q.screen.lt.sm ? "calc(100vh - 160px)" : "calc(100vh - 148px)",
// }));

// const weekRangeLabel = computed(() => {
//   const date = new Date(selectedDate.value + "T00:00:00");
//   const day  = date.getDay();
//   const diff = day === 0 ? -6 : 1 - day;
//   const mon  = new Date(date); mon.setDate(date.getDate() + diff);
//   const sun  = new Date(mon);  sun.setDate(mon.getDate() + 6);
//   const fmt  = new Intl.DateTimeFormat("pt-BR", { day: "2-digit", month: "short" });
//   return `${fmt.format(mon)} – ${fmt.format(sun)} · ${sun.getFullYear()}`;
// });

const filteredAppointments = computed<Appointment[]>(() => {
  if (activeFilters.value.length === 0) return appointments.value;
  return appointments.value.filter((a) => activeFilters.value.includes(a.status));
});

// ─── Navegação ────────────────────────────────────────────────────────────────

// function goToToday(): void {
//   const t = new Date();
//   selectedDate.value = `${t.getFullYear()}-${String(t.getMonth() + 1).padStart(2, "0")}-${String(t.getDate()).padStart(2, "0")}`;
// }

// function prevWeek(): void {
//   const d = new Date(selectedDate.value + "T00:00:00");
//   d.setDate(d.getDate() - 7);
//   selectedDate.value = d.toISOString().split("T")[0];
// }

// function nextWeek(): void {
//   const d = new Date(selectedDate.value + "T00:00:00");
//   d.setDate(d.getDate() + 7);
//   selectedDate.value = d.toISOString().split("T")[0];
// }

// ─── Filtros e Contagens ──────────────────────────────────────────────────────

function isToday(date: string): boolean {
  const t = new Date();
  return date === `${t.getFullYear()}-${String(t.getMonth() + 1).padStart(2, "0")}-${String(t.getDate()).padStart(2, "0")}`;
}

// function toggleFilter(status: AppointmentStatus): void {
//   const idx = activeFilters.value.indexOf(status);
//   idx === -1 ? activeFilters.value.push(status) : activeFilters.value.splice(idx, 1);
// }

// function clearFilters(): void {
//   activeFilters.value = [];
// }

// function countByStatus(status: AppointmentStatus): number {
//   return appointments.value.filter((a) => a.status === status).length;
// }

function getCount(date: string): number {
  return filteredAppointments.value.filter((a) => a.date === date).length;
}

// ─── Calendário ───────────────────────────────────────────────────────────────

function getDuration(start: string, end: string): number {
  const [sh, sm] = start.split(":").map(Number);
  const [eh, em] = end.split(":").map(Number);

  const startMinutes = ((sh || 0) * 60) + (sm || 0);
  const endMinutes = ((eh || 0) * 60) + (em || 0);

  return endMinutes - startMinutes;
}

function getEventsForDate(date: string): CalendarEvent[] {
  return filteredAppointments.value
    .filter((a) => a.date === date)
    .map((a) => ({
      id: a.id,
      duration: getDuration(a.startTime, a.endTime),
      bgcolor: getServiceColor(a.service),
      appointment: a,
    }));
}

function getEventStyle(ev: CalendarEvent): Record<string, string> {
  const [sh, sm] = ev.appointment.startTime.split(":").map(Number);

  const hour = sh || 0;
  const minute = sm || 0;

  const top    = ((hour - 8) * 60 + minute);
  const height = Math.max(ev.duration - 2, 20);
  return {
    position: "absolute",
    top: `${top}px`,
    left: "2px",
    right: "2px",
    height: `${height}px`,
    backgroundColor: ev.bgcolor,
    borderLeft: `3px solid ${ev.bgcolor}cc`,
    borderRadius: "6px",
    color: "#ffffff",
    cursor: "pointer",
    overflow: "hidden",
    zIndex: "1",
    boxShadow: "0 1px 4px rgba(0,0,0,0.3)",
  };
}

// ─── Modais ───────────────────────────────────────────────────────────────────

function openDetail(appointment: Appointment): void {
  selected.value = appointment;
  detailDialog.value = true;
}

// function formatDate(dateStr: string): string {
//   return new Intl.DateTimeFormat("pt-BR", {
//     weekday: "long", day: "2-digit", month: "long", year: "numeric",
//   }).format(new Date(dateStr + "T00:00:00"));
// }

// function copyPhone(phone: string): void {
//   navigator.clipboard.writeText(phone).then(() => {
//     $q.notify({ message: "Número copiado!", icon: "content_copy", color: "positive", timeout: 1500 });
//   });
// }

// function changeStatus(status: AppointmentStatus): void {
//   if (!selected.value) return;
//   const idx = appointments.value.findIndex((a) => a.id === selected.value!.id);
//   if (idx !== -1) {
//     appointments.value[idx] = { ...appointments.value[idx], status };
//     selected.value = { ...selected.value, status };
//   }
//   $q.notify({
//     message: `Status: "${STATUS_CONFIG[status].label}"`,
//     color: STATUS_CONFIG[status].color,
//     icon: STATUS_CONFIG[status].icon,
//     timeout: 2000,
//   });
// }

// let nextId = INITIAL_APPOINTMENTS.length + 1;

// function addAppointment(): void {
//   if (!form.clientName || !form.clientPhone || !form.service || !form.date) {
//     $q.notify({ message: "Preencha todos os campos obrigatórios", color: "negative", icon: "warning" });
//     return;
//   }
//   appointments.value.push({
//     id: nextId++,
//     clientId: nextId,
//     clientName: form.clientName,
//     clientPhone: form.clientPhone,
//     status: form.status,
//     date: form.date,
//     startTime: form.startTime,
//     endTime: form.endTime,
//     service: form.service,
//     notes: form.notes,
//   });
//   selectedDate.value  = form.date;
//   showNewDialog.value = false;
//   Object.assign(form, { clientName: "", clientPhone: "", service: "", notes: "", status: "pendente" });
//   $q.notify({ message: "Agendamento criado!", color: "positive", icon: "check_circle", timeout: 2000 });
// }
</script>

<style scoped>
.scheduler-calendar { border-radius: 8px; }

.event-block { transition: opacity 0.15s ease, transform 0.15s ease; }
.event-block:hover { opacity: 0.88; transform: scale(1.01); }

.event-content {
  padding: 3px 5px;
  height: 100%;
  display: flex;
  flex-direction: column;
  gap: 1px;
  overflow: hidden;
}

.event-service {
  font-size: 11px;
  font-weight: 700;
  line-height: 1.3;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.event-client {
  font-size: 10px;
  opacity: 0.9;
  display: flex;
  align-items: center;
  gap: 2px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.event-time {
  font-size: 10px;
  opacity: 0.85;
  display: flex;
  align-items: center;
  gap: 2px;
}

.today-circle {
  background-color: #1565C0;
  color: white;
  border-radius: 50%;
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.1rem;
  font-weight: 700;
}
</style>
