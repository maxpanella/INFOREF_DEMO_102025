<template>
  <div class="demo-root">
    <aside class="sidebar">
      <div class="brand">Inforef RTLS Demo</div>

      <!-- Link esterno subito sotto il brand -->
      <div class="external-link-wrapper">
        <a
          class="top-link-btn"
          href="https://demo.sgslweb.com/backofficeAzienda.jsp"
          target="_blank"
          rel="noopener"
          title="Apri INOFREF DEMO"
        >
          SICURWEB DEMO
        </a>
      </div>

      <div class="controls">
        <label>Struttura</label>
        <select v-model="selectedSiteId">
          <option v-for="s in sites" :key="s.id" :value="s.id">{{ s.ragioneSociale }}</option>
        </select>

        <div class="btn-row">
          <button @click="showOverview" :disabled="overviewMode">Vista generale</button>
          <button @click="showSiteView" :disabled="!overviewMode">Vista sito</button>
        </div>
        <div class="btn-row">
          <button v-if="overviewMode" @click="fitAllSites">Inquadra tutte</button>
          <button v-else @click="fitAll">Centra mappa</button>
        </div>

        <div class="btn-row">
          <button @click="startDemo" :disabled="demoRunning || overviewMode">Start demo</button>
          <button @click="stopDemo" :disabled="!demoRunning">Stop</button>
        </div>
        <div class="btn-row">
          <button @click="simulateOnce" :disabled="overviewMode">Simula sample</button>
        </div>
      </div>

      <maintenance-list />

      <div class="events">
        <h4>Eventi</h4>
        <ul>
          <li v-for="(e,i) in events" :key="i"><small>{{ e.t }}</small> — {{ e.msg }}</li>
        </ul>
      </div>
    </aside>

    <main class="main">
      <div class="map-wrapper" ref="mapWrapEl">
        <div id="map"></div>
        
        <!-- Controlli mappa avanzati -->
        <div v-if="!overviewMode" class="map-controls">
          <div class="control-group">
            <button @click="toggleClustering" :class="{ active: useClustering }" class="control-btn">
              {{ useClustering ? '🔗' : '🔘' }} Cluster
            </button>
            <button @click="filterByCategory('all')" :class="{ active: currentFilter === 'all' }" class="control-btn">
              🌐 Tutto
            </button>
          </div>
          
          <div class="control-group">
            <button @click="filterByCategory('personale')" :class="{ active: currentFilter === 'personale' }" class="control-btn category-personale">
              👨‍⚕️ Personale
            </button>
            <button @click="filterByCategory('pazienti')" :class="{ active: currentFilter === 'pazienti' }" class="control-btn category-pazienti">
              🛌 Pazienti
            </button>
            <button @click="filterByCategory('dispositivi')" :class="{ active: currentFilter === 'dispositivi' }" class="control-btn category-dispositivi">
              💊 Dispositivi
            </button>
          </div>
        </div>

        <!-- Legenda mappa -->
        <div v-if="!overviewMode" class="map-legend">
          <div class="legend-title">Legenda</div>
          <div class="legend-items">
            <div class="legend-item">
              <span class="legend-color personale"></span>
              <span>Personale</span>
            </div>
            <div class="legend-item">
              <span class="legend-color pazienti"></span>
              <span>Pazienti</span>
            </div>
            <div class="legend-item">
              <span class="legend-color dispositivi"></span>
              <span>Dispositivi</span>
            </div>
            <div class="legend-item">
              <span class="legend-color sicurezza"></span>
              <span>Sicurezza</span>
            </div>
          </div>
        </div>
      </div>

      <!-- OVERVIEW: dashboard KPI -->
      <div v-if="overviewMode" class="overview">
        <h3>Strutture - Dashboard KPI</h3>
        <div class="sites-list">
          <div class="site-card" v-for="card in overviewCards" :key="card.id">
            <div class="site-head">
              <div class="site-name">{{ card.name }}</div>
              <button class="link" @click="selectSite(card.id)">Vai al sito</button>
            </div>

            <!-- KPI: PATRIMONI -->
            <div class="kpi-section">
              <div class="kpi-header">🏛️ PATRIMONI</div>
              <div class="kpi-grid">
                <div class="kpi-item">
                  <div class="kpi-label">Risorse Totali</div>
                  <div class="kpi-value">{{ card.totals.risorse }}</div>
                  <div class="kpi-detail">
                    <span class="badge blue">Sanitari: {{ card.counts.sanitari }}</span>
                    <span class="badge green">Dispositivi: {{ card.counts.dispositivi }}</span>
                  </div>
                </div>
                <div class="kpi-item">
                  <div class="kpi-label">Utilizzo Risorse</div>
                  <div class="kpi-value" :class="getUtilizationClass(card.utilization)">
                    {{ Math.round(card.utilization * 100) }}%
                  </div>
                  <div class="kpi-detail">
                    <small>Efficienza operativa</small>
                  </div>
                </div>
              </div>
            </div>

            <!-- KPI: PRESTAZIONI SANITARIE -->
            <div class="kpi-section">
              <div class="kpi-header">🏥 PRESTAZIONI SANITARIE</div>
              <div class="kpi-grid">
                <div class="kpi-item">
                  <div class="kpi-label">Pazienti Attivi</div>
                  <div class="kpi-value">{{ card.counts.pazienti }}</div>
                  <div class="kpi-detail">
                    <span class="badge green">In degenza: {{ card.prestazioni.degenza }}</span>
                  </div>
                </div>
                <div class="kpi-item">
                  <div class="kpi-label">Visitatori Oggi</div>
                  <div class="kpi-value">{{ card.counts.visitatori }}</div>
                  <div class="kpi-detail">
                    <span class="badge amber">Prestaz. erogate: {{ card.prestazioni.erogate }}</span>
                  </div>
                </div>
              </div>
            </div>

            <!-- KPI: SICUREZZA -->
            <div class="kpi-section">
              <div class="kpi-header">🛡️ SICUREZZA</div>
              <div class="kpi-grid">
                <div class="kpi-item">
                  <div class="kpi-label">Stato Sicurezza</div>
                  <div class="kpi-value" :class="card.sicurezza.statusClass">
                    {{ card.sicurezza.statusText }}
                  </div>
                  <div class="kpi-detail">
                    <span class="badge" :class="card.sicurezza.antincendioClass">
                      Antincendio: {{ card.sicurezza.antincendio }}
                    </span>
                  </div>
                </div>
                <div class="kpi-item">
                  <div class="kpi-label">Manutenzioni</div>
                  <div class="kpi-value" :class="card.sicurezza.manutenzioneClass">
                    {{ card.sicurezza.manutenzioniInCorso }}
                  </div>
                  <div class="kpi-detail">
                    <small>Dispositivi da verificare</small>
                  </div>
                </div>
              </div>
            </div>

            <!-- INDICATORE SINTETICO RISCHIO -->
            <div class="risk-indicator">
              <div class="risk-label">Indice di Rischio Complessivo</div>
              <div class="risk-score" :class="card.risk.levelClass">
                {{ card.risk.score }}/100
                <span class="risk-level">{{ card.risk.levelText }}</span>
              </div>
            </div>

            <div v-if="card.id==='sorrisi'" class="extra-actions">
              <small>Demo piantina reparto (SORRISI):</small>
              <button class="link" @click="openFloorPlan('sorrisi','Degenza',1)">Apri piantina Degenza</button>
            </div>
          </div>
        </div>
      </div>

      <!-- VISTA SITO: SOLO mappa + tabella risorse -->
      <div v-else class="site-dashboard">
        <!-- Header sito -->
        <div class="site-header sticky">
          <div class="site-info">
            <h2>{{ selectedSiteLabel }}</h2>
            <div class="site-stats">
              <div class="stat">
                <div class="stat-value">{{ filteredAssets.length }}</div>
                <div class="stat-label">Risorse Totali</div>
              </div>
              <div class="stat">
                <div class="stat-value">{{ activePatientsCount }}</div>
                <div class="stat-label">Pazienti Attivi</div>
              </div>
              <div class="stat">
                <div class="stat-value">{{ activeStaffCount }}</div>
                <div class="stat-label">Personale in Servizio</div>
              </div>
              <div class="stat">
                <div class="stat-value" :class="getStatusClass(operationalStatus)">
                  {{ operationalStatus }}
                </div>
                <div class="stat-label">Stato Operativo</div>
              </div>
            </div>
          </div>
          <div class="site-actions">
            <button class="btn-primary" @click="openMainFloorPlan">Apri Mappa Principale</button>
            <button class="btn-secondary" @click="showOverview">Torna alla Panoramica</button>
          </div>
        </div>

        <!-- Tabella risorse per reparto -->
        <div class="dept-panel">
          <div class="dept-header sticky">
            <h3>Risorse per Reparto — {{ selectedSiteLabel }}</h3>
            <div class="spacer"></div>

            <!-- Filtro testuale live -->
            <div class="filters">
              <input
                class="filter-input"
                type="text"
                v-model="tableSearch"
                placeholder="Filtra nome, categoria o stanza…"
                aria-label="Filtro risorse"
              />
              <button class="link clear" v-if="tableSearch" @click="tableSearch=''">Pulisci</button>
            </div>

            <!-- Filtri rapidi per tipologia -->
            <div class="chips">
              <button :class="['chip', {active: tableCategory==='all'}]" @click="tableCategory='all'">Tutto</button>
              <button :class="['chip', {active: tableCategory==='pazienti'}]" @click="tableCategory='pazienti'">Pazienti</button>
              <button :class="['chip', {active: tableCategory==='personale'}]" @click="tableCategory='personale'">Personale</button>
              <button :class="['chip', {active: tableCategory==='dispositivi'}]" @click="tableCategory='dispositivi'">Dispositivi</button>
            </div>

            <!-- Tasti Formazione e Visite -->
            <div class="quick-actions">
              <select v-model="selectedPersonForQual" class="qa-select" aria-label="Seleziona personale">
                <option :value="null">Seleziona personale…</option>
                <option v-for="p in staffList" :key="p.id" :value="p.id">{{ p.nome }}</option>
              </select>
              <button class="qa-btn" :disabled="!selectedPersonForQual" @click="openQualifyById(selectedPersonForQual)">Apri Formazione</button>

              <select v-model="selectedPatientForVisits" class="qa-select" aria-label="Seleziona paziente">
                <option :value="null">Seleziona paziente…</option>
                <option v-for="p in patientList" :key="p.id" :value="p.id">{{ p.nome }}</option>
              </select>
              <button class="qa-btn" :disabled="!selectedPatientForVisits" @click="openVisitsById(selectedPatientForVisits)">Visite mediche</button>
            </div>

            <button class="link" @click="expandAll">Espandi tutto</button>
            <button class="link" @click="collapseAll">Comprimi tutto</button>
          </div>

          <div class="accordion">
            <div class="section" v-for="sec in groupedByDept" :key="sec.key">
              <div class="section-head" @click="toggleSection(sec.key)">
                <div class="title">
                  <span class="chev" :class="{open: isOpen(sec.key)}">▸</span>
                  <strong>{{ sec.key }}</strong>
                </div>
                <div class="badge">{{ sec.items.length }}</div>
                <div class="actions" @click.stop>
                  <button class="link" @click="openFloorPlan(selectedSite?.id || selectedSiteId, sec.key, 1)">
                    Mappa Live
                  </button>
                </div>
              </div>

              <div class="section-body" v-show="isOpen(sec.key)">
                <table class="resources-table">
                  <thead>
                    <tr>
                      <th class="col-name">Nome</th>
                      <th class="col-cat">Categoria</th>
                      <th class="col-room">Stanza</th>
                      <th class="col-state">Stato</th>
                      <th class="th-actions">Azioni</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="a in sec.items" :key="a.id">
                      <td>
                        <div class="cell-name">
                          <strong>{{ a.nome }}</strong>
                          <small v-if="isPatient(a) && a.admissionDate" class="muted">— degenza: {{ daysOfStay(a) }} gg</small>
                        </div>
                      </td>
                      <td>{{ a.categoria || a.tipo }}</td>
                      <td>{{ a.stanza || '-' }}</td>
                      <td>
                        <span :class="['pill', getStatusClass(a.stato)]">
                          {{ a.stato || 'Operativo' }}
                        </span>
                      </td>
                      <td class="td-actions">
                        <button @click="openCamera(a.cameraUrl)" :disabled="!a.cameraUrl" class="btn-ghost">Camera</button>
                        <!-- Etichetta "Formazione" -->
                        <button v-if="isHealthcare(a)" @click="openQualify(a)" class="btn-ghost">Formazione</button>
                        <!-- Manutenzione include anche ESTINTORI -->
                        <button v-if="isMaintainable(a)" @click="openMaintenanceFor(a)" class="btn-ghost">Manutenzione</button>
                        <button v-if="isPatient(a) && a.ehrUrl" @click="openPatientRecord(a)" class="btn-ghost">Fascicolo</button>
                        <button v-if="isPatient(a) || isHealthcare(a)" @click="openMedicalVisits(a)" class="btn-ghost">Visite mediche</button>
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>

            <div v-if="groupedByDept.length===0" class="empty">Nessun asset per questa struttura</div>
          </div>
        </div>
      </div>

      <camera-modal v-if="cameraModal.show" :url="cameraModal.url" @close="cameraModal.show=false" />
      <qualification-panel
        v-if="qualPanel.show"
        :person="qualPanel.person"
        :skills="qualPanel.skills"
        @close="qualPanel.show=false"
      />
      <maintenance-panel
        v-if="maintPanel.show"
        :asset="maintPanel.asset"
        @close="maintPanel.show=false"
      />
      <medical-visits-modal
        v-if="visitsModal.show"
        :asset="visitsModal.asset"
        @close="visitsModal.show=false"
      />
      <floor-plan
        v-if="floorPlan.show"
        :site-id="floorPlan.siteId"
        :dept="floorPlan.dept"
        :floor="floorPlan.floor"
        :assets="floorPlan.assets"
        @ward-entry="handleWardEntry"
        @alert="handleAlert"
        @close="floorPlan.show=false"
      />
    </main>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, onUnmounted, nextTick } from 'vue'
