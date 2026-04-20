<template>
  <q-page class="row bg-grey-1">
    <div class="col column q-pa-md">
      
      <div class="row items-center justify-between q-mb-md bg-white q-pa-sm rounded-borders shadow-1">
        <div class="row items-center">
          <q-icon name="calendar_month" class="q-mr-sm" color="primary" size="24px" />
          <div>
            <div class="text-h6 text-weight-bold text-dark" style="line-height: 1;">Agenda Semanal</div>
            <div class="text-caption text-grey-6">Semana de 20 a 26 de Abril, 2026</div>
          </div>
        </div>
        
        <q-btn-toggle
          v-model="viewMode"
          toggle-color="primary"
          color="grey-2"
          text-color="grey-8"
          unelevated
          rounded
          class="shadow-1"
          :options="[
            { label: 'Horários', value: 'hourly', icon: 'schedule' },
            { label: 'Períodos', value: 'period', icon: 'date_range' }
          ]"
        />
      </div>

      <div class="col bg-white rounded-borders shadow-1 overflow-hidden flex">
        
        <div v-if="viewMode === 'period'" class="full-width full-height column">
          <div class="bg-grey-2 q-pa-sm border-bottom text-weight-bold text-grey-8 row items-center">
            <q-icon name="info" class="q-mr-sm" /> 
            Exibindo apenas agendamentos de múltiplos dias (Cursos, Retiros, Férias)
          </div>
          
          <div class="row flex-center border-bottom bg-white" style="height: 60px;">
             <div v-for="(day, index) in weekDates" :key="`header-${day}`" class="col text-center border-right">
                <div class="text-caption text-uppercase text-weight-bold text-grey-5">{{ getWeekdayName(index) }}</div>
                <div class="text-h6 text-weight-bolder" :class="isToday(day) ? 'text-primary' : 'text-dark'">
                  {{ day.split('-')[2] }}
                </div>
             </div>
          </div>

          <div class="row col bg-grey-1 relative-position">
            <div 
              v-for="day in weekDates" 
              :key="day" 
              class="col border-right q-pa-sm column q-gutter-y-sm"
            >
              <template v-for="ev in getPeriodEventsForDate(day)" :key="ev.id">
                <q-card 
                  flat 
                  class="period-card cursor-pointer" 
                  :style="{ borderLeft: `4px solid ${ev.theme.solid}`, backgroundColor: ev.theme.light }"
                  @click="openDetail(ev.appointment)"
                >
                  <q-card-section class="q-pa-sm">
                    <div class="text-weight-bold text-dark ellipsis" style="font-size: 0.85rem;">{{ ev.appointment.clientName }}</div>
                    <div class="text-caption text-grey-8 ellipsis">{{ ev.appointment.service }}</div>
                    <div class="row items-center q-mt-xs text-grey-6" style="font-size: 0.7rem;">
                      <q-icon name="event" size="12px" class="q-mr-xs" />
                      {{ ev.appointment.startDate.split('-')[2] }}/{{ ev.appointment.startDate.split('-')[1] }} até {{ ev.appointment.endDate.split('-')[2] }}/{{ ev.appointment.endDate.split('-')[1] }}
                    </div>
                  </q-card-section>
                </q-card>
              </template>
              
              <div v-if="getPeriodEventsForDate(day).length === 0" class="text-center text-grey-4 q-mt-md text-caption">
                Livre
              </div>
            </div>
          </div>
        </div>

        <q-calendar-day
          v-else
          v-model="selectedDate"
          view="week"
          locale="pt-BR"
          :weekdays="[1, 2, 3, 4, 5, 6, 0]"
          :interval-start="8"
          :interval-count="14"
          :interval-height="65" 
          :hour24-format="true"
          no-active-date
          animated
          bordered
          class="scheduler-calendar full-width full-height"
        >
          <template #head-day-label="{ scope }">
            <div class="column items-center q-py-sm">
              <div class="text-caption text-uppercase text-weight-bold text-grey-5">
                {{ scope.timestamp.weekdayName }}
              </div>
              <div 
                class="text-h5 text-weight-bolder q-mt-xs q-mb-xs flex flex-center" 
                :class="isToday(scope.timestamp.date) ? 'today-badge' : 'text-dark'"
              >
                {{ scope.timestamp.day }}
              </div>
            </div>
          </template>

          <template #day-body="{ scope }">
            <div 
              v-if="isToday(scope.timestamp.date) && isWorkingHours" 
              class="current-time-line"
              :style="{ top: `${currentTimeTop}px` }"
            >
              <div class="time-dot"></div>
            </div>

            <template v-for="ev in getTimedEventsForDate(scope.timestamp.date)" :key="ev.id">
              <div
                class="event-card shadow-2"
                :class="{ 'event-compact': ev.isCompact }"
                :style="getEventStyle(ev)"
                @click.stop="openDetail(ev.appointment)"
              >
                <div v-if="ev.isCompact" class="row items-center no-wrap full-height q-px-xs">
                  <q-avatar :style="`background: ${ev.theme.solid}; color: white`" size="18px" font-size="10px" class="q-mr-xs shadow-1">
                    {{ getInitial(ev.appointment.clientName) }}
                  </q-avatar>
                  <span class="text-weight-bold text-dark q-mr-xs" style="font-size: 10px">{{ ev.appointment.startTime }}</span>
                  <span class="text-dark ellipsis" style="font-size: 11px">{{ getFirstName(ev.appointment.clientName) }}</span>
                </div>
                
                <div v-else class="column full-height q-pa-xs">
                  <div class="row items-center no-wrap q-mb-xs">
                    <q-avatar :style="`background: ${ev.theme.solid}; color: white`" size="20px" font-size="11px" class="q-mr-xs shadow-1">
                      {{ getInitial(ev.appointment.clientName) }}
                    </q-avatar>
                    <div class="text-weight-bold text-dark ellipsis" style="font-size: 0.8rem; line-height: 1;">
                      {{ ev.appointment.clientName }}
                    </div>
                  </div>
                  <div class="text-grey-8 ellipsis" style="font-size: 0.7rem; line-height: 1.1;">
                    {{ ev.appointment.service }}
                  </div>
                  <q-space />
                  <div class="text-grey-6 text-weight-medium" style="font-size: 0.65rem;">
                    {{ ev.appointment.startTime }} - {{ ev.appointment.endTime }}
                  </div>
                </div>

                <q-tooltip transition-show="fade" transition-hide="fade" class="bg-white text-dark shadow-4 border-radius-md" :offset="[10, 10]">
                  <div class="row items-center q-mb-sm">
                    <q-avatar :style="`background: ${ev.theme.solid}; color: white`" size="32px" class="q-mr-sm">
                      {{ getInitial(ev.appointment.clientName) }}
                    </q-avatar>
                    <div>
                      <div class="text-subtitle2 text-weight-bold">{{ ev.appointment.clientName }}</div>
                      <div class="text-caption text-grey-6">{{ ev.appointment.clientPhone }}</div>
                    </div>
                  </div>
                  <q-separator class="q-mb-sm" />
                  <div class="q-mb-xs"><q-icon name="content_cut" class="q-mr-xs text-grey"/> {{ ev.appointment.service }}</div>
                  <div><q-icon name="schedule" class="q-mr-xs text-grey"/> {{ ev.appointment.startTime }} - {{ ev.appointment.endTime }}</div>
                </q-tooltip>
              </div>
            </template>
          </template>
        </q-calendar-day>
      </div>
    </div>

    <q-dialog v-model="detailDialog">
      <q-card style="width: 350px; border-radius: 12px;">
         <q-card-section class="row items-center">
            <div class="text-h6">Detalhes do Agendamento</div>
            <q-space />
            <q-btn icon="close" flat round dense v-close-popup />
         </q-card-section>
         <q-card-section class="q-pt-none text-center text-grey-8">
            (Integração do agendamento aqui)
         </q-card-section>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from "vue";
