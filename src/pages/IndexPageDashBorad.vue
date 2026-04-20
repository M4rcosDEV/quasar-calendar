<template>
  <q-page class="q-pa-md bg-grey-2">
    
    <div class="row items-center justify-between q-mb-lg">
      <div>
        <div class="text-h5 text-weight-bold text-dark">Bom dia, Lucas! 👋</div>
        <div class="text-subtitle2 text-grey-6">Aqui está o resumo da sua operação para hoje, {{ todayFormatted }}.</div>
      </div>
      <q-btn color="primary" icon="add" label="Novo Agendamento" unelevated class="border-radius-md" />
    </div>

    <div class="row q-col-gutter-md q-mb-lg">
      <div class="col-12 col-sm-6 col-md-3">
        <q-card flat bordered class="kpi-card border-radius-md">
          <q-card-section>
            <div class="row items-center justify-between">
              <div class="text-grey-7 text-weight-medium">Agendamentos Hoje</div>
              <q-avatar size="38px" color="blue-1" text-color="primary" icon="calendar_today" />
            </div>
            <div class="text-h4 text-weight-bolder q-mt-sm">{{ dailyMetrics.total }}</div>
            <div class="text-caption text-positive row items-center q-mt-xs">
              <q-icon name="trending_up" class="q-mr-xs" /> +12% em relação a ontem
            </div>
          </q-card-section>
        </q-card>
      </div>

      <div class="col-12 col-sm-6 col-md-3">
        <q-card flat bordered class="kpi-card border-radius-md">
          <q-card-section>
            <div class="row items-center justify-between">
              <div class="text-grey-7 text-weight-medium">Faturamento Estimado</div>
              <q-avatar size="38px" color="green-1" text-color="positive" icon="attach_money" />
            </div>
            <div class="text-h4 text-weight-bolder q-mt-sm">R$ {{ dailyMetrics.revenue.toFixed(2) }}</div>
            <div class="text-caption text-grey-6 row items-center q-mt-xs">
              Baseado nos serviços confirmados
            </div>
          </q-card-section>
        </q-card>
      </div>

      <div class="col-12 col-sm-6 col-md-3">
        <q-card flat bordered class="kpi-card border-radius-md">
          <q-card-section>
            <div class="row items-center justify-between">
              <div class="text-grey-7 text-weight-medium">Concluídos</div>
              <q-avatar size="38px" color="teal-1" text-color="teal" icon="done_all" />
            </div>
            <div class="text-h4 text-weight-bolder q-mt-sm">{{ dailyMetrics.completed }}</div>
            <div class="text-caption text-grey-6 row items-center q-mt-xs">
              {{ dailyMetrics.pending }} aguardando atendimento
            </div>
          </q-card-section>
        </q-card>
      </div>

      <div class="col-12 col-sm-6 col-md-3">
        <q-card flat bordered class="kpi-card border-radius-md">
          <q-card-section>
            <div class="row items-center justify-between">
              <div class="text-grey-7 text-weight-medium">Cancelamentos</div>
              <q-avatar size="38px" color="red-1" text-color="negative" icon="cancel" />
            </div>
            <div class="text-h4 text-weight-bolder q-mt-sm">{{ dailyMetrics.canceled }}</div>
            <div class="text-caption text-negative row items-center q-mt-xs">
              <q-icon name="warning" class="q-mr-xs" /> Atenção: Taxa alta hoje
            </div>
          </q-card-section>
        </q-card>
      </div>
    </div>

    <div class="row q-col-gutter-md">
      
      <div class="col-12 col-md-8 column">
        <q-card flat bordered class="border-radius-md col flex column">
          <q-card-section class="row items-center justify-between bg-white z-top" style="border-bottom: 1px solid #f0f0f0;">
            <div class="text-h6 text-weight-bold text-dark">Próximos Clientes</div>
            <q-btn flat color="primary" label="Ver Agenda Completa" to="/agenda" />
          </q-card-section>
          
          <q-card-section class="q-pa-none flex-auto">
            <q-list separator>
              <q-item v-for="app in upcomingAppointments" :key="app.id" class="q-py-md hover-grey transition-generic">
                
                <q-item-section avatar>
                  <div class="column items-center">
                    <div class="text-weight-bold text-dark">{{ app.startTime }}</div>
                    <q-badge :color="getUrgencyColor(app.startTime)" rounded class="q-mt-xs" />
                  </div>
                </q-item-section>

                <q-item-section>
                  <q-item-label class="text-weight-bold text-subtitle1">{{ app.clientName }}</q-item-label>
                  <q-item-label caption class="text-grey-8">
                    <q-icon name="content_cut" size="xs" class="q-mr-xs"/> {{ app.service }} 
                    <span class="q-mx-sm">•</span> 
                    <q-icon name="person" size="xs" class="q-mr-xs"/> {{ app.staffName }}
                  </q-item-label>
                </q-item-section>

                <q-item-section side class="row no-wrap items-center">
                  <q-btn outline color="positive" icon="check" size="sm" class="q-mr-sm" label="Chegou" v-if="app.status === 'confirmado'" />
                  <q-btn outline color="warning" icon="access_time" size="sm" class="q-mr-sm" label="Atrasado" v-if="app.status === 'confirmado'" />
                  
                  <q-btn flat round color="grey-6" icon="more_vert">
                    <q-menu>
                      <q-list style="min-width: 150px">
                        <q-item clickable v-close-popup>
                          <q-item-section avatar><q-icon name="edit" color="primary" /></q-item-section>
                          <q-item-section>Editar</q-item-section>
                        </q-item>
                        <q-item clickable v-close-popup>
                          <q-item-section avatar><q-icon name="cancel" color="negative" /></q-item-section>
                          <q-item-section>Cancelar</q-item-section>
                        </q-item>
                      </q-list>
                    </q-menu>
                  </q-btn>
                </q-item-section>
              </q-item>

              <div v-if="upcomingAppointments.length === 0" class="text-center q-pa-xl text-grey-5">
                <q-icon name="sentiment_satisfied" size="48px" />
                <div class="text-subtitle1 q-mt-sm">Fila limpa! Nenhum cliente aguardando.</div>
              </div>
            </q-list>
          </q-card-section>
        </q-card>
      </div>

      <div class="col-12 col-md-4 column q-gutter-y-md">
        
        <q-card flat bordered class="border-radius-md bg-primary text-white text-center q-pa-md">
          <div class="text-subtitle1 text-weight-medium q-mb-md">Meta Diária (R$ 1.500)</div>
          <q-circular-progress
            show-value
            font-size="24px"
            :value="goalPercentage"
            size="120px"
            :thickness="0.15"
            color="white"
            track-color="blue-8"
            class="q-mb-sm text-weight-bold"
          >
            {{ goalPercentage }}%
          </q-circular-progress>
          <div class="text-caption text-blue-2 q-mt-sm">Faltam R$ {{ (1500 - dailyMetrics.revenue).toFixed(2) }} para bater a meta!</div>
        </q-card>

        <q-card flat bordered class="border-radius-md flex-auto">
          <q-date
            v-model="selectedDate"
            flat
            class="full-width no-border-radius"
            color="primary"
            minimal
          />
        </q-card>

      </div>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";