import OlMap from 'ol/Map.js'
import View from 'ol/View.js'
import TileLayer from 'ol/layer/Tile.js'
import OSM from 'ol/source/OSM.js'
import VectorLayer from 'ol/layer/Vector.js'
import VectorSource from 'ol/source/Vector.js'
import ClusterSource from 'ol/source/Cluster.js'
import Feature from 'ol/Feature.js'
import { Point, LineString } from 'ol/geom.js'
import Style from 'ol/style/Style.js'
import CircleStyle from 'ol/style/Circle.js'
import Stroke from 'ol/style/Stroke.js'
import Fill from 'ol/style/Fill.js'
import Text from 'ol/style/Text.js'
import { fromLonLat, toLonLat } from 'ol/proj.js'

import CameraModal from './CameraModal.vue'
import MaintenanceList from './MaintenanceList.vue'
import QualificationPanel from './QualificationPanel.vue'
import MaintenancePanel from './MaintenancePanel.vue'
import MedicalVisitsModal from './MedicalVisitsModal.vue'
import FloorPlan from './FloorPlan.vue'
import createLocalsenseAdapter from '../services/localsenseAdapter.js'

import { sites } from '../data/sites.js'
import { assetList } from '../data/assets.js'
import { skillsByPerson } from '../data/skills.js'

const demoRunning = ref(false)
const events = ref([])

// Mappa e layer
let map = null

// OVERVIEW: layer assets e layer siti
let assetSrc = null, assetLayer = null
let siteSrc = null, siteLayer = null

// SITE VIEW: sorgente singoli + cluster
let siteAssetSource = null
let singleSiteLayer = null
let clusterSource = null, clusterLayer = null

// Map nativo per tenere traccia delle feature in overview
const featureMap = new Map()

const mapWrapEl = ref(null)
let ro = null

// Stato vista
const overviewMode = ref(true)
// Preseleziona 'sorrisi' se presente
const selectedSiteId = ref((sites.find(s => s.id === 'sorrisi')?.id) || sites[0]?.id || null)

