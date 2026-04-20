<template>
  <q-page class="q-pa-md bg-grey-2">
    
    <div class="row items-center justify-between q-mb-md bg-white q-pa-sm rounded-borders shadow-1">
      <div class="text-h6 text-weight-bold text-dark row items-center">
        <q-icon name="groups" class="q-mr-sm" color="primary" />
        Visão de Equipe (Vertical)
      </div>
      <div class="row items-center q-gutter-sm">
        <q-btn flat dense icon="chevron_left" color="primary" />
        <div class="text-subtitle1 text-weight-bold">20 de Abril, 2026</div>
        <q-btn flat dense icon="chevron_right" color="primary" />
      </div>
    </div>

    <div class="calendar-container bg-white rounded-borders shadow-1">
      
      <div class="professionals-header row no-wrap border-bottom">
        <div class="corner-header bg-grey-1 shadow-1 border-right"></div>
        
        <div 
          v-for="staff in professionals" 
          :key="staff.id" 
          class="staff-header-col column items-center justify-center q-pa-sm border-right bg-white"
        >
          <q-avatar size="48px" class="shadow-1 q-mb-xs">
            <img :src="staff.avatar" />
          </q-avatar>
          <div class="text-subtitle2 text-weight-bold text-dark ellipsis text-center" style="width: 100%;">
            {{ staff.name }}
          </div>
          <div class="text-caption text-grey-6 text-uppercase" style="font-size: 10px;">
            {{ staff.role }}
          </div>
        </div>
      </div>

      <div class="calendar-body row no-wrap relative-position">
        
        <div class="time-col column bg-grey-1 border-right shadow-1">
          <div 
            v-for="hour in timeSlots" 
            :key="hour" 
            class="time-slot row items-start justify-center q-pt-sm text-caption text-weight-bold text-grey-7"
          >
            {{ hour }}:00
          </div>
        </div>

        <div class="events-area row no-wrap relative-position">
          
          <div class="grid-lines column absolute-full pointer-events-none">
            <div v-for="hour in timeSlots" :key="`grid-${hour}`" class="hour-grid-line border-bottom"></div>
          </div>

          <div 
            v-for="staff in professionals" 
            :key="`col-${staff.id}`" 
            class="staff-column relative-position border-right"
          >
            <template v-for="ev in getEventsForStaff(staff.id)" :key="ev.id">
              <div
                class="event-block shadow-2 transition-generic cursor-pointer flex column"
                :style="getEventStyle(ev)"
                @click="openDetail(ev.appointment)"
              >
                <div class="row no-wrap full-height q-pa-sm">
                  <div class="status-indicator q-mr-sm" :style="`background-color: ${ev.color}`"></div>
                  
                  <div class="column overflow-hidden">
                    <div class="text-weight-bold text-white ellipsis" style="font-size: 0.8rem; line-height: 1.2;">
                      {{ ev.appointment.clientName }}
                    </div>
                    <div class="text-white text-opacity-80 ellipsis" style="font-size: 0.7rem; margin-top: 2px;">
                      {{ ev.appointment.service }}
                    </div>
                    <div class="text-white text-opacity-80 ellipsis" style="font-size: 0.65rem;">
                      {{ ev.appointment.startTime }} - {{ ev.appointment.endTime }}
                    </div>
                  </div>
                </div>

                <q-tooltip transition-show="scale" transition-hide="scale" class="bg-dark text-white shadow-4" :offset="[0, 5]">
                  <div class="text-subtitle2 text-weight-bold">{{ ev.appointment.service }}</div>
                  <div><q-icon name="person" class="q-mr-xs"/> {{ ev.appointment.clientName }}</div>
                  <div><q-icon name="schedule" class="q-mr-xs"/> {{ ev.appointment.startTime }} - {{ ev.appointment.endTime }}</div>
                </q-tooltip>
              </div>
            </template>
          </div>

        </div>
      </div>

    </div>
  </q-page>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";

// ─── Tipos e Interfaces ───────────────────────────────────────────────────────
interface Professional {
  id: number;
  name: string;
  role: string;
  avatar: string;
}

interface Appointment {
  id: number;
  staffId: number;
  clientName: string;
  startTime: string;
  endTime: string;
  service: string;
  status: string;
}

interface TimelineEvent {
  id: number;
  appointment: Appointment;
  top: number;    // Eixo Y: Topo em pixels
  height: number; // Eixo Y: Altura em pixels
  color: string;
}

// ─── Configurações de Escala (Engenharia do Layout Vertical) ──────────────────
const START_HOUR = 8; // Começa às 08:00
const END_HOUR = 20;  // Termina às 20:00
const PIXELS_PER_HOUR = 120; // Altura de cada hora. 1 hora = 120px (portanto, 1 min = 2px)
const PIXELS_PER_MINUTE = PIXELS_PER_HOUR / 60;

// Gera o array de horas [8, 9, 10... 19]
const timeSlots = Array.from({ length: END_HOUR - START_HOUR }, (_, i) => START_HOUR + i);

// Cores por serviço
const SERVICE_COLORS: Record<string, string> = {
  "Corte Masculino": "#26A69A", // Teal
  "Barba": "#29B6F6",           // Light Blue
  "Coloração": "#AB47BC",       // Purple
  "Manicure": "#EC407A",        // Pink
};

// ─── Dados Fakes ──────────────────────────────────────────────────────────────
const professionals = ref<Professional[]>([
  { id: 1, name: "Lucas Barber", role: "Barbeiro Sênior", avatar: "https://cdn.quasar.dev/img/avatar1.jpg" },
  { id: 2, name: "Ana Stylist", role: "Colorista", avatar: "https://cdn.quasar.dev/img/avatar2.jpg" },
  { id: 3, name: "Pedro Fade", role: "Barbeiro", avatar: "https://cdn.quasar.dev/img/avatar3.jpg" },
  { id: 4, name: "Julia Nails", role: "Manicure", avatar: "https://cdn.quasar.dev/img/avatar4.jpg" },
]);