import { useQuasar } from "quasar";
import { QCalendarDay } from "@quasar/quasar-ui-qcalendar";

// ─── Tipos e Interfaces ───────────────────────────────────────────────────────
type AppointmentStatus = "confirmado" | "pendente" | "cancelado" | "concluido";

interface Appointment {
  id: number;
  clientName: string;
  clientPhone: string;
  status: AppointmentStatus;
  startDate: string;
  endDate: string;
  startTime: string;
  endTime: string;
  service: string;
}

interface ThemeColor {
  solid: string;
  light: string;
}

interface CalendarEventView {
  id: string;
  duration: number;
  theme: ThemeColor;
  appointment: Appointment;
  top?: number;
  height?: number;
  left?: number;
  width?: number;
  isCompact?: boolean;
  zIndex?: number;
  date: string;
}

// ─── Estado da Aplicação ──────────────────────────────────────────────────────
const $q = useQuasar();
const selectedDate = ref("2026-04-20");
const detailDialog = ref(false);
const viewMode = ref<"hourly" | "period">("hourly");

const weekDates = [
  "2026-04-20", "2026-04-21", "2026-04-22", "2026-04-23", 
  "2026-04-24", "2026-04-25", "2026-04-26"
];

// ─── Paleta de Cores ──────────────────────────────────────────────────────────
const SERVICE_THEMES: Record<string, ThemeColor> = {
  "Corte + Escova": { solid: "#3b82f6", light: "#eff6ff" },
  "Barba": { solid: "#0ea5e9", light: "#f0f9ff" },
  "Coloração + Hidratação": { solid: "#a855f7", light: "#faf5ff" },
  "Corte Masculino": { solid: "#10b981", light: "#ecfdf5" },
  "Manicure + Pedicure": { solid: "#f43f5e", light: "#fff1f2" },
  "Retiro / Curso": { solid: "#f59e0b", light: "#fffbeb" },
};