// Controlli mappa avanzati
const useClustering = ref(true)
const currentFilter = ref('all')

// Centro/rotazione iniziali: preferisci 'sorrisi' se presente
const defaultSiteForCenter = sites.find(s => s.id === 'sorrisi') || sites[0]
let defaultCenter = defaultSiteForCenter?.lon && defaultSiteForCenter?.lat
  ? fromLonLat([Number(defaultSiteForCenter.lon), Number(defaultSiteForCenter.lat)])
  : fromLonLat([14.8484, 40.6369])
let rotationDeg = Number(defaultSiteForCenter?.rotationDeg || 0)

// Coords puliti
const sitesWithCoords = computed(() =>
  sites.map(s => ({ ...s, lon: Number(s.lon), lat: Number(s.lat) }))
       .filter(s => Number.isFinite(s.lon) && Number.isFinite(s.lat))
)

const selectedSite = computed(() => sites.find(s => s.id === selectedSiteId.value) || null)
const selectedSiteLabel = computed(() => selectedSite.value ? selectedSite.value.ragioneSociale : '—')

// Base: filtra per sito (veicoli sempre visibili anche in vista sito)
const filteredAssets = computed(() =>
  assetList.value.filter(a => {
    if (!selectedSiteId.value) return true
    if ((a.categoria || '').toLowerCase() === 'veicolo') return true
    return a.siteId === selectedSiteId.value
  })
)

// Filtro testuale e per tipologia (tabella)
const tableSearch = ref('')
const tableCategory = ref('all') // all | pazienti | personale | dispositivi

const tableAssets = computed(() => {
  const q = (tableSearch.value || '').trim().toLowerCase()
  return filteredAssets.value.filter(a => {
    // filtro categoria rapida
    if (tableCategory.value !== 'all') {
      const cat = (a.categoria || '').toLowerCase()
      if (tableCategory.value === 'pazienti' && !cat.includes('paziente')) return false
      if (tableCategory.value === 'personale' && !cat.includes('personale')) return false
      if (tableCategory.value === 'dispositivi' && !(cat.includes('dispositivo') || cat.includes('infrastruttura') || cat.includes('sicurezza') || cat.includes('sensor'))) return false
    }
    // filtro testuale
    if (!q) return true
    const nome = (a.nome || '').toLowerCase()
    const catFull = ((a.categoria || a.tipo || '')).toLowerCase()
    const stanza = (a.stanza || '').toLowerCase()
    return nome.includes(q) || catFull.includes(q) || stanza.includes(q)
  })
})


// Apre il modal "Visite mediche" per l'asset selezionato (paziente o personale)
function openMedicalVisits(asset) {
  visitsModal.value = { show: true, asset }
}

// Dalla tendina rapida: cerca prima tra pazienti poi tra personale e apre il modal
function openVisitsById(patientId) {
  const asset =
    patientList.value.find(p => p.id === patientId) ||
    staffList.value.find(p => p.id === patientId)
  if (asset) openMedicalVisits(asset)
}

// Liste per "Formazione" e "Visite mediche"
const staffList = computed(() => tableAssets.value.filter(isHealthcare))
const patientList = computed(() => tableAssets.value.filter(isPatient))
const selectedPersonForQual = ref(null)
const selectedPatientForVisits = ref(null)

// Calcoli KPI (non influenzati dal filtro testuale)
const activePatientsCount = computed(() => 
  filteredAssets.value.filter(a => isPatient(a) && (!a.stato || !String(a.stato).toLowerCase().includes('dimesso'))).length
)
const activeStaffCount = computed(() =>
  filteredAssets.value.filter(a => isHealthcare(a) && (!a.stato || String(a.stato).toLowerCase().includes('servizio'))).length
)
const operationalStatus = computed(() => {
  const issues = filteredAssets.value.filter(a => 
    a.stato && ['guasto', 'manutenzione', 'critico'].some(s => String(a.stato).toLowerCase().includes(s))
  ).length
  return issues === 0 ? 'OTTIMO' : issues <= 2 ? 'BUONO' : 'ATTENZIONE'
})

// Helpers reparto
function getDeptKey(a){
  if (a.reparto) return a.reparto
  const tipo = (a.tipo||'').toLowerCase()
  const cat  = (a.categoria||'').toLowerCase()
  const stanza = (a.stanza||'').toLowerCase()

  if (tipo.includes('letto')) return 'Degenza'

  if (cat === 'personale') {
    if (stanza.includes('reception')) return 'Reception'
    return 'Infermeria'
  }
  if (cat === 'dispositivo-medico') {
    if (stanza.includes('ambulatorio') || tipo.includes('ecg')) return 'Infermeria'
    if (stanza.includes('osservazione') || stanza.includes('camera') || tipo.includes('monitor')) return 'Degenza'
    return 'Infermeria'
  }
  if (cat === 'sensor' || tipo.includes('sensore')) {
    if (tipo.includes('letto') || stanza.includes('camera')) return 'Degenza'
    if (tipo.includes('porta') || tipo.includes('uscita')) return 'Esterni'
    if (tipo.includes('caduta') || stanza.includes('corridoio')) return 'Corridoi'
    if (stanza.includes('bagno')) return 'Bagni assistiti'
    return 'Sicurezza'
  }
  if (cat === 'ausilio' || tipo.includes('carrello') || tipo.includes('wheelchair')) {
    if (stanza.includes('magazzino')) return 'Magazzino ausili'
    if (stanza.includes('infermeria')) return 'Infermeria'
    return 'Magazzino ausili'
  }
  if (cat === 'sicurezza' || tipo.includes('estintore') || tipo.includes('allarme') || tipo.includes('camera')) {
    if (tipo.includes('camera')) return 'Corridoi'
    return 'Sicurezza'
  }
  if (cat === 'infrastruttura' || tipo.includes('porta') || tipo.includes('ascensore') || tipo.includes('generatore') || tipo.includes('ups') || cat==='arredo') {
    if (tipo.includes('generatore') || tipo.includes('ups') || stanza.includes('tecnico')) return 'Locale tecnico'
    if (tipo.includes('porta') || tipo.includes('ascensore') || stanza.includes('corridoio')) return 'Corridoi'
    if (tipo.includes('letto')) return 'Degenza'
    return 'Infrastruttura'
  }
  if (cat === 'consumabili' || tipo.includes('kit')) {
    if (stanza.includes('reception')) return 'Reception'
    return 'Infermeria'
  }
  if (cat === 'paziente' || tipo.includes('paziente')) return 'Degenza'
  if (cat === 'visitatori' || tipo.includes('visitatore')) return 'Reception'
  if (stanza.includes('fisioterapia') || stanza.includes('riabilitazione')) return 'Riabilitazione'
  if (a.piano != null) return `Piano ${a.piano}`
  return 'Altro'
}

const deptPriority = [
  'Infermeria','Degenza','Riabilitazione','Fisioterapia','Corridoi','Bagni assistiti',
  'Magazzino ausili','Reception','Locale tecnico','Esterni','Sicurezza','Infrastruttura'
]

// Gruppo per reparto basato sui risultati del filtro
const groupedByDept = computed(() => {
  const buckets = Object.create(null)
  for (const a of tableAssets.value) {
    const key = getDeptKey(a)
    ;(buckets[key] ||= []).push(a)
  }
  const arr = Object.keys(buckets).map(k => ({ key: k, items: buckets[k] }))
  arr.sort((x, y) => {
    const ix = deptPriority.indexOf(x.key); const iy = deptPriority.indexOf(y.key)
    if (ix !== -1 && iy !== -1) return ix - iy
    if (ix !== -1) return -1
    if (iy !== -1) return 1
    if (x.key === 'Altro') return 1
    if (y.key === 'Altro') return -1
    return x.key.localeCompare(y.key, 'it')
  })
  return arr
})

const openSections = ref(new Set())
function isOpen(key){ return openSections.value.has(key) }
function toggleSection(key){
  const s = new Set(openSections.value)
  if (s.has(key)) s.delete(key); else s.add(key)
  openSections.value = s
}
function expandAll(){ openSections.value = new Set(groupedByDept.value.map(s => s.key)) }
function collapseAll(){ openSections.value = new Set() }

