<script setup>
import { ref, computed } from 'vue'


const initialStates = [
  { id: 's1', name: 'IDLE', x: 100, y: 150, isInitial: true },
  { id: 's2', name: 'FETCHING', x: 380, y: 150, isInitial: false },
  { id: 's3', name: 'RESOLVED', x: 660, y: 80, isInitial: false },
  { id: 's4', name: 'REJECTED', x: 660, y: 260, isInitial: false }
]

const initialTransitions = [
  { id: 't1', from: 's1', to: 's2', event: 'FETCH' },
  { id: 't2', from: 's2', to: 's3', event: 'RESOLVE' },
  { id: 't3', from: 's2', to: 's4', event: 'REJECT' },
  { id: 't4', from: 's4', to: 's2', event: 'RETRY' }
]


const states = ref(initialStates)
const transitions = ref(initialTransitions)
const currentStateId = ref('s1')


const newStateName = ref('')
const newEventName = ref('')
const selectedFromState = ref('s1')
const selectedToState = ref('s2')


const eventLogs = ref([
  ' DevState Engine initialized. Initial state set to IDLE.'
])


const activeState = computed(() => {
  return states.value.find(s => s.id === currentStateId.value)
})


const availableTriggers = computed(() => {
  return transitions.value.filter(t => t.from === currentStateId.value)
})


const handleAddState = () => {
  const cleanName = newStateName.value.trim().toUpperCase().replace(/\s+/g, '_')
  if (!cleanName) return

  const newState = {
    id: `s-${Date.now().toString().slice(-4)}`,
    name: cleanName,
    x: Math.floor(Math.random() * 400) + 100,
    y: Math.floor(Math.random() * 200) + 100,
    isInitial: false
  }

  states.value.push(newState)
  eventLogs.value.unshift(`📥 STATE PROVISIONED: Created state node [${cleanName}]`)
  newStateName.value = ''
}


const handleAddTransition = () => {
  const cleanEvent = newEventName.value.trim().toUpperCase().replace(/\s+/g, '_')
  if (!cleanEvent) return

  const newTrans = {
    id: `t-${Date.now().toString().slice(-4)}`,
    from: selectedFromState.value,
    to: selectedToState.value,
    event: cleanEvent
  }

  transitions.value.push(newTrans)
  
  const fromName = states.value.find(s => s.id === selectedFromState.value)?.name
  const toName = states.value.find(s => s.id === selectedToState.value)?.name
  
  eventLogs.value.unshift(` TRANSITION LINKED: [${fromName}] ➔ (${cleanEvent}) ➔ [${toName}]`)
  newEventName.value = ''
}


const triggerEvent = (transition) => {
  const targetState = states.value.find(s => s.id === transition.to)
  if (!targetState) return

  const oldStateName = activeState.value?.name
  currentStateId.value = targetState.id

  eventLogs.value.unshift(
    `⚡ EVENT DISPATCHED: "${transition.event}" | Transitioned from [${oldStateName}] ➔ [${targetState.name}]`
  )
}


const resetMachine = () => {
  const init = states.value.find(s => s.isInitial) || states.value[0]
  if (init) {
    currentStateId.value = init.id
    eventLogs.value.unshift(`🔄 FSM RESET: Reverted back to initial state [${init.name}]`)
  }
}


const getNodeCoordinates = (id) => {
  const node = states.value.find(s => s.id === id)
  if (!node) return { x: 0, y: 0 }
  return { x: node.x + 70, y: node.y + 35 }
}


const exportedJSON = computed(() => {
  const schema = {
    initial: states.value.find(s => s.isInitial)?.name || 'UNKNOWN',
    states: {}
  }

  states.value.forEach(s => {
    schema.states[s.name] = {
      on: {}
    }
  })

  transitions.value.forEach(t => {
    const fromName = states.value.find(s => s.id === t.from)?.name
    const toName = states.value.find(s => s.id === t.to)?.name
    if (fromName && toName) {
      schema.states[fromName].on[t.event] = toName
    }
  })

  return JSON.stringify(schema, null, 2)
})
</script>