const appointments = ref<Appointment[]>([
  { id: 1, staffId: 1, clientName: "Carlos Silva", startTime: "08:30", endTime: "09:30", service: "Corte Masculino", status: "confirmado" },
  { id: 2, staffId: 1, clientName: "Roberto Alves", startTime: "10:00", endTime: "10:45", service: "Barba", status: "pendente" },
  { id: 3, staffId: 2, clientName: "Mariana Costa", startTime: "09:00", endTime: "12:00", service: "Coloração", status: "confirmado" },
  { id: 4, staffId: 3, clientName: "João Pedro", startTime: "08:00", endTime: "08:45", service: "Corte Masculino", status: "concluido" },
  { id: 5, staffId: 3, clientName: "Marcos V.", startTime: "14:00", endTime: "15:00", service: "Corte Masculino", status: "confirmado" },
  { id: 6, staffId: 4, clientName: "Fernanda Lima", startTime: "10:30", endTime: "11:30", service: "Manicure", status: "confirmado" },
  { id: 7, staffId: 4, clientName: "Beatriz Oliveira", startTime: "13:00", endTime: "14:30", service: "Manicure", status: "pendente" },
]);

// ─── Motor Matemático do Calendário (Vertical) ───────────────────────────────

function parseTime(timeStr: string): { h: number; m: number } {
  const parts = timeStr.split(":");
  return { h: Number(parts[0]) || 0, m: Number(parts[1]) || 0 };
}

const processedEvents = computed(() => {
  return appointments.value.map(app => {
    const start = parseTime(app.startTime);
    const end = parseTime(app.endTime);
    
    // Cálculo do Eixo Y (Top)
    const minutesFromStart = ((start.h - START_HOUR) * 60) + start.m;
    const topPx = minutesFromStart * PIXELS_PER_MINUTE;
    
    // Cálculo da Altura (Height)
    const durationMinutes = ((end.h * 60) + end.m) - ((start.h * 60) + start.m);
    const heightPx = durationMinutes * PIXELS_PER_MINUTE;

    return {
      id: app.id,
      appointment: app,
      top: topPx,
      height: heightPx,
      color: SERVICE_COLORS[app.service] || "#546E7A"
    } as TimelineEvent;
  });
});

// ─── Controladores da View ───────────────────────────────────────────────────

function getEventsForStaff(staffId: number): TimelineEvent[] {
  return processedEvents.value.filter(ev => ev.appointment.staffId === staffId);
}

function getEventStyle(ev: TimelineEvent): Record<string, string> {
  return {
    position: "absolute",
    top: `${ev.top}px`,
    height: `${ev.height}px`,
    left: "5px",            // Margem lateral esquerda
    right: "5px",           // Margem lateral direita
    width: "calc(100% - 10px)", // Preenche a coluna respeitando a margem
    backgroundColor: "#34495e", 
    borderRadius: "8px",
    zIndex: "5"
  };
}

function openDetail(app: Appointment) {
  console.log("Abrindo detalhes de:", app);
}
</script>

<style scoped>
/* Container Principal - Habilita rolagem vertical e horizontal */
.calendar-container {
  overflow: auto;
  max-height: 75vh; /* Limita a altura para que a página em si não role inteira (ajuste se necessário) */
  position: relative;
}

/* --- Eixos Fixos (Sticky) e Expansão --- */

/* Garante que o corpo do calendário abraçe todos os profissionais, 
   ou expanda para preencher a tela se forem poucos */
.professionals-header, .calendar-body {
  min-width: 100%;
  width: max-content;
}

/* Cabeçalho de Profissionais Fixo no Topo */
.professionals-header {
  position: sticky;
  top: 0;
  z-index: 20;
}

/* Coluna de Horas Fixa na Esquerda */
.time-col {
  position: sticky;
  left: 0;
  width: 70px;
  min-width: 70px;
  z-index: 10;
}

/* Célula Superior Esquerda (Interseção das áreas fixas) */
.corner-header {
  position: sticky;
  left: 0;
  top: 0;
  width: 70px;
  min-width: 70px;
  z-index: 30; /* Tem que ficar acima do topo e da lateral */
}

/* --- Corpo da Grade --- */

/* A área de eventos ocupa o restante do espaço ao lado das horas */
.events-area {
  flex: 1;
  height: calc(12 * 120px); /* (END_HOUR - START_HOUR) * PIXELS_PER_HOUR */
}

/* Largura flexível da coluna: preenche o vazio se poucos, ou mantém 220px se muitos */
.staff-header-col, .staff-column {
  flex: 1 1 220px;
  min-width: 220px;
}

/* Altura fixa para as divisões de hora */
.time-slot, .hour-grid-line {
  height: 120px; /* PIXELS_PER_HOUR */
  flex-shrink: 0;
}

/* Linhas separadoras */
.border-right { border-right: 1px solid rgba(0,0,0,0.08); }
.border-bottom { border-bottom: 1px solid rgba(0,0,0,0.08); }

/* Estilo do Bloco de Agendamento */
.event-block {
  overflow: hidden;
}
.event-block:hover {
  transform: scale(1.02);
  filter: brightness(1.1);
  box-shadow: 0 4px 12px rgba(0,0,0,0.2) !important;
  z-index: 15 !important;
}

/* Barra colorida lateral do evento */
.status-indicator {
  width: 4px;
  height: 100%;
  border-radius: 4px;
  flex-shrink: 0;
}

/* Utilitários */
.transition-generic { transition: all 0.2s ease; }
.pointer-events-none { pointer-events: none; }
</style>