// Trasformazione coordinate (DXF -> map space)
function transformDxf(x, y) {
  const x_m = (Number(x) || 0) / 1000, y_m = (Number(y) || 0) / 1000
  const theta = rotationDeg * Math.PI / 180
  const x_rot = x_m * Math.cos(theta) - y_m * Math.sin(theta)
  const y_rot = x_m * Math.sin(theta) + y_m * Math.cos(theta)
  return [ defaultCenter[0] + x_rot, defaultCenter[1] - y_rot ]
}

function addEvent(msg) {
  events.value.unshift({ t: new Date().toLocaleTimeString(), msg })
  if (events.value.length > 300) events.value.pop()
}

// Stile avanzato con icone e colori per categoria
function makeAdvancedStyle(feature) {
  const props = feature.getProperties()
  const categoria = (props.categoria || '').toLowerCase()
  const stato = (props.stato || '').toLowerCase()
  
  let color, icon
  switch (true) {
    case categoria.includes('personale'):
      color = '#2563eb'
      icon = '👨‍⚕️'
      break
    case categoria.includes('paziente'):
      color = '#0891b2'
      icon = '🛌'
      break
    case categoria.includes('dispositivo'):
      color = '#16a34a'
      icon = '💊'
      break
    case categoria.includes('sicurezza'):
      color = '#dc2626'
      icon = '🛡️'
      break
    case categoria.includes('infrastruttura'):
      color = '#6b7280'
      icon = '🏗️'
      break
    case categoria.includes('veicolo'):
      color = '#ef4444'
      icon = '🚑'
      break
    default:
      color = '#8b5cf6'
      icon = '📦'
  }

  let strokeColor = '#ffffff'
  if (stato.includes('guasto') || stato.includes('critico')) {
    strokeColor = '#ef4444'
  } else if (stato.includes('manutenzione')) {
    strokeColor = '#f59e0b'
  }

  return new Style({
    image: new CircleStyle({
      radius: 8,
      fill: new Fill({ color }),
      stroke: new Stroke({
        color: strokeColor,
        width: 3
      })
    }),
    text: new Text({
      text: icon,
      offsetY: 2,
      fill: new Fill({ color: '#ffffff' }),
      font: '10px Arial',
      scale: 1.2
    })
  })
}

function makeSiteStyle(s) {
  const halo = new Style({
    image: new CircleStyle({
      radius: 16,
      fill: new Fill({ color: 'rgba(14,165,233,0.18)' }),
      stroke: new Stroke({ color: '#06b6d4', width: 3 })
    })
  })
  halo.setZIndex(20)

  const core = new Style({
    image: new CircleStyle({
      radius: 9,
      fill: new Fill({ color: '#0284c7' }),
      stroke: new Stroke({ color: '#ffffff', width: 2 })
    }),
    text: new Text({
      text: s.ragioneSociale,
      offsetY: -22,
      fill: new Fill({ color: '#0c4a6e' }),
      font: '700 13px Inter, Arial'
    })
  })
  core.setZIndex(21)
  return [halo, core]
}

// Mappa init avanzata
function initMap() {
  nextTick(() => {
    const mapElement = document.getElementById('map')
    if (!mapElement) {
      console.error('Elemento mappa non trovato')
      return
    }

    mapElement.style.minHeight = '400px'
    
    setTimeout(() => {
      try {
        map = new OlMap({
          target: 'map',
          layers: [ new TileLayer({ source: new OSM() }) ],
          view: new View({ 
            center: defaultCenter, 
            zoom: 16 
          })
        })

        // Layer per i siti (solo in overview)
        siteSrc = new VectorSource({ features: [] })
        siteLayer = new VectorLayer({ source: siteSrc })
        map.addLayer(siteLayer)

        // OVERVIEW: layer asset
        assetSrc = new VectorSource({ features: [] })
        assetLayer = new VectorLayer({ source: assetSrc })
        map.addLayer(assetLayer)

        // SITE VIEW: sorgente singoli e cluster
        siteAssetSource = new VectorSource({ features: [] })

        // layer singoli (quando cluster off)
        singleSiteLayer = new VectorLayer({
          source: siteAssetSource,
          style: (f) => makeAdvancedStyle(f),
          visible: !useClustering.value
        })
        map.addLayer(singleSiteLayer)

        // cluster source + layer
        clusterSource = new ClusterSource({
          distance: useClustering.value ? 40 : 0,
          minDistance: 10,
          source: siteAssetSource
        })

        clusterLayer = new VectorLayer({
          source: clusterSource,
          visible: useClustering.value,
          style: function(feature) {
            const features = feature.get('features')
            const size = Array.isArray(features) ? features.length : 0
            if (size > 1) {
              // stile cluster
              return new Style({
                image: new CircleStyle({
                  radius: 15 + Math.min(size, 10),
                  fill: new Fill({
                    color: 'rgba(59, 130, 246, 0.8)'
                  }),
                  stroke: new Stroke({
                    color: 'rgba(255, 255, 255, 0.8)',
                    width: 2
                  })
                }),
                text: new Text({
                  text: size.toString(),
                  fill: new Fill({ color: '#fff' }),
                  font: '12px Inter, Arial'
                })
              })
            } else {
              // elemento singolo
              const featureInner = Array.isArray(features) ? features[0] : feature
              return makeAdvancedStyle(featureInner)
            }
          }
        })
        map.addLayer(clusterLayer)

        // Interazione con click sui siti in overview
        map.on('singleclick', (evt) => {
          if (overviewMode.value) {
            map.forEachFeatureAtPixel(evt.pixel, (feature) => {
              const fid = feature?.getId?.()
              if (fid && String(fid).startsWith('site-')) {
                const siteId = String(fid).slice(5)
                selectSite(siteId)
                return true
              }
            })
          }
        })

        // Resize observer
        ro = new ResizeObserver(() => { 
          if (map) {
            setTimeout(() => map.updateSize(), 50)
          }
        })
        
        if (mapWrapEl.value) {
          ro.observe(mapWrapEl.value)
        }

        // Primo refresh
        setTimeout(() => {
          if (map) {
            map.updateSize()
            refreshSites()
            refreshAssets()
            // inquadra overview alla prima apertura
            if (overviewMode.value) fitAllSites()
          }
        }, 300)

      } catch (error) {
        console.error('Errore inizializzazione mappa:', error)
        addEvent('Errore inizializzazione mappa')
      }
    }, 100)
  })
}

// Refresh assets con filtri e cluster
function refreshAssets() {
  if (overviewMode.value) {
    // OVERVIEW: usa assetSrc
    if (!assetSrc) return
    
    try {
      const visibleIds = new Set()
      for (const a of filteredAssets.value) {
        const id = a.id
        const coords = (a.lon != null && a.lat != null)
          ? fromLonLat([Number(a.lon), Number(a.lat)])
          : transformDxf(a.x || 0, a.y || 0)

        let f = featureMap.get(id)
        if (!f) {
          f = new Feature({ 
            geometry: new Point(coords),
            ...a
          })
          f.setId(id)
          f.setStyle(makeAdvancedStyle(f))
          assetSrc.addFeature(f)
          featureMap.set(id, f)
        } else {
          animateFeatureTo(f, coords, 350)
          f.setStyle(makeAdvancedStyle(f))
        }
        visibleIds.add(id)
      }
      
      // Rimuovi feature non più presenti (solo overview)
      for (const [id, f] of featureMap.entries()) {
        if (!visibleIds.has(id) && assetSrc) {
          assetSrc.removeFeature(f)
          featureMap.delete(id)
        }
      }
    } catch (error) {
      console.error('Errore refresh assets (overview):', error)
    }
  } else {
    // SITE VIEW: usa siteAssetSource (+clusterSource che la avvolge)
    if (!siteAssetSource) return
    
    try {
      siteAssetSource.clear()
      
      const filtered = filteredAssets.value.filter(asset => {
        if (currentFilter.value === 'all') return true
        
        const cat = (asset.categoria || '').toLowerCase()
        switch (currentFilter.value) {
          case 'personale': return cat.includes('personale')
          case 'pazienti': return cat.includes('paziente')
          case 'dispositivi': return cat.includes('dispositivo') || cat.includes('infrastruttura') || cat.includes('sicurezza') || cat.includes('sensor')
          default: return true
        }
      })

      filtered.forEach(asset => {
        const coords = (asset.lon != null && asset.lat != null)
          ? fromLonLat([Number(asset.lon), Number(asset.lat)])
          : transformDxf(asset.x || 0, asset.y || 0)

        const feature = new Feature({
          geometry: new Point(coords),
          ...asset
        })
        feature.setId(asset.id)
        siteAssetSource.addFeature(feature)
      })

      // Aggiorna toggle cluster
      if (clusterSource?.setDistance) {
        clusterSource.setDistance(useClustering.value ? 40 : 0)
      }
      if (clusterLayer) clusterLayer.setVisible(useClustering.value)
      if (singleSiteLayer) singleSiteLayer.setVisible(!useClustering.value)

    } catch (error) {
      console.error('Errore refresh assets (site view):', error)
    }
  }
}

