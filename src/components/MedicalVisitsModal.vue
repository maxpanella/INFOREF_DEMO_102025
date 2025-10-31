<template>
  <div class="mp-overlay" @click.self="$emit('close')">
    <!-- Nota: .mp-modal è stilata globalmente da te (color: black) -->
    <section class="mp-modal" role="dialog" aria-modal="true">
      <header class="mv-header">
        <h3>Visite mediche — {{ asset?.nome || '—' }}</h3>
        <button class="mv-close" @click="$emit('close')" aria-label="Chiudi">✕</button>
      </header>

      <div class="mv-body">
        <div v-if="asset?.visitsUrl" class="mv-info">
          Per questo soggetto è disponibile un portale esterno.
          <a :href="asset.visitsUrl" target="_blank" rel="noopener">Apri portale visite</a>
        </div>

        <div class="mv-grid">
          <div class="mv-card">
            <div class="mv-card-title">Prossime visite</div>
            <ul class="mv-list">
              <li v-for="(v,i) in nextVisits" :key="'n'+i">
                <strong>{{ v.titolo }}</strong>
                <div class="muted">{{ v.data }} — {{ v.luogo }}</div>
              </li>
              <li v-if="!nextVisits.length" class="muted">Nessuna visita programmata</li>
            </ul>
          </div>

          <div class="mv-card">
            <div class="mv-card-title">Storico</div>
            <ul class="mv-list">
              <li v-for="(v,i) in pastVisits" :key="'p'+i">
                <strong>{{ v.titolo }}</strong>
                <div class="muted">{{ v.data }} — Esito: {{ v.esito }}</div>
              </li>
              <li v-if="!pastVisits.length" class="muted">Nessuna visita registrata</li>
            </ul>
          </div>
        </div>
      </div>

      <footer class="mv-footer">
        <button class="mv-btn" @click="$emit('close')">Chiudi</button>
      </footer>
    </section>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  asset: { type: Object, default: null }
})

// Dati dimostrativi: in integrazione reale popolare da API
const nextVisits = computed(() => {
  const baseName = props.asset?.nome || '—'
  return [
    { titolo: `Controllo periodico — ${baseName}`, data: new Date(Date.now() + 7*86400000).toLocaleString(), luogo: 'Ambulatorio 2' },
    { titolo: `Esami diagnostici — ${baseName}`, data: new Date(Date.now() + 15*86400000).toLocaleString(), luogo: 'Radiologia' }
  ]
})

const pastVisits = computed(() => [
  { titolo: 'Visita di controllo', data: new Date(Date.now() - 20*86400000).toLocaleDateString(), esito: 'OK' },
  { titolo: 'ECG basale', data: new Date(Date.now() - 45*86400000).toLocaleDateString(), esito: 'Nella norma' }
])
</script>

<style scoped>
/* overlay neutro; non tocchiamo .mp-modal per rispettare il tuo color: black */
.mp-overlay {
  position: fixed;
  inset: 0;
  background: rgba(2,6,23,.55);
  backdrop-filter: blur(2px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 3000;
}

.mv-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 14px 16px;
  border-bottom: 1px solid #e5e7eb;
  background: #f8fafc;
}

.mv-header h3 {
  margin: 0;
  font-size: 16px;
}

.mv-close {
  background: #fff;
  border: 1px solid #e5e7eb;
  border-radius: 6px;
  padding: 6px 10px;
  cursor: pointer;
}

.mv-body {
  padding: 16px;
}

.mv-info {
  margin-bottom: 10px;
}

.mv-info a {
  margin-left: 6px;
  color: #2563eb;
  text-decoration: underline;
}

.mv-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 12px;
}

@media (min-width: 800px) {
  .mv-grid { grid-template-columns: 1fr 1fr; }
}

.mv-card {
  background: #ffffff;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  padding: 12px;
}

.mv-card-title {
  font-weight: 700;
  margin-bottom: 8px;
}

.mv-list {
  list-style: none;
  margin: 0;
  padding: 0;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.muted {
  color: #64748b;
  font-size: 12px;
}

.mv-footer {
  padding: 12px 16px;
  border-top: 1px solid #e5e7eb;
  display: flex;
  justify-content: flex-end;
  gap: 8px;
  background: #f8fafc;
}

.mv-btn {
  background: #1f2937;
  color: #fff;
  border: 1px solid #111827;
  border-radius: 6px;
  padding: 8px 12px;
  cursor: pointer;
}
.mv-btn:hover { background: #111827; }
</style>