function getServiceTheme(service: string): ThemeColor {
  return SERVICE_THEMES[service] ?? { solid: "#64748b", light: "#f8fafc" };
}

// ─── Dados Fakes (Com Conflitos de Horário) ──────────────────────────────────
const appointments = ref<Appointment[]>([
  // 🔴 GRADE: Choque de Horários para demonstração na Segunda (20/04/2026)
  { id: 1, clientName: "Ana Silva", clientPhone: "(11) 98765-4321", status: "confirmado", startDate: "2026-04-20", endDate: "2026-04-20", startTime: "09:00", endTime: "10:30", service: "Corte + Escova" },
  { id: 2, clientName: "Carlos Mendes", clientPhone: "(11) 91234-5678", status: "pendente", startDate: "2026-04-20", endDate: "2026-04-20", startTime: "09:30", endTime: "11:00", service: "Barba" },
  { id: 3, clientName: "João Pedro", clientPhone: "(11) 97654-3210", status: "cancelado", startDate: "2026-04-20", endDate: "2026-04-20", startTime: "09:15", endTime: "10:00", service: "Corte Masculino" },
  
  // Evento normal em outro dia
  { id: 4, clientName: "Roberto Alves", clientPhone: "(11) 97654-3210", status: "cancelado", startDate: "2026-04-21", endDate: "2026-04-21", startTime: "14:00", endTime: "14:45", service: "Corte Masculino" },
  
  // 🗓️ PERÍODOS: Múltiplos dias
  { id: 5, clientName: "Treinamento Interno", clientPhone: "-", status: "confirmado", startDate: "2026-04-21", endDate: "2026-04-23", startTime: "08:00", endTime: "18:00", service: "Retiro / Curso" },
  { id: 6, clientName: "Reforma Loja", clientPhone: "-", status: "confirmado", startDate: "2026-04-25", endDate: "2026-04-26", startTime: "08:00", endTime: "22:00", service: "Retiro / Curso" },
]);

// ─── Lógica da Linha do Tempo ─────────────────────────────────────────────────
const currentTimeTop = ref(0);
const isWorkingHours = ref(true);
let timeInterval: ReturnType<typeof setInterval>;