// Animazione movimento
function animateFeatureTo(feature, newCoords, duration = 350) {
  const geom = feature.getGeometry()
  if (!(geom instanceof Point)) return
  
  const oldCoords = geom.getCoordinates()
  const startTime = Date.now()
  
  function animate() {
    const elapsed = Date.now() - startTime
    const progress = Math.min(elapsed / duration, 1)
    const easeProgress = 1 - Math.pow(1 - progress, 3) // easing
    
    const currentCoords = [
      oldCoords[0] + (newCoords[0] - oldCoords[0]) * easeProgress,
      oldCoords[1] + (newCoords[1] - oldCoords[1]) * easeProgress
    ]
    
    geom.setCoordinates(currentCoords)
    
    if (progress < 1) {
      requestAnimationFrame(animate)
    }
  }
  
  requestAnimationFrame(animate)
}

// Refresh siti per overview
function refreshSites() {
  if (!siteSrc) return
  
  siteSrc.clear()
  if (!overviewMode.value) return

  for (const s of sitesWithCoords.value) {
    const f = new Feature({
      geometry: new Point(fromLonLat([s.lon, s.lat]))
    })
    f.setId(`site-${s.id}`)
    f.setStyle(makeSiteStyle(s))
    siteSrc.addFeature(f)
  }
}

// Navigazione
function showOverview() {
  overviewMode.value = true
  selectedSiteId.value = null
  nextTick(() => {
    refreshSites()
    refreshAssets()
    fitAllSites()
  })
}

function showSiteView() {
  if (!selectedSiteId.value) return
  overviewMode.value = false
  nextTick(() => {
    refreshSites()
    refreshAssets()
    fitAll()
  })
}

function selectSite(siteId) {
  selectedSiteId.value = siteId
  showSiteView()
}

// Fit bounds
function fitAllSites() {
  if (!map || !siteSrc) return
  const extent = siteSrc.getExtent()
  if (extent && extent.every(Number.isFinite)) {
    map.getView().fit(extent, { 
      padding: [40, 40, 40, 40],
      duration: 800,
      maxZoom: 10
    })
  }
}

function fitAll() {
  if (!map) return
  // in vista sito, calcola extent dei marker del sito
  const extent = siteAssetSource?.getExtent()
  if (extent && extent.every(Number.isFinite)) {
    map.getView().fit(extent, { 
      padding: [60, 60, 60, 60],
      duration: 800,
      maxZoom: 19
    })
  } else if (selectedSite.value?.lon && selectedSite.value?.lat) {
    const center = fromLonLat([Number(selectedSite.value.lon), Number(selectedSite.value.lat)])
    map.getView().animate({ center, zoom: 17, duration: 800 })
  }
}

// Controlli mappa
function toggleClustering() {
  useClustering.value = !useClustering.value
  refreshAssets()
}

function filterByCategory(category) {
  currentFilter.value = category
  refreshAssets()
}

// Demo e simulazioni (placeholder)
function startDemo() {
  if (demoRunning.value) return
  demoRunning.value = true
  addEvent('Demo avviata')
}

function stopDemo() {
  demoRunning.value = false
  addEvent('Demo fermata')
}

function simulateOnce() {
  addEvent('Simulazione singola sample')
}

// --- helper: route curva ---
function buildCurvedRoute(fromCoord, toCoord, maxOffset = null) {
  const dx = toCoord[0] - fromCoord[0]
  const dy = toCoord[1] - fromCoord[1]
  const len = Math.hypot(dx, dy) || 1
  const mid = [(fromCoord[0] + toCoord[0]) / 2, (fromCoord[1] + toCoord[1]) / 2]
  const nx = -dy / len
  const ny = dx / len
  const offset = Math.min(maxOffset || len * 0.2, len * 0.2)
  mid[0] += nx * offset
  mid[1] += ny * offset
  return [fromCoord, mid, toCoord]
}

// --- Simulazione ambulanza come asset animato (overview) ---
let currentSimCancel = null
let ambulanceTimer = null

function simulateVehicleRouteAsAsset(fromSiteId, toSiteId, duration = 18000) {
  if (!map || !sites) return null
  const fromSite = sites.find(s => s.id === fromSiteId)
  const toSite = sites.find(s => s.id === toSiteId)
  if (!fromSite || !toSite) {
    addEvent('Simulazione: sito non trovato')
    return null
  }

  const vehId = 'ambulance-1'
  let veh = assetList.value.find(a => a.id === vehId)

  // Crea/aggiorna asset veicolo nel dataset
  if (!veh) {
    veh = {
      id: vehId,
      nome: 'Ambulanza 1',
      categoria: 'veicolo',
      tipo: 'ambulanza',
      siteId: fromSiteId,
      lon: Number(fromSite.lon),
      lat: Number(fromSite.lat)
    }
    assetList.value.push(veh)
  } else {
    veh.categoria = 'veicolo'
    veh.tipo = 'ambulanza'
    veh.siteId = fromSiteId
    veh.lon = Number(fromSite.lon)
    veh.lat = Number(fromSite.lat)
  }

  // Feature visibile in OVERVIEW (riusa se esiste)
  let overviewFeature = null
  if (assetSrc) {
    overviewFeature = assetSrc.getFeatureById(vehId)
    const startCoord = fromLonLat([Number(fromSite.lon), Number(fromSite.lat)])
    if (!overviewFeature) {
      overviewFeature = new Feature({
        geometry: new Point(startCoord),
        ...veh
      })
      overviewFeature.setId(vehId)
      overviewFeature.setStyle(new Style({
        image: new CircleStyle({
          radius: 12,
          fill: new Fill({ color: '#ef4444' }),
          stroke: new Stroke({ color: '#fff', width: 2 })
        }),
        text: new Text({
          text: '🚑',
          offsetY: 0,
          font: '16px Arial',
          fill: new Fill({ color: '#fff' })
        })
      }))
      assetSrc.addFeature(overviewFeature)
    } else {
      // riposiziona alla partenza
      overviewFeature.getGeometry().setCoordinates(startCoord)
    }
  }

  const fromCoord = fromLonLat([Number(fromSite.lon), Number(fromSite.lat)])
  const toCoord = fromLonLat([Number(toSite.lon), Number(toSite.lat)])
  const pts = buildCurvedRoute(fromCoord, toCoord)
  const routeGeom = new LineString(pts)

  let start = null
  let rafId = null

  function step(ts) {
    if (!start) start = ts
    const t = Math.min(1, (ts - start) / duration)
    const coord = routeGeom.getCoordinateAt(t)

    if (coord) {
      if (overviewFeature) {
        overviewFeature.getGeometry().setCoordinates(coord)
      }
      const [lon, lat] = toLonLat(coord)
      veh.lon = lon
      veh.lat = lat
      veh.siteId = 'in-transit'
    }

    if (t < 1) {
      rafId = requestAnimationFrame(step)
    } else {
      // Arrivo a destinazione
      veh.siteId = toSiteId
      const [lonF, latF] = toLonLat(toCoord)
      veh.lon = lonF
      veh.lat = latF
      if (overviewFeature) {
        overviewFeature.getGeometry().setCoordinates(toCoord)
      }
      currentSimCancel = null
      addEvent(`Ambulanza: ${fromSite.ragioneSociale} → ${toSite.ragioneSociale} (arrivo)`)
    }
  }

  rafId = requestAnimationFrame(step)

  // Cleanup function (ferma animazione e rimuove risorsa)
  return () => {
    if (rafId) cancelAnimationFrame(rafId)
    try {
      const feat = assetSrc?.getFeatureById(vehId)
      if (feat && assetSrc) assetSrc.removeFeature(feat)
    } catch {}
    const idx = assetList.value.findIndex(a => a.id === vehId)
    if (idx !== -1) assetList.value.splice(idx, 1)
    currentSimCancel = null
  }
}

