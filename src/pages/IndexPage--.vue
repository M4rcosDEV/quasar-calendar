<template>
  <q-page class="row">
    <div class="col column">
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
          <template #head-day-label="{ scope }">
            <div class="column items-center q-py-xs">
              <div class="text-h6 text-weight-bold" :class="isToday(scope.timestamp.date) ? 'text-primary' : 'text-grey-8'">
                {{ scope.timestamp.day }}
              </div>
              <q-badge v-if="getCount(scope.timestamp.date) > 0" color="primary" :label="getCount(scope.timestamp.date)" style="font-size: 9px" />
            </div>
          </template>

          <template #day-body="{ scope }">
            <template v-for="ev in getEventsForDate(scope.timestamp.date)" :key="ev.id">
              <div
                class="event-block"
                :style="getEventStyle(ev)"
                @click.stop="openDetail(ev.appointment)"
              >
                <div v-if="ev.isCompact" class="event-compact-content">
                  <span class="text-weight-bold q-mr-xs">{{ ev.appointment.startTime }}</span>
                  <span class="ellipsis">{{ getFirstName(ev.appointment.clientName) }}</span>
                </div>
                
                <div v-else class="event-content">
                  <div class="event-service text-weight-bold ellipsis">{{ ev.appointment.service }}</div>
                  <div class="event-client ellipsis"><q-icon name="person" size="11px" /> {{ ev.appointment.clientName }}</div>
                </div>

                <q-tooltip transition-show="scale" transition-hide="scale" class="bg-dark text-white shadow-4" :offset="[10, 10]">
                  <div class="text-subtitle2 text-weight-bold">{{ ev.appointment.service }}</div>
                  <div class="q-my-xs"><q-icon name="person" class="q-mr-xs"/> {{ ev.appointment.clientName }}</div>
                  <div><q-icon name="schedule" class="q-mr-xs"/> {{ ev.appointment.startTime }} - {{ ev.appointment.endTime }}</div>
                  <q-badge class="q-mt-sm" :color="STATUS_CONFIG[ev.appointment.status].color" :label="STATUS_CONFIG[ev.appointment.status].label" />
                </q-tooltip>
              </div>
            </template>
          </template>
        </q-calendar-day>
      </div>
    </div>

    <q-dialog v-model="detailDialog" :maximized="$q.screen.lt.sm">
      <q-card style="width: 400px; max-width: 80vw;">
        <q-card-section class="row items-center q-pb-none">
          <div class="text-h6">Detalhes do Agendamento</div>
          <q-space />
          <q-btn icon="close" flat round dense v-close-popup />
        </q-card-section>

        <q-card-section v-if="selected" class="q-pt-md">
          <p><strong>Cliente:</strong> {{ selected.clientName }}</p>
          <p><strong>Serviço:</strong> {{ selected.service }}</p>
          <p><strong>Horário:</strong> {{ selected.startTime }} - {{ selected.endTime }}</p>
          <p><strong>Status:</strong> <q-badge :color="STATUS_CONFIG[selected.status].color">{{ STATUS_CONFIG[selected.status].label }}</q-badge></p>
        </q-card-section>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import { useQuasar } from "quasar";
import { QCalendarDay } from "@quasar/quasar-ui-qcalendar";

// ─── Tipos e Interfaces ───────────────────────────────────────────────────────
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

interface CalendarEventView {
  id: number;
  duration: number;
  bgcolor: string;
  appointment: Appointment;
  top: number;
  height: number;
  left: number;
  width: number;
  isCompact: boolean;
  zIndex: number;
}

// ─── Configurações Constantes ─────────────────────────────────────────────────
const STATUS_CONFIG: Record<AppointmentStatus, { label: string; color: string }> = {
  confirmado: { label: "Confirmado", color: "positive" },
  pendente: { label: "Pendente", color: "warning" },
  cancelado: { label: "Cancelado", color: "negative" },
  concluido: { label: "Concluído", color: "blue-grey-6" },
};

const SERVICE_COLORS: Record<string, string> = {
  "Corte + Escova": "#1565C0",
  "Barba": "#0277BD",
  "Coloração + Hidratação": "#7B1FA2",
  "Corte Masculino": "#00695C",
  "Manicure + Pedicure": "#C62828",
  "Corte + Barba": "#1B5E20",
  "Progressiva": "#E65100",
  "Barba Modelagem": "#004D40",
  "Escova + Penteado": "#880E4F",
  "Corte Degradê": "#01579B",
  "Manutenção Coloração": "#4A148C",
  "Sobrancelha + Cílios": "#BF360C",
};