function updateCurrentTime() {
  const now = new Date();
  const h = now.getHours();
  const m = now.getMinutes();
  if (h < 8 || h >= 22) {
    isWorkingHours.value = false;
    return;
  }
  isWorkingHours.value = true;
  currentTimeTop.value = ((h - 8) * 65) + ((m / 60) * 65);
}

onMounted(() => {
  updateCurrentTime();
  timeInterval = setInterval(updateCurrentTime, 60000); 
});

onUnmounted(() => { clearInterval(timeInterval); });

// ─── Helpers ──────────────────────────────────────────────────────────────────
function parseTime(timeStr: string): { h: number; m: number } {
  if (!timeStr) return { h: 0, m: 0 };
  const parts = timeStr.split(":");
  return { h: Number(parts[0]) || 0, m: Number(parts[1]) || 0 };
}
function getInitial(name: string): string { return name ? name.charAt(0).toUpperCase() : "?"; }
function getFirstName(fullName: string): string {
  if (!fullName) return "";
  return fullName.trim().split(" ")[0] || fullName;
}
function getDatesInRange(startDate: string, endDate: string): string[] {
  const dates = [];
  let current = new Date(`${startDate}T12:00:00`); 
  const end = new Date(`${endDate}T12:00:00`);
  while (current <= end) {
    dates.push(current.toISOString().split('T')[0]);
    current.setDate(current.getDate() + 1);
  }
  return dates;
}
function getWeekdayName(index: number): string {
  const days = ["Seg", "Ter", "Qua", "Qui", "Sex", "Sáb", "Dom"];
  return days[index] || "";
}

// ─── Motor Matemático de Renderização ─────────────────────────────────────────

const periodEvents = computed(() => {
  return appointments.value
    .filter(app => app.startDate !== app.endDate)
    .flatMap(app => {
      return getDatesInRange(app.startDate, app.endDate).map(date => ({
        id: `${app.id}-${date}`,
        date,
        duration: 0,
        theme: getServiceTheme(app.service),
        appointment: app
      } as CalendarEventView));
    });
});

const timedEvents = computed(() => {
  const sameDayApps = appointments.value.filter(app => app.startDate === app.endDate);
  const byDate: Record<string, typeof sameDayApps> = {};
  
  sameDayApps.forEach(app => {
    if (!byDate[app.startDate]) byDate[app.startDate] = [];
    byDate[app.startDate].push(app);
  });

  const processed: CalendarEventView[] = [];

  for (const date in byDate) {
    let dailyApps = byDate[date];

    // Mapeamento em minutos para fácil verificação de colisão matemática
    const mappedApps = dailyApps.map(app => {
      const start = parseTime(app.startTime);
      const end = parseTime(app.endTime);
      return { 
        app, 
        startMins: start.h * 60 + start.m, 
        endMins: end.h * 60 + end.m 
      };
    });

    // Passo 1: Organizar do mais cedo para o mais tarde
    mappedApps.sort((a, b) => {
      if (a.startMins === b.startMins) return b.endMins - a.endMins;
      return a.startMins - b.startMins;
    });

    // Passo 2: Separar em "Clusters" (Grupos de eventos que se sobrepõem no tempo)
    let clusters: any[][] = [];
    let currentCluster: any[] = [];
    let clusterEnd = 0;

    mappedApps.forEach(item => {
      if (currentCluster.length === 0) {
        currentCluster.push(item);
        clusterEnd = item.endMins;
      } else {
        if (item.startMins < clusterEnd) {
          currentCluster.push(item);
          clusterEnd = Math.max(clusterEnd, item.endMins); // Expande o alcance do cluster
        } else {
          clusters.push(currentCluster);
          currentCluster = [item];
          clusterEnd = item.endMins;
        }
      }
    });
    if (currentCluster.length > 0) clusters.push(currentCluster);

    // Passo 3: Dentro de cada cluster, organizar em Colunas visuais
    clusters.forEach(cluster => {
      const columns: number[] = []; // Guarda a que horas a última tarefa terminou em cada coluna

      cluster.forEach(item => {
        // Tenta achar a primeira coluna cujo evento já terminou antes de este começar
        let colIndex = columns.findIndex(colEndTime => item.startMins >= colEndTime);
        
        if (colIndex === -1) { // Se não tem coluna vazia, cria uma nova
          colIndex = columns.length;
          columns.push(item.endMins);
        } else {
          columns[colIndex] = item.endMins;
        }
        item.colIndex = colIndex;
      });

      const numCols = columns.length;
      // Define a largura baseada na quantidade máxima de eventos concorrentes neste bloco (deixando 1% pra respiro)
      const colWidth = (96 / numCols) - 1; 

      cluster.forEach((item, index) => {
        const durationMins = item.endMins - item.startMins;
        const pixelsPerMinute = 65 / 60; // 65px de altura por hora 

        processed.push({
          id: String(item.app.id),
          date: item.app.startDate,
          duration: durationMins,
          theme: getServiceTheme(item.app.service),
          appointment: item.app,
          top: (item.startMins - (8 * 60)) * pixelsPerMinute, // Base 8h
          height: Math.max(durationMins * pixelsPerMinute, 26), 
          left: (item.colIndex * (96 / numCols)), // Posição calculada baseada na coluna
          width: colWidth, 
          isCompact: durationMins <= 30,
          zIndex: 10 + index
        } as CalendarEventView);
      });
    });
  }

  return processed;
});

