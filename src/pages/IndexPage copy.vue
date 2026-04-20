<script setup lang="ts">
import { ref, type CSSProperties } from 'vue'
import { QCalendarDay, today } from '@quasar/quasar-ui-qcalendar'
import { useQuasar } from 'quasar' 
import '@quasar/quasar-ui-qcalendar/dist/index.css'

const $q = useQuasar() 

// --- Interfaces ---
interface CalendarEvent {
  id: string
  title: string
  date: string
  time: string
  duration: number
  bgcolor: string
  textColor?: string
  professional: string
  client: string
  phone: string
  imageUrl: string
  status: string
}

// --- Helpers ---
let eventGuid = 0
const createEventId = (): string => String(eventGuid++)
const currentDate = today()

// --- Estado Reativo ---
const selectedDate = ref(today())
const weekends = ref<boolean>(true)
const calendar = ref<any>(null)
const leftDrawerOpen = ref(false) // Começa fechado no mobile

// --- Dados Fictícios ---
const INITIAL_EVENTS = ref<CalendarEvent[]>([
  {
    id: createEventId(),
    title: 'Corte Cabelo + Barba',
    date: currentDate,
    time: '10:00',
    duration: 30,
    bgcolor: '#28a745',
    professional: 'Carlos Silva',
    client: 'João Mendes',
    phone: '(11) 98765-4321',
    imageUrl: 'https://i.pravatar.cc/150?u=carlos',
    status: 'Confirmado'
  },
  {
    id: createEventId(),
    title: 'Design de Sobrancelha',
    date: currentDate,
    time: '13:00',
    duration: 30,
    bgcolor: '#ffc107',
    textColor: 'black',
    professional: 'Ana Paula',
    client: 'Mariana Souza',
    phone: '(11) 91234-5678',
    imageUrl: 'https://i.pravatar.cc/150?u=ana',
    status: 'Aguardando Pagamento'
  },
  {
    id: createEventId(),
    title: 'Limpeza de Pele',
    date: currentDate,
    time: '15:30',
    duration: 90,
    bgcolor: '#17a2b8',
    professional: 'Dra. Camila',
    client: 'Roberto Alves',
    phone: '(11) 99999-8888',
    imageUrl: 'https://i.pravatar.cc/150?u=camila',
    status: 'Em Atendimento'
  }
])

// --- Funções do Calendário ---
const getEventsForDate = (date: string): CalendarEvent[] => {
  return INITIAL_EVENTS.value.filter(event => event.date === date)
}

const getEventStyle = (event: CalendarEvent, timeStartPos: any, timeDurationHeight: any): CSSProperties => {
  return {
    top: timeStartPos(event.time) + 'px',
    // Mantemos a inteligência de crescer o card se necessário, 
    // mas com menos padding no mobile
    minHeight: timeDurationHeight(event.duration) + 'px', 
    height: 'fit-content', 
    paddingBottom: $q.screen.lt.md ? '4px' : '24px', 
    backgroundColor: event.bgcolor,
    color: event.textColor || 'white',
    left: '1px',
    right: '1px',
    position: 'absolute',
    borderRadius: '4px',
    zIndex: 10,
    // Impede vazar texto na coluna fina do mobile
    overflow: $q.screen.lt.md ? 'hidden' : 'visible'
  }
}

// --- Ações ---
const onTimeClick = (timeInfo: { date: string; time: string }): void => {
  const title = prompt(`Novo agendamento às ${timeInfo.time}: Digite o serviço`)
  if (title) {
    INITIAL_EVENTS.value.push({
      id: createEventId(),
      title,
      date: timeInfo.date,
      time: timeInfo.time,
      duration: 60,
      bgcolor: '#6c757d',
      professional: 'A Definir',
      client: 'Novo Cliente',
      phone: 'Sem registro',
      imageUrl: '',
      status: 'Pendente'
    })
  }
}

const onEventClick = (event: CalendarEvent): void => {
  if (confirm(`Deseja cancelar o agendamento de '${event.client}' para '${event.title}'?`)) {
    INITIAL_EVENTS.value = INITIAL_EVENTS.value.filter(e => e.id !== event.id)
  }
}
</script>

<template>
  <q-layout view="hHh Lpr lff" class="bg-grey-2">
    
    <q-drawer v-model="leftDrawerOpen" show-if-above bordered :width="320" class="bg-white">
      <div class="q-pa-md">
        <div class="text-h6 q-mb-md">Painel de Controle</div>
        
        <q-toggle v-model="weekends" label="Mostrar Finais de Semana" class="q-mb-md" color="primary" />

        <div class="text-subtitle2 text-weight-bold q-mb-sm text-grey-8">
          Agendamentos ({{ INITIAL_EVENTS.length }})
        </div>

        <q-list padding dense>
          <q-item v-for="event in INITIAL_EVENTS" :key="event.id" class="bg-grey-1 q-mb-sm rounded-borders">
            <q-item-section>
              <q-item-label class="text-weight-bold">{{ event.time }} - {{ event.title }}</q-item-label>
              <q-item-label caption>🧑 {{ event.client }}</q-item-label>
            </q-item-section>
          </q-item>
        </q-list>
      </div>
    </q-drawer>

    <q-page-container>
      <q-page class="q-pa-xs q-pa-md-md"> 
        <q-card flat bordered class="rounded-borders overflow-hidden">
          
          <q-toolbar class="bg-grey-1 text-black border-bottom">
            <q-btn flat dense round icon="menu" @click="leftDrawerOpen = !leftDrawerOpen" class="q-mr-sm lt-md" />
            <q-btn flat dense round icon="chevron_left" @click="calendar?.prev()" />
            <q-toolbar-title class="text-subtitle1 text-center text-weight-bold">
              {{ selectedDate }}
            </q-toolbar-title>
            <q-btn flat dense round icon="chevron_right" @click="calendar?.next()" />
          </q-toolbar>

          <q-calendar-day 
            ref="calendar" 
            v-model="selectedDate" 
            view="week" 
            :no-active-date="true" 
            :interval-start="8"
            :interval-count="12" 
            :interval-minutes="60" 
            :interval-height="120" 
            :short-weekday-label="true" 
            :hide-header="!weekends"
            time-clicks-clamped 
            animated
            @timeclick="onTimeClick" 
            style="height: calc(100vh - 120px);"
          >
            <template #day-body="{ scope: { timestamp, timeStartPos, timeDurationHeight } }">
              <template v-for="event in getEventsForDate(timestamp.date)" :key="event.id">

                <div 
                  class="shadow-2 cursor-pointer transition-generic column no-wrap" 
                  :class="$q.screen.lt.md ? 'q-pa-none q-px-xs' : 'q-pa-xs'"
                  :style="getEventStyle(event, timeStartPos, timeDurationHeight)"
                  @click.stop="onEventClick(event)"
                >
                  
                  <div v-if="$q.screen.lt.md" class="micro-text">
                     {{ event.title }}
                  </div>

                  <template v-else>
                    <div class="row items-center no-wrap q-mb-xs">
                      <q-avatar size="24px" class="q-mr-xs" v-if="event.imageUrl">
                        <img :src="event.imageUrl" />
                      </q-avatar>
                      <div class="text-caption text-weight-bold ellipsis">
                        {{ event.time }} - {{ event.title }}
                      </div>
                    </div>

                    <div class="text-caption ellipsis" style="font-size: 0.7rem; line-height: 1;">
                      <q-icon name="person" size="xs"/> {{ event.client }}
                    </div>
                    <div class="text-caption ellipsis" style="font-size: 0.7rem; line-height: 1;">
                      <q-icon name="phone" size="xs"/> {{ event.phone }}
                    </div>
                    
                    <div class="absolute-bottom-right q-pa-xs">
                       <q-badge transparent color="black" class="text-white" :label="event.status" style="opacity: 0.6;" />
                    </div>
                  </template>

                </div>

              </template>
            </template>
          </q-calendar-day>
        </q-card>
      </q-page>
    </q-page-container>
  </q-layout>
</template>

<style scoped>
.border-bottom {
  border-bottom: 1px solid #e0e0e0;
}

.transition-generic {
  transition: transform 0.2s ease, filter 0.2s;
}

.transition-generic:hover {
  transform: scale(1.01);
  filter: brightness(1.1);
  z-index: 20 !important;
}

/* CSS especial para o texto no celular não quebrar o layout */
.micro-text {
  font-size: 0.6rem;
  line-height: 1.1;
  font-weight: bold;
  overflow-wrap: break-word; /* Quebra palavras grandes para caber na coluna */
  word-break: break-word;
  display: -webkit-box;
  -webkit-line-clamp: 3; /* Limita a 3 linhas para não estourar o card pequeno */
  -webkit-box-orient: vertical;
  overflow: hidden;
  padding-top: 2px;
}
</style>