// Avvio automatico in overview (ciclo)
function startAmbulanceAuto() {
  if (!overviewMode.value) return
  if (!sites || sites.length < 2) return
  // evita duplicati
  if (ambulanceTimer) return

  const runOnce = () => {
    if (!overviewMode.value) return
    const i = Math.floor(Math.random() * sites.length)
    let j = Math.floor(Math.random() * sites.length)
    if (j === i) j = (j + 1) % sites.length
    const fromSiteId = sites[i].id
    const toSiteId = sites[j].id

    // stop eventuale sim in corso
    if (typeof currentSimCancel === 'function') {
      try { currentSimCancel() } catch {}
      currentSimCancel = null
    }
    currentSimCancel = simulateVehicleRouteAsAsset(fromSiteId, toSiteId, 16000)
    addEvent(`Ambulanza (auto): ${sites[i].ragioneSociale} → ${sites[j].ragioneSociale}`)

    // pianifica prossima corsa
    const gap = 4000 + Math.floor(Math.random() * 4000) // 4-8s di pausa
    ambulanceTimer = setTimeout(() => {
      ambulanceTimer = null
      runOnce()
    }, 16000 + gap)
  }

  runOnce()
}

// Stop ciclo e pulizia
function stopAmbulanceAuto() {
  if (ambulanceTimer) {
    clearTimeout(ambulanceTimer)
    ambulanceTimer = null
  }
  if (typeof currentSimCancel === 'function') {
    try { currentSimCancel() } catch {}
    currentSimCancel = null
  }
}

// (Manuale, non esposto in UI) simulazione casuale
function simulateAmbulance() {
  const all = sites
  if (!all || all.length < 2) return
  const i = Math.floor(Math.random() * all.length)
  let j = Math.floor(Math.random() * all.length)
  if (j === i) j = (j + 1) % all.length
  if (typeof currentSimCancel === 'function') {
    try { currentSimCancel() } catch {}
    currentSimCancel = null
  }
  currentSimCancel = simulateVehicleRouteAsAsset(all[i].id, all[j].id, 16000)
  addEvent(`Ambulanza (manuale): ${all[i].ragioneSociale} → ${all[j].ragioneSociale}`)
}


// Modals e panels
const cameraModal = ref({ show: false, url: null })
const qualPanel = ref({ show: false, person: null, skills: [] })
const maintPanel = ref({ show: false, asset: null })
const visitsModal = ref({ show: false, asset: null })
const floorPlan = ref({ show: false, siteId: null, dept: null, floor: null, assets: [] })

function openCamera(url) {
  if (!url) return
  cameraModal.value = { show: true, url }
}

function openQualify(person) {
  const personSkills = skillsByPerson[person.id] || []
  qualPanel.value = { show: true, person, skills: personSkills }
}
function openQualifyById(personId) {
  const person = staffList.value.find(p => p.id === personId)
  if (person) openQualify(person)
}

function openMaintenanceFor(asset) {
  maintPanel.value = { show: true, asset }
}

function openMainFloorPlan() {
  if (!selectedSite.value) return
  floorPlan.value = {
    show: true,
    siteId: selectedSite.value.id,
    dept: 'Principale',
    floor: 1,
    assets: filteredAssets.value
  }
}

function openFloorPlan(siteId, dept, floor) {
  const assetsInDept = filteredAssets.value.filter(a => getDeptKey(a) === dept)
  floorPlan.value = {
    show: true,
    siteId,
    dept,
    floor,
    assets: assetsInDept
  }
}

function handleWardEntry(event) {
  addEvent(`Ingresso in reparto ${event.ward} da ${event.personName}`)
}

function handleAlert(event) {
  addEvent(`🔔 Allarme: ${event.message}`)
}

// Helper functions
function isHealthcare(a) {
  return (a.categoria || '').toLowerCase().includes('personale')
}

function isPatient(a) {
  return (a.categoria || '').toLowerCase().includes('paziente')
}

function isVisitor(a) {
  const c = (a.categoria || '').toLowerCase()
  return c.includes('visitat') // 'visitatore' o 'visitatori'
}

// Nuovo: mostra il bottone Manutenzione per tutti tranne personale, pazienti e visitatori
function shouldShowMaintenance(a) {
  return !isHealthcare(a) && !isPatient(a) && !isVisitor(a)
}

function isMaintainable(a) {
  const tipo = (a.tipo || '').toLowerCase()
  // Include estintori
  return tipo.includes('ecg') || tipo.includes('monitor') || tipo.includes('defibrillatore') || 
         tipo.includes('pompa') || tipo.includes('ventilatore') || tipo.includes('dispositivo') ||
         tipo.includes('estintore')||
         tipo.includes('carrello')
}

function getStatusClass(status) {
  const s = String(status).toLowerCase()
  if (s.includes('ottimo') || s.includes('operativo')) return 'status-ok'
  if (s.includes('buono') || s.includes('attivo')) return 'status-good'
  if (s.includes('attenzione') || s.includes('manutenzione')) return 'status-warning'
  if (s.includes('critico') || s.includes('guasto')) return 'status-error'
  return 'status-unknown'
}

function getDeptStatus(items) {
  const hasCritical = items.some(a => 
    a.stato && ['guasto', 'critico'].some(s => a.stato.toLowerCase().includes(s))
  )
  const hasMaintenance = items.some(a => 
    a.stato && a.stato.toLowerCase().includes('manutenzione')
  )
  
  if (hasCritical) return 'status-critical'
  if (hasMaintenance) return 'status-maintenance'
  return 'status-ok'
}

function getDeptStatusText(items) {
  const status = getDeptStatus(items)
  switch (status) {
    case 'status-critical': return 'Critico'
    case 'status-maintenance': return 'Manutenzione'
    default: return 'Operativo'
  }
}

function daysOfStay(patient) {
  if (!patient.admissionDate) return 0
  const admission = new Date(patient.admissionDate)
  const today = new Date()
  return Math.floor((today - admission) / (1000 * 60 * 60 * 24))
}

function openPatientRecord(patient) {
  if (patient.ehrUrl) {
    window.open(patient.ehrUrl, '_blank')
  }
}

// Overview dashboard (resta come nel tuo file)
const overviewCards = computed(() => {
  return sites.map(site => {
    const siteAssets = assetList.value.filter(a => a.siteId === site.id)
    const counts = {
      sanitari: siteAssets.filter(a => isHealthcare(a)).length,
      dispositivi: siteAssets.filter(a => isMaintainable(a)).length,
      pazienti: siteAssets.filter(a => isPatient(a)).length,
      visitatori: siteAssets.filter(a => (a.categoria || '').includes('visitatore')).length
    }
    
    const totals = { risorse: siteAssets.length }
    const utilization = totals.risorse > 0 ? (counts.sanitari + counts.dispositivi) / totals.risorse : 0
    
    const prestazioni = { degenza: counts.pazienti, erogate: Math.floor(Math.random() * 50) + 20 }
    const sicurezza = {
      statusText: Math.random() > 0.2 ? 'SICURO' : 'ATTENZIONE',
      statusClass: Math.random() > 0.2 ? 'status-ok' : 'status-warning',
      antincendio: Math.random() > 0.1 ? 'OK' : 'DA VERIFICARE',
      antincendioClass: Math.random() > 0.1 ? 'badge-green' : 'badge-amber',
      manutenzioniInCorso: Math.floor(Math.random() * 3),
      manutenzioneClass: Math.random() > 0.7 ? 'status-warning' : 'status-ok'
    }
    const skills = { acquired: Math.floor(Math.random() * 50) + 100, expiring: Math.floor(Math.random() * 10), todo: Math.floor(Math.random() * 5) }
    const riskScore = Math.floor(Math.random() * 30) + 10
    const risk = {
      score: riskScore,
      levelClass: riskScore < 20 ? 'risk-low' : riskScore < 40 ? 'risk-medium' : 'risk-high',
      levelText: riskScore < 20 ? 'BASSO' : riskScore < 40 ? 'MEDIO' : 'ALTO'
    }
    return { id: site.id, name: site.ragioneSociale, counts, totals, utilization, prestazioni, sicurezza, skills, risk }
  })
})