<template>
  <div class="studio-container">
    
    <header class="hud-header">
      <div>
        <h1 class="brand-title"> DevState Automata Architect</h1>
        <p class="brand-subtitle">
          Vue 3 Composition API visual finite state machine compiler and transition interpreter.
        </p>
      </div>

      <div class="header-actions">
        <div class="active-badge">
          <span>Active State:</span>
          <strong>{{ activeState ? activeState.name : 'NONE' }}</strong>
        </div>
        <button class="btn btn-reset" @click="resetMachine"> Reset Machine</button>
      </div>
    </header>

    
    <div class="workspace-grid">
      
     
      <main class="canvas-card">
        <div class="card-header">
          <h3>Visual State Machine Canvas Graph</h3>
          <span class="hint">Interactive node coordinates</span>
        </div>

       
        <div class="canvas-viewport">
          
          <svg class="svg-layer">
            <defs>
              <marker id="arrow" viewBox="0 0 10 10" refX="28" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
                <path d="M 0 0 L 10 5 L 0 10 z" fill="#38bdf8" />
              </marker>
            </defs>

            <g v-for="t in transitions" :key="t.id">
              <line
                :x1="getNodeCoordinates(t.from).x"
                :y1="getNodeCoordinates(t.from).y"
                :x2="getNodeCoordinates(t.to).x"
                :y2="getNodeCoordinates(t.to).y"
                stroke="#38bdf8"
                stroke-width="2"
                marker-end="url(#arrow)"
              />
              <text
                :x="(getNodeCoordinates(t.from).x + getNodeCoordinates(t.to).x) / 2"
                :y="(getNodeCoordinates(t.from).y + getNodeCoordinates(t.to).y) / 2 - 8"
                fill="#a78bfa"
                font-size="10"
                font-family="monospace"
                text-anchor="middle"
              >
                {{ t.event }}
              </text>
            </g>
          </svg>

          
          <div
            v-for="s in states"
            :key="s.id"
            class="state-node"
            :class="{ 'is-active': currentStateId === s.id, 'is-initial': s.isInitial }"
            :style="{ left: `${s.x}px`, top: `${s.y}px` }"
            @click="currentStateId = s.id"
          >
            <div class="node-badge" v-if="s.isInitial">INIT</div>
            <div class="node-title">{{ s.name }}</div>
            <div class="node-id">{{ s.id }}</div>
          </div>
        </div>
      </main>

      
      <aside class="controls-column">
        
       
        <section class="control-card">
          <h3>Event Trigger Dispatcher</h3>
          <div v-if="availableTriggers.length > 0" class="trigger-buttons">
            <button
              v-for="t in availableTriggers"
              :key="t.id"
              class="btn btn-trigger"
              @click="triggerEvent(t)"
            >
              ⚡ {{ t.event }} ➔ {{ states.find(s => s.id === t.to)?.name }}
            </button>
          </div>
          <div v-else class="empty-state">
            🚫 No outgoing transitions available from current state.
          </div>
        </section>

        <!-- STATE PROVISIONING FORM -->
        <section class="control-card">
          <h3>Provision State Node</h3>
          <form @submit.prevent="handleAddState" class="form-row">
            <input
              v-model="newStateName"
              type="text"
              placeholder="e.g. PROCESSING"
              class="input-field"
            />
            <button type="submit" class="btn btn-primary">Add </button>
          </form>
        </section>

        
        <section class="control-card">
          <h3>Create Event Transition Link</h3>
          <form @submit.prevent="handleAddTransition" class="form-col">
            <input
              v-model="newEventName"
              type="text"
              placeholder="Event Name (e.g. TIMEOUT)"
              class="input-field"
            />
            <div class="form-row">
              <select v-model="selectedFromState" class="select-field">
                <option v-for="s in states" :key="s.id" :value="s.id">
                  From: {{ s.name }}
                </option>
              </select>
              <select v-model="selectedToState" class="select-field">
                <option v-for="s in states" :key="s.id" :value="s.id">
                  To: {{ s.name }}
                </option>
              </select>
            </div>
            <button type="submit" class="btn btn-accent">Link Transition </button>
          </form>
        </section>

      </aside>

    </div>


    <div class="bottom-grid">
      <section class="log-card">
        <h3>Runtime State Logs</h3>
        <div class="log-terminal">
          <div v-for="(log, idx) in eventLogs" :key="idx" class="log-line">
            {{ log }}
          </div>
        </div>
      </section>

      <section class="json-card">
        <h3>Compiled FSM JSON Schema</h3>
        <pre class="json-preview">{{ exportedJSON }}</pre>
      </section>
    </div>

  </div>