function getServiceColor(service: string): string {
  return SERVICE_COLORS[service] ?? "#546E7A";
}

// ─── Estado da Aplicação ──────────────────────────────────────────────────────
const $q = useQuasar();
const selectedDate = ref("2026-04-20");
const detailDialog = ref(false);
const selected = ref<Appointment | null>(null);

const appointments = ref<Appointment[]>([
  { id: 1, clientId: 1, clientName: "Ana Silva", clientPhone: "(11) 98765-4321", status: "confirmado", date: "2026-04-20", startTime: "09:00", endTime: "10:00", service: "Corte + Escova", notes: "Prefere corte em camadas" },
  { id: 2, clientId: 2, clientName: "Carlos Mendes", clientPhone: "(11) 91234-5678", status: "pendente", date: "2026-04-20", startTime: "10:30", endTime: "10:45", service: "Barba", notes: "" },
  { id: 3, clientId: 3, clientName: "Mariana Costa", clientPhone: "(21) 99876-5432", status: "confirmado", date: "2026-04-21", startTime: "14:00", endTime: "16:30", service: "Coloração + Hidratação", notes: "Loiro mel" },
  { id: 4, clientId: 4, clientName: "Roberto Alves", clientPhone: "(11) 97654-3210", status: "cancelado", date: "2026-04-21", startTime: "09:00", endTime: "09:50", service: "Corte Masculino", notes: "Cancelado" },
  { id: 5, clientId: 5, clientName: "Fernanda Lima", clientPhone: "(31) 98888-7777", status: "concluido", date: "2026-04-22", startTime: "11:00", endTime: "12:30", service: "Manicure + Pedicure", notes: "Esmalte clássico" },
  { id: 6, clientId: 6, clientName: "Paulo Santos", clientPhone: "(11) 96666-5555", status: "confirmado", date: "2026-04-22", startTime: "14:00", endTime: "15:00", service: "Corte + Barba", notes: "" }
]);

// ─── Helpers de Formatação ────────────────────────────────────────────────────

function parseTime(timeStr: string): { h: number; m: number } {
  if (!timeStr) return { h: 0, m: 0 };
  const parts = timeStr.split(":");
  return {
    h: Number(parts[0]) || 0,
    m: Number(parts[1]) || 0
  };
}

// 🟢 NOVO: Pega apenas o primeiro nome para caber no bloco pequeno
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
    
    const duration = (end.h * 60 + end.m) - (start.h * 60 + start.m);
    const top = (start.h - 8) * 60 + start.m;

    return {
      id: app.id,
      duration,
      bgcolor: getServiceColor(app.service),
      appointment: app,
      top,
      height: Math.max(duration, 24), // Ajustei para 24px para encaixar perfeitamente 1 linha
      left: 0, 
      width: 98,
      isCompact: duration <= 30,
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
    backgroundColor: ev.bgcolor,
    borderRadius: "6px",
    padding: "0", // Padding zerado no container, controlado pelo conteúdo
    zIndex: `${ev.zIndex}`,
    boxShadow: "0 2px 4px rgba(0,0,0,0.15)",
    cursor: "pointer",
    transition: "all 0.2s ease"
  };
}

function openDetail(app: Appointment) {
  selected.value = app;
  detailDialog.value = true;
}

function isToday(date: string): boolean {
  const t = new Date();
  const todayStr = `${t.getFullYear()}-${String(t.getMonth() + 1).padStart(2, "0")}-${String(t.getDate()).padStart(2, "0")}`;
  return date === todayStr;
}

function getCount(date: string): number {
  return appointments.value.filter((a) => a.date === date).length;
}
</script>

<style scoped>
.event-block {
  border-left: 4px solid rgba(0, 0, 0, 0.2);
  overflow: hidden;
  color: white;
}

/* 🟢 NOVO: Layout em linha única, texto pequeno, oculta o que não couber */
.event-compact-content {
  display: flex;
  flex-direction: row;
  align-items: center;
  padding: 0 4px;
  height: 100%;
  width: 100%;
  font-size: 0.7rem; /* Aproximadamente 11px */
  line-height: 1;
  white-space: nowrap;
}

.event-content {
  padding: 4px 6px;
}

.event-service {
  font-size: 0.8rem;
  line-height: 1.1;
  margin-bottom: 2px;
}

.event-client {
  font-size: 0.75rem;
  opacity: 0.9;
}

.scheduler-calendar {
  height: calc(100vh - 50px);
  min-height: 600px;
}
</style>