function getUtilizationClass(utilization) {
  if (utilization > 0.8) return 'utilization-high'
  if (utilization > 0.5) return 'utilization-medium'
  return 'utilization-low'
}

// Lifecycle: avvio con 'sorrisi' selezionata in vista sito e demo attiva
onMounted(() => {
  // seleziona 'sorrisi' se presente e passa subito alla vista sito
  const defaultSite = sites.find(s => s.id === 'sorrisi') || sites[0] || null
  if (defaultSite) selectedSiteId.value = defaultSite.id
  overviewMode.value = false
  initMap()
  // avvia la demo e centra la mappa quando i layer sono pronti
  setTimeout(() => {
    startDemo()
    refreshSites()
    refreshAssets()
    fitAll()
  }, 700)
})

onUnmounted(() => {
  if (ro && mapWrapEl.value) ro.unobserve(mapWrapEl.value)
  stopAmbulanceAuto()
})

// Watchers
watch([overviewMode, selectedSiteId], () => {
  nextTick(() => {
    refreshSites()
    refreshAssets()
    // Gestione auto-ambulanza on/off a seconda della vista
    if (overviewMode.value) {
      startAmbulanceAuto()
    } else {
      stopAmbulanceAuto()
    }
  })
})

watch([useClustering, currentFilter], () => {
  if (!overviewMode.value) {
    refreshAssets()
  }
})

watch(assetList, () => {
  refreshAssets()
}, { deep: true })
</script>

<style scoped>
.demo-root {
  display: flex;
  height: 100vh;
  font-family: 'Inter', sans-serif;
  background: #0f172a;
  color: #e2e8f0;
}

.sidebar {
  width: 360px;
  background: #1e293b;
  border-right: 1px solid #334155;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.brand {
  padding: 20px;
  font-size: 18px;
  font-weight: 700;
  background: #0f172a;
  color: #38bdf8;
  border-bottom: 1px solid #334155;
}

/* Bottone link esterno sotto il brand */
.external-link-wrapper {
  padding: 12px 20px 0 20px;
  border-bottom: 1px solid #334155;
}
.top-link-btn {
  display: inline-block;
  width: 100%;
  text-align: center;
  padding: 10px 12px;
  background: #3b82f6;
  color: #0f172a;
  font-weight: 700;
  border: 1px solid #60a5fa;
  border-radius: 8px;
  text-decoration: none;
}
.top-link-btn:hover {
  background: #2563eb;
  color: #fff;
}

.controls {
  padding: 20px;
  border-bottom: 1px solid #334155;
}

.controls label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
  color: #cbd5e1;
}

.controls select {
  width: 100%;
  padding: 10px;
  margin-bottom: 16px;
  background: #334155;
  border: 1px solid #475569;
  border-radius: 6px;
  color: #e2e8f0;
}

.btn-row {
  display: flex;
  gap: 8px;
  margin-bottom: 12px;
}

.btn-row button {
  flex: 1;
  padding: 10px;
  background: #475569;
  border: 1px solid #64748b;
  border-radius: 6px;
  color: #e2e8f0;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-row button:hover:not(:disabled) {
  background: #64748b;
  border-color: #94a3b8;
}

.btn-row button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.events {
  flex: 1;
  padding: 20px;
  overflow-y: auto;
}

.events h4 {
  margin-bottom: 12px;
  color: #cbd5e1;
}

.events ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.events li {
  padding: 8px 0;
  border-bottom: 1px solid #334155;
  font-size: 13px;
  line-height: 1.4;
}

.events li small {
  color: #94a3b8;
}

.main {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.map-wrapper {
  position: relative;
  flex: 1;
  min-height: 320px;
}

#map {
  width: 100%;
  height: 100%;
  background: #1e293b;
}