// ─── Tipos e Dados Fakes ──────────────────────────────────────────────────────
interface Appointment {
  id: number;
  clientName: string;
  staffName: string;
  status: "pendente" | "confirmado" | "concluido" | "cancelado";
  date: string;
  startTime: string;
  service: string;
  price: number;
}

const selectedDate = ref("2026-04-20");
const todayFormatted = ref("20 de Abril de 2026");

const appointments = ref<Appointment[]>([
  { id: 1, clientName: "Carlos Silva", staffName: "Lucas Barber", status: "concluido", date: "2026-04-20", startTime: "08:30", service: "Corte Masculino", price: 50 },
  { id: 2, clientName: "Roberto Alves", staffName: "Lucas Barber", status: "cancelado", date: "2026-04-20", startTime: "10:00", service: "Barba", price: 35 },
  { id: 3, clientName: "Mariana Costa", staffName: "Ana Stylist", status: "confirmado", date: "2026-04-20", startTime: "14:00", service: "Coloração", price: 250 },
  { id: 4, clientName: "João Pedro", staffName: "Pedro Fade", status: "confirmado", date: "2026-04-20", startTime: "14:30", service: "Corte + Barba", price: 80 },
  { id: 5, clientName: "Fernanda Lima", staffName: "Julia Nails", status: "pendente", date: "2026-04-20", startTime: "15:00", service: "Manicure", price: 40 },
  { id: 6, clientName: "Marcos V.", staffName: "Lucas Barber", status: "confirmado", date: "2026-04-20", startTime: "16:00", service: "Corte Masculino", price: 50 },
]);

// ─── Engenharia de Dados (Computed Properties) ────────────────────────────────

// 1. Cálculos de KPI (Reativos)
const dailyMetrics = computed(() => {
  const todayApps = appointments.value.filter(a => a.date === selectedDate.value);
  
  let total = todayApps.length;
  let completed = 0;
  let canceled = 0;
  let pending = 0;
  let revenue = 0;

  todayApps.forEach(app => {
    if (app.status === 'concluido') {
      completed++;
      revenue += app.price; // Já pagou
    } else if (app.status === 'cancelado') {
      canceled++;
    } else if (app.status === 'confirmado' || app.status === 'pendente') {
      pending++;
      revenue += app.price; // Previsão de entrada
    }
  });

  return { total, completed, canceled, pending, revenue };
});

// 2. Cálculo da Meta (Exemplo: Meta de 1500 reais)
const goalPercentage = computed(() => {
  const goal = 1500;
  const percentage = (dailyMetrics.value.revenue / goal) * 100;
  return Math.min(Math.round(percentage), 100); // Trava em 100% max na UI
});

// 3. Fila de "Próximos" (Filtra cancelados/concluídos e ordena por horário)
const upcomingAppointments = computed(() => {
  return appointments.value
    .filter(a => a.date === selectedDate.value && (a.status === 'confirmado' || a.status === 'pendente'))
    .sort((a, b) => a.startTime.localeCompare(b.startTime))
    .slice(0, 5); // Mostra apenas os 5 próximos para não poluir
});

// ─── Helpers Visuais ──────────────────────────────────────────────────────────

// Define a cor da "bolinha" baseada em quão perto está o agendamento
function getUrgencyColor(timeStr: string): string {
  // Num cenário real, compararíamos com o `new Date()` atual.
  // Como estamos com dados falsos, faremos uma simulação baseada na hora:
  const hour = parseInt(timeStr.split(":")[0]);
  if (hour < 12) return 'negative'; // Já passou ou está muito atrasado
  if (hour >= 12 && hour < 15) return 'warning'; // Próximo
  return 'positive'; // Tranquilo (mais tarde)
}
</script>

<style scoped>
/* Utilitários de Design Moderno */
.border-radius-md {
  border-radius: 12px;
}

.kpi-card {
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}
.kpi-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 16px rgba(0,0,0,0.05);
}

.hover-grey:hover {
  background-color: #fcfcfc;
}

.transition-generic {
  transition: all 0.2s ease;
}
</style>