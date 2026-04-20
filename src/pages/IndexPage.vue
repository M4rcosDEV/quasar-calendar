<template>
  <q-page class="row bg-grey-1">
    <div class="col column q-pa-md">
      
      <div class="row items-center justify-between q-mb-md bg-white q-pa-sm rounded-borders shadow-1">
        <div class="text-h6 text-weight-bold text-dark row items-center">
          <q-icon name="calendar_month" class="q-mr-sm" color="primary" />
          Agenda Semanal
        </div>
        <div class="text-subtitle2 text-grey-6">
          Semana de 20 de Abril, 2026
        </div>
      </div>

      <div class="col bg-white rounded-borders shadow-1 overflow-hidden flex">
        <q-calendar-day
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
          class="scheduler-calendar col"
        >
          <template #head-day-label="{ scope }">
            <div class="column items-center q-py-sm">
              <div class="text-caption text-uppercase text-weight-bold text-grey-5">{{ scope.timestamp.weekdayName }}</div>
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

            <template v-for="ev in getEventsForDate(scope.timestamp.date)" :key="ev.id">
              <div
                class="event-card"
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
            <div class="text-h6">Agendamento</div>
            <q-space />
            <q-btn icon="close" flat round dense v-close-popup />
         </q-card-section>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from "vue";
import { useQuasar } from "quasar";
import { QCalendarDay } from "@quasar/quasar-ui-qcalendar";

// ─── Tipos ────────────────────────────────────────────────────────────────────
type AppointmentStatus = "confirmado" | "pendente" | "cancelado" | "concluido";

interface Appointment {
  id: number;
  clientName: string;
  clientPhone: string;
  status: AppointmentStatus;
  date: string;
  startTime: string;
  endTime: string;
  service: string;
}

interface ThemeColor {
  solid: string;
  light: string;
}

interface CalendarEventView {
  id: number;
  duration: number;
  theme: ThemeColor;
  appointment: Appointment;
  top: number;
  height: number;
  left: number;
  width: number;
  isCompact: boolean;
  zIndex: number;
}

// ─── Paleta de Cores (Estilo Soft/Pastel) ─────────────────────────────────────
const SERVICE_THEMES: Record<string, ThemeColor> = {
  "Corte + Escova": { solid: "#3b82f6", light: "#eff6ff" }, // Azul
  "Barba": { solid: "#0ea5e9", light: "#f0f9ff" },          // Cyan
  "Coloração + Hidratação": { solid: "#a855f7", light: "#faf5ff" }, // Roxo
  "Corte Masculino": { solid: "#10b981", light: "#ecfdf5" }, // Verde
  "Manicure + Pedicure": { solid: "#f43f5e", light: "#fff1f2" }, // Rosa/Vermelho
};

function getServiceTheme(service: string): ThemeColor {
  return SERVICE_THEMES[service] ?? { solid: "#64748b", light: "#f8fafc" }; // Slate default
}

// ─── Estado da Aplicação ──────────────────────────────────────────────────────
const $q = useQuasar();
const selectedDate = ref("2026-04-20");
const detailDialog = ref(false);

const appointments = ref<Appointment[]>([
  { id: 1, clientName: "Ana Silva", clientPhone: "(11) 98765-4321", status: "confirmado", date: "2026-04-20", startTime: "09:00", endTime: "10:00", service: "Corte + Escova" },
  { id: 2, clientName: "Carlos Mendes", clientPhone: "(11) 91234-5678", status: "pendente", date: "2026-04-20", startTime: "10:30", endTime: "10:45", service: "Barba" },
  { id: 3, clientName: "Mariana Costa", clientPhone: "(21) 99876-5432", status: "confirmado", date: "2026-04-21", startTime: "14:00", endTime: "16:30", service: "Coloração + Hidratação" },
  { id: 4, clientName: "Roberto Alves", clientPhone: "(11) 97654-3210", status: "cancelado", date: "2026-04-21", startTime: "09:00", endTime: "09:30", service: "Corte Masculino" },
  { id: 5, clientName: "Fernanda Lima", clientPhone: "(31) 98888-7777", status: "concluido", date: "2026-04-22", startTime: "11:00", endTime: "12:30", service: "Manicure + Pedicure" },
]);

// ─── Lógica da Linha do Tempo (Current Time) ──────────────────────────────────
// Neste exemplo, usamos a data e hora atual do sistema do usuário
const currentTimeTop = ref(0);
const isWorkingHours = ref(true);
let timeInterval: ReturnType<typeof setInterval>;