/* Controlli mappa avanzati */
.map-controls {
  position: absolute;
  top: 20px;
  right: 20px;
  z-index: 1000;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.control-group {
  display: flex;
  flex-direction: column;
  gap: 5px;
  background: rgba(15, 23, 42, 0.9);
  padding: 10px;
  border-radius: 8px;
  border: 1px solid #334155;
}

.control-btn {
  padding: 8px 12px;
  background: #475569;
  border: 1px solid #64748b;
  border-radius: 6px;
  color: #e2e8f0;
  cursor: pointer;
  font-size: 12px;
  transition: all 0.2s;
  white-space: nowrap;
}

.control-btn:hover {
  background: #64748b;
}

.control-btn.active {
  background: #3b82f6;
  border-color: #60a5fa;
}

.control-btn.category-personale.active {
  background: #2563eb;
}

.control-btn.category-pazienti.active {
  background: #0891b2;
}

.control-btn.category-dispositivi.active {
  background: #16a34a;
}

/* Legenda mappa */
.map-legend {
  position: absolute;
  bottom: 20px;
  left: 20px;
  z-index: 1000;
  background: rgba(15, 23, 42, 0.9);
  padding: 15px;
  border-radius: 8px;
  border: 1px solid #334155;
  min-width: 150px;
}

.legend-title {
  font-weight: 600;
  margin-bottom: 10px;
  color: #cbd5e1;
  font-size: 14px;
}

.legend-items {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 12px;
}

.legend-color {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  border: 2px solid #fff;
}

.legend-color.personale { background: #2563eb; }
.legend-color.pazienti { background: #0891b2; }
.legend-color.dispositivi { background: #16a34a; }
.legend-color.sicurezza { background: #dc2626; }

/* Overview dashboard */
.overview {
  flex: 1;
  padding: 20px;
  overflow-y: auto;
}

.overview h3 {
  margin-bottom: 20px;
  color: #e2e8f0;
}

.sites-list {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
  gap: 20px;
}

.site-card {
  background: #1e293b;
  border: 1px solid #334155;
  border-radius: 8px;
  padding: 20px;
}

.site-head {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding-bottom: 15px;
  border-bottom: 1px solid #334155;
}

.site-name {
  font-size: 18px;
  font-weight: 700;
  color: #e2e8f0;
}

.link {
  background: none;
  border: none;
  color: #3b82f6;
  cursor: pointer;
  text-decoration: underline;
  font-size: 14px;
}

.link:hover {
  color: #60a5fa;
}

.kpi-section {
  margin-bottom: 20px;
}

.kpi-header {
  font-weight: 600;
  margin-bottom: 12px;
  color: #cbd5e1;
  font-size: 14px;
}

.kpi-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

.kpi-item {
  background: #0f172a;
  padding: 12px;
  border-radius: 6px;
  border: 1px solid #334155;
}

.kpi-label {
  font-size: 12px;
  color: #94a3b8;
  margin-bottom: 4px;
}

.kpi-value {
  font-size: 20px;
  font-weight: 700;
  margin-bottom: 6px;
}

.kpi-detail {
  display: flex;
  flex-wrap: wrap;
  gap: 4px;
}

.badge {
  padding: 2px 6px;
  border-radius: 4px;
  font-size: 11px;
  font-weight: 600;
}

.badge.blue { background: #1e3a8a; color: #dbeafe; }
.badge.green { background: #065f46; color: #d1fae5; }
.badge.amber { background: #92400e; color: #fef3c7; }
.badge.red { background: #991b1b; color: #fecaca; }

.utilization-high { color: #ef4444; }
.utilization-medium { color: #f59e0b; }
.utilization-low { color: #10b981; }

.status-ok { color: #10b981; }
.status-good { color: #22c55e; }
.status-warning { color: #f59e0b; }
.status-error { color: #ef4444; }
.status-unknown { color: #6b7280; }

.risk-indicator {
  margin-top: 20px;
  padding-top: 15px;
  border-top: 1px solid #334155;
}

.risk-label {
  font-size: 12px;
  color: #94a3b8;
  margin-bottom: 4px;
}

.risk-score {
  font-size: 24px;
  font-weight: 700;
  display: flex;
  align-items: center;
  gap: 8px;
}

.risk-level {
  font-size: 12px;
  padding: 2px 8px;
  border-radius: 12px;
  background: #065f46;
  color: #d1fae5;
}

.risk-low { color: #10b981; }
.risk-medium { color: #f59e0b; }
.risk-high { color: #ef4444; }

.risk-low .risk-level { background: #065f46; }
.risk-medium .risk-level { background: #92400e; }
.risk-high .risk-level { background: #991b1b; }

.extra-actions {
  margin-top: 15px;
  padding-top: 15px;
  border-top: 1px solid #334155;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

/* Site dashboard */
.site-dashboard {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: auto;
}

.site-header {
  background: #1e293b;
  border-bottom: 1px solid #334155;
  padding: 20px;
}
.site-header.sticky { position: sticky; top: 0; z-index: 5; }

.site-info h2 {
  margin-bottom: 15px;
  color: #e2e8f0;
}

.site-stats {
  display: flex;
  gap: 30px;
}

.stat {
  text-align: center;
}

.stat-value {
  font-size: 24px;
  font-weight: 700;
  margin-bottom: 4px;
}

.stat-label {
  font-size: 12px;
  color: #94a3b8;
}

.site-actions {
  margin-top: 15px;
  display: flex;
  gap: 10px;
}

.btn-primary {
  background: #3b82f6;
  border: 1px solid #60a5fa;
  color: white;
  padding: 10px 16px;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 600;
}

.btn-primary:hover {
  background: #2563eb;
}

.btn-secondary {
  background: #475569;
  border: 1px solid #64748b;
  color: #e2e8f0;
  padding: 10px 16px;
  border-radius: 6px;
  cursor: pointer;
}

.btn-secondary:hover {
  background: #64748b;
}

/* Tabella risorse (tema scuro, leggibile) */
.dept-panel { margin-top: 10px; background: #0f172a; border-top: 1px solid #334155; border-bottom: 1px solid #334155; }
.dept-header { display: flex; align-items: center; padding: 12px 16px; border-bottom: 1px solid #334155; background: #0f172a; gap: 10px; flex-wrap: wrap; position: relative; z-index: 6; }
.dept-header.sticky { position: sticky; top: 72px; } /* resta visibile durante lo scroll */
.dept-header h3 { margin: 0; color: #e2e8f0; }
.dept-header .spacer { flex: 1; }
.dept-header .link { color: #60a5fa; }

.filters { display: flex; align-items: center; gap: 8px; }
.filter-input {
  width: 260px;
  max-width: 70vw;
  padding: 8px 10px;
  background: #0b152b;
  border: 1px solid #243244;
  border-radius: 6px;
  color: #e2e8f0;
}
.link.clear { color: #93c5fd; }

.chips { display: flex; gap: 6px; align-items: center; flex-wrap: wrap; }
.chip { padding: 6px 10px; border-radius: 999px; border: 1px solid #334155; background: #132641; color: #e2e8f0; font-size: 12px; cursor: pointer; }
.chip.active { background: #2563eb; border-color: #3b82f6; }

.quick-actions { display: flex; gap: 8px; align-items: center; flex-wrap: wrap; }
.qa-select {
  padding: 6px 8px;
  background: #0b152b;
  border: 1px solid #243244;
  border-radius: 6px;
  color: #e2e8f0;
  min-width: 200px;
}
.qa-btn {
  padding: 6px 10px;
  border-radius: 6px;
  border: 1px solid #334155;
  background: #1e293b;
  color: #e2e8f0;
  cursor: pointer;
}
.qa-btn:disabled { opacity: 0.6; cursor: not-allowed; }
.qa-btn:not(:disabled):hover { background: #243446; border-color: #3d4e63; }

.accordion {
  display: flex;
  flex-direction: column;
}

.section {
  border-bottom: 1px solid #182338;
  background: #0f172a;
}

.section-head {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 14px 20px;
  cursor: pointer;
  background: #0b1324;
  border-bottom: 1px solid #0f1a32;
}

.section-head:hover {
  background: #0d1831;
}

.section-head .title {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #e2e8f0;
}

.section-head .chev {
  transition: transform 0.2s;
}

.section-head .chev.open {
  transform: rotate(90deg);
}

.section-head .badge {
  margin-left: auto;
  background: #1e293b;
  border: 1px solid #334155;
  color: #cbd5e1;
  border-radius: 10px;
  padding: 2px 8px;
  font-size: 12px;
}

.section-head .actions {
  margin-left: 10px;
}

.section-head .actions .link {
  color: #38bdf8;
  text-decoration: underline;
}

.section-body {
  padding: 0;
}

.resources-table {
  width: 100%;
  border-collapse: collapse;
  background: #0f172a;
  table-layout: fixed;
}

.resources-table thead th {
  text-align: left;
  background: #0b1324;
  color: #cbd5e1;
  font-size: 12px;
  padding: 10px 16px;
  border-bottom: 1px solid #16233f;
  position: sticky;
  top: 0;
  z-index: 3;
}

.resources-table .col-name { width: 32%; }
.resources-table .col-cat { width: 18%; }
.resources-table .col-room { width: 18%; }
.resources-table .col-state { width: 14%; }
.th-actions { width: 18%; }

.resources-table tbody td {
  padding: 10px 16px;
  border-bottom: 1px solid #16233f;
  color: #e2e8f0;
  font-size: 13px;
  word-break: break-word;
}

.resources-table tbody tr:nth-child(even) {
  background: #0e1a32;
}

.resources-table tbody tr:hover {
  background: #0d1831;
}

.cell-name strong {
  display: block;
  color: #e2e8f0;
}

.cell-name .muted {
  color: #94a3b8;
  font-size: 11px;
}

.td-actions {
  text-align: left;
  white-space: nowrap;
}

.btn-ghost {
  background: #1e293b;
  color: #e2e8f0;
  border: 1px solid #334155;
  border-radius: 6px;
  padding: 6px 10px;
  margin-right: 6px;
  cursor: pointer;
  font-size: 12px;
}

.btn-ghost:hover:not(:disabled) {
  background: #243446;
  border-color: #3d4e63;
}

.btn-ghost:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.pill {
  display: inline-block;
  padding: 2px 8px;
  border-radius: 999px;
  font-size: 11px;
  border: 1px solid #334155;
  background: #12203a;
  color: #cbd5e1;
}

.status-ok.pill,
.pill.status-ok { background: #064e3b; color: #d1fae5; border-color: #065f46; }
.status-good.pill,
.pill.status-good { background: #14532d; color: #bbf7d0; border-color: #047857; }
.status-warning.pill,
.pill.status-warning { background: #7c2d12; color: #fde68a; border-color: #92400e; }
.status-error.pill,
.pill.status-error { background: #7f1d1d; color: #fecaca; border-color: #991b1b; }

.empty {
  padding: 16px 20px;
  color: #94a3b8;
  font-size: 14px;
}

/* Responsive */
@media (max-width: 768px) {
  .demo-root {
    flex-direction: column;
  }
  
  .sidebar {
    width: 100%;
    height: 40vh;
  }
  
  .sites-list {
    grid-template-columns: 1fr;
  }
  
  .kpi-grid {
    grid-template-columns: 1fr;
  }
  
  .site-stats {
    flex-wrap: wrap;
    gap: 15px;
  }
  
  .map-controls {
    top: 10px;
    right: 10px;
  }
  
  .map-legend {
    bottom: 10px;
    left: 10px;
  }

  .filter-input { width: 100%; max-width: 100%; }
  .resources-table thead th,
  .resources-table tbody td {
    padding: 10px 12px;
  }
}
</style>