function getPeriodEventsForDate(date: string) { return periodEvents.value.filter(ev => ev.date === date); }
function getTimedEventsForDate(date: string) { return timedEvents.value.filter(ev => ev.date === date); }

function getEventStyle(ev: CalendarEventView): Record<string, string> {
  return {
    position: "absolute",
    top: `${ev.top}px`,
    left: `${ev.left}%`,
    width: `${ev.width}%`,
    height: `${ev.height}px`,
    backgroundColor: ev.theme.light,
    borderLeft: `4px solid ${ev.theme.solid}`,
    borderTop: `1px solid ${ev.theme.solid}22`,
    borderRight: `1px solid ${ev.theme.solid}22`,
    borderBottom: `1px solid ${ev.theme.solid}22`,
    borderRadius: "6px",
    zIndex: `${ev.zIndex}`,
    cursor: "pointer",
  };
}

function openDetail(app: Appointment) { detailDialog.value = true; }
function isToday(date: string): boolean { return date === "2026-04-20"; }
</script>

<style scoped>
/* Oculta scrollbars indesejadas no Quasar Calendar */
.scheduler-calendar {
  min-height: 500px;
}

/* ─── CARDS NA VISÃO DE PERÍODO ─── */
.period-card {
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}
.period-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.1) !important;
}

/* ─── EVENTOS NA GRADE DE HORÁRIOS ─── */
.event-card {
  overflow: hidden;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}
.event-card:hover {
  transform: scale(1.02);
  box-shadow: 0 4px 12px rgba(0,0,0,0.15) !important;
  z-index: 50 !important;
}

/* ─── HEADER DIAS ─── */
.today-badge {
  background-color: var(--q-primary);
  color: white;
  width: 36px;
  height: 36px;
  border-radius: 50%;
  box-shadow: 0 2px 4px rgba(0,0,0,0.2);
}

/* ─── LINHA DO TEMPO ─── */
.current-time-line {
  position: absolute;
  left: 0;
  right: 0;
  height: 2px;
  background-color: #ea4335;
  z-index: 100;
  pointer-events: none;
}
.time-dot {
  position: absolute;
  left: -4px;
  top: -4px;
  width: 10px;
  height: 10px;
  background-color: #ea4335;
  border-radius: 50%;
}

/* ─── UTILITÁRIOS ─── */
.border-right { border-right: 1px solid rgba(0,0,0,0.08); }
.border-bottom { border-bottom: 1px solid rgba(0,0,0,0.08); }
</style>