function updateCurrentTime() {
  const now = new Date();
  const h = now.getHours();
  const m = now.getMinutes();
  
  // Se estiver fora do horário de funcionamento (antes das 8h ou depois das 22h), esconde
  if (h < 8 || h >= 22) {
    isWorkingHours.value = false;
    return;
  }
  
  isWorkingHours.value = true;
  // O calendário começa às 8h. Cada hora tem a altura configurada em :interval-height (65px)
  // Cálculo: (Hora atual - 8) * 65 + (Minutos / 60) * 65
  currentTimeTop.value = ((h - 8) * 65) + ((m / 60) * 65);
}

onMounted(() => {
  updateCurrentTime();
  timeInterval = setInterval(updateCurrentTime, 60000); // Atualiza a cada 1 minuto
});

onUnmounted(() => {
  clearInterval(timeInterval);
});

// ─── Helpers de Formatação ────────────────────────────────────────────────────
function parseTime(timeStr: string): { h: number; m: number } {
  if (!timeStr) return { h: 0, m: 0 };
  const parts = timeStr.split(":");
  return { h: Number(parts[0]) || 0, m: Number(parts[1]) || 0 };
}

function getInitial(name: string): string {
  return name ? name.charAt(0).toUpperCase() : "?";
}

function getFirstName(fullName: string): string {
  if (!fullName) return "";

  let name = fullName.trim().split(" ")[0];
  return name || fullName; // Retorna o primeiro nome ou o nome completo se for apenas um
}

// ─── Motor de Layout (Computado) ──────────────────────────────────────────────
const processedEvents = computed(() => {
  return appointments.value.map((app, index) => {
    const start = parseTime(app.startTime);
    const end = parseTime(app.endTime);
    
    const durationMins = (end.h * 60 + end.m) - (start.h * 60 + start.m);
    
    // Atenção: O Multiplicador deve ser (65 / 60) porque o interval-height agora é 65
    // Para manter a proporção exata dos minutos em pixels:
    const pixelsPerMinute = 65 / 60; 
    
    const duration = durationMins * pixelsPerMinute;
    const top = ((start.h - 8) * 60 + start.m) * pixelsPerMinute;

    return {
      id: app.id,
      duration: durationMins,
      theme: getServiceTheme(app.service),
      appointment: app,
      top,
      height: Math.max(duration, 26), // Altura mínima garantida
      left: 0, 
      width: 96, // Deixa um pequeno respiro do lado direito
      isCompact: durationMins <= 30,
      zIndex: 10 + index
    } as CalendarEventView;
  });
});

// ─── Funções de Controle da View ──────────────────────────────────────────────
function getEventsForDate(date: string): CalendarEventView[] {
  return processedEvents.value.filter((ev) => ev.appointment.date === date);
}

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
    transition: "transform 0.2s ease, box-shadow 0.2s ease"
  };
}

function openDetail(app: Appointment) {
  detailDialog.value = true;
}

function isToday(date: string): boolean {
  // Para fins de teste com seus dados de "2026", estou forçando o dia "2026-04-20" a ser "hoje"
  // Em produção, use a lógica real comentada abaixo:
  /*
  const t = new Date();
  const todayStr = `${t.getFullYear()}-${String(t.getMonth() + 1).padStart(2, "0")}-${String(t.getDate()).padStart(2, "0")}`;
  return date === todayStr;
  */
  return date === "2026-04-20"; 
}
</script>

<style scoped>
/* Oculta scrollbars indesejadas no Quasar Calendar */
.scheduler-calendar {
  height: calc(100vh - 120px);
  min-height: 600px;
}

/* Animação de Hover nos Cards */
.event-card {
  overflow: hidden;
}
.event-card:hover {
  transform: scale(1.01) translateX(2px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.1) !important;
  z-index: 50 !important;
}

/* Estilo do dia atual no cabeçalho */
.today-badge {
  background-color: var(--q-primary);
  color: white;
  width: 36px;
  height: 36px;
  border-radius: 50%;
  box-shadow: 0 2px 4px rgba(0,0,0,0.2);
}

/* 🔴 LINHA DO TEMPO (Current Time Indicator) */
.current-time-line {
  position: absolute;
  left: 0;
  right: 0;
  height: 2px;
  background-color: #ea4335; /* Vermelho estilo Google */
  z-index: 100;
  pointer-events: none; /* Ignora cliques para não atrapalhar os eventos */
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
</style>