</template>

<style>
/* Clean dark studio styling sheet */
body {
  margin: 0;
  background-color: #070a13;
  color: #f8fafc;
  font-family: monospace;
}

.studio-container {
  max-width: 1300px;
  margin: 30px auto;
  padding: 0 24px;
}

.hud-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid #1e293b;
  padding-bottom: 20px;
  margin-bottom: 25px;
}

.brand-title {
  margin: 0;
  font-size: 24px;
  color: #38bdf8;
}

.brand-subtitle {
  margin: 4px 0 0 0;
  color: #475569;
  font-size: 12px;
}

.header-actions {
  display: flex;
  align-items: center;
  gap: 15px;
}

.active-badge {
  background-color: #0f172a;
  border: 1px solid #38bdf8;
  padding: 8px 14px;
  border-radius: 8px;
  font-size: 12px;
  color: #38bdf8;
}

.workspace-grid {
  display: grid;
  grid-template-columns: 1fr 360px;
  gap: 25px;
  margin-bottom: 25px;
}

.canvas-card, .control-card, .log-card, .json-card {
  background-color: #0f172a;
  border: 1px solid #1e293b;
  border-radius: 14px;
  padding: 20px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.card-header h3 {
  margin: 0;
  font-size: 13px;
  color: #64748b;
  text-transform: uppercase;
}

.hint {
  font-size: 10px;
  color: #475569;
}

.canvas-viewport {
  position: relative;
  min-height: 380px;
  background-color: #020617;
  border: 1px dashed #334155;
  border-radius: 12px;
  overflow: hidden;
  background-image: radial-gradient(#1e293b 1px, transparent 1px);
  background-size: 20px 20px;
}

.svg-layer {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

.state-node {
  position: absolute;
  width: 140px;
  padding: 12px;
  background-color: #0f172a;
  border: 1px solid #1e293b;
  border-radius: 10px;
  cursor: pointer;
  text-align: center;
  transition: all 0.2s;
  box-sizing: border-box;
}

.state-node.is-active {
  border-color: #38bdf8;
  box-shadow: 0 0 15px rgba(56, 189, 248, 0.3);
  background-color: #0369a1;
}

.state-node.is-initial {
  border-left: 4px solid #10b981;
}

.node-badge {
  font-size: 8px;
  color: #10b981;
  font-weight: bold;
}

.node-title {
  font-size: 13px;
  font-weight: bold;
  color: #fff;
}

.node-id {
  font-size: 9px;
  color: #94a3b8;
  margin-top: 4px;
}

.controls-column {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.control-card h3 {
  margin: 0 0 12px 0;
  font-size: 12px;
  color: #64748b;
  text-transform: uppercase;
}

.form-row {
  display: flex;
  gap: 10px;
}

.form-col {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.input-field, .select-field {
  flex: 1;
  padding: 8px 12px;
  background-color: #070a13;
  border: 1px solid #1e293b;
  border-radius: 6px;
  color: #fff;
  font-size: 12px;
  font-family: monospace;
}

.btn {
  padding: 8px 14px;
  border: none;
  border-radius: 6px;
  font-size: 12px;
  font-weight: bold;
  cursor: pointer;
}

.btn-primary { background-color: #334155; color: #fff; }
.btn-accent { background-color: #38bdf8; color: #070a13; }
.btn-reset { background-color: #1e293b; color: #cbd5e1; border: 1px solid #334155; }

.btn-trigger {
  width: 100%;
  text-align: left;
  background-color: #070a13;
  border: 1px solid #a78bfa;
  color: #a78bfa;
  margin-bottom: 8px;
}

.btn-trigger:hover {
  background-color: #a78bfa;
  color: #070a13;
}

.empty-state {
  font-size: 11px;
  color: #475569;
}

.bottom-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 25px;
}

.log-terminal {
  background-color: #070a13;
  border-radius: 8px;
  padding: 12px;
  height: 120px;
  overflow-y: auto;
}

.log-line {
  font-size: 11px;
  color: #38bdf8;
  margin-bottom: 4px;
}

.json-preview {
  margin: 0;
  background-color: #070a13;
  padding: 12px;
  border-radius: 8px;
  height: 120px;
  overflow-y: auto;
  font-size: 10px;
  color: #10b981;
}
</style>