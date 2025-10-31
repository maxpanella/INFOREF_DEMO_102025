<template>
  <div class="overlay" @click.self="emitClose">
    <section class="panel" role="dialog" aria-modal="true">
      <header class="panel-header">
        <!-- Cambiato in "Formazione" -->
        <h3>Formazione — {{ person?.nome || 'Sconosciuto' }}</h3>
        <button class="btn-close" @click="emitClose" aria-label="Chiudi">✕</button>
      </header>

      <div class="panel-body">
        <div class="grid">
          <div class="card">
            <div class="card-title">Dati Personali</div>
            <div class="kv">
              <div class="row"><span class="k">ID</span><span class="v">{{ person?.id || '-' }}</span></div>
              <div class="row"><span class="k">Ruolo</span><span class="v">{{ person?.tipo || '-' }}</span></div>
              <div class="row"><span class="k">Reparto</span><span class="v">{{ person?.reparto || '-' }}</span></div>
            </div>
          </div>

          <div class="card">
            <div class="card-title">Formazione e Competenze</div>
            <template v-if="Array.isArray(skills)">
              <ul class="bullets" v-if="skills.length">
                <li v-for="(s,i) in skills" :key="i">{{ itemText(s) }}</li>
              </ul>
              <div v-else class="muted">Nessuna voce disponibile</div>
            </template>
            <template v-else>
              <div class="cols">
                <div>
                  <div class="sub">Acquisite</div>
                  <ul class="bullets">
                    <li v-for="(s,i) in (skills?.acquired || [])" :key="'a'+i">{{ itemText(s) }}</li>
                  </ul>
                </div>
                <div>
                  <div class="sub">Da fare</div>
                  <ul class="bullets">
                    <li v-for="(s,i) in (skills?.todo || [])" :key="'t'+i">{{ itemText(s) }}</li>
                  </ul>
                </div>
                <div>
                  <div class="sub">In scadenza</div>
                  <ul class="bullets">
                    <li v-for="(s,i) in (skills?.expiring || [])" :key="'e'+i">{{ itemText(s) }}</li>
                  </ul>
                </div>
              </div>
            </template>
          </div>

          <div class="card">
            <div class="card-title">Note</div>
            <div class="muted">Aggiungi corsi, ECM, attestati, scadenze di aggiornamento, ecc.</div>
          </div>
        </div>
      </div>

      <footer class="panel-footer">
        <button class="btn ghost" @click="emitClose">Chiudi</button>
      </footer>
    </section>
  </div>
</template>

<script setup>
const props = defineProps({
  person: { type: Object, default: null },
  skills: { type: [Array, Object], default: () => [] }
})
const emit = defineEmits(['close'])
function emitClose(){ emit('close') }
function itemText(s){
  if (!s) return '-'
  if (typeof s === 'string') return s
  if (typeof s === 'object') return s.name || s.title || JSON.stringify(s)
  return String(s)
}
</script>

<style scoped>
.overlay{position:fixed;inset:0;background:rgba(2,6,23,.7);backdrop-filter:saturate(120%) blur(2px);display:flex;align-items:center;justify-content:center;z-index:2000}
.panel{width:min(920px,96vw);max-height:90vh;background:#0b1324;border:1px solid #334155;border-radius:10px;display:flex;flex-direction:column;color:#e2e8f0}
.panel-header{display:flex;align-items:center;justify-content:space-between;padding:14px 16px;border-bottom:1px solid #243244;background:#0e1931}
.panel-header h3{margin:0;font-size:16px}
.btn-close{background:#1f2937;border:1px solid #334155;color:#e2e8f0;border-radius:6px;padding:6px 10px;cursor:pointer}
.btn-close:hover{background:#273447}
.panel-body{padding:16px;overflow:auto}
.grid{display:grid;grid-template-columns:1fr;gap:12px}
@media(min-width:900px){.grid{grid-template-columns:1fr 1fr 1fr}}
.card{background:#0f172a;border:1px solid #243244;border-radius:8px;padding:12px}
.card-title{font-weight:700;margin-bottom:8px;color:#e2e8f0}
.kv .row{display:flex;justify-content:space-between;padding:6px 0;border-bottom:1px dashed #22324b}
.kv .row:last-child{border-bottom:none}
.k{color:#93a4bf}
.v{color:#e2e8f0;font-weight:600}
.cols{display:grid;grid-template-columns:1fr;gap:10px}
@media(min-width:700px){.cols{grid-template-columns:1fr 1fr 1fr}}
.sub{font-size:12px;color:#93a4bf;margin-bottom:6px}
.bullets{margin:0;padding-left:16px;display:flex;flex-direction:column;gap:4px}
.muted{color:#94a3b8}
.panel-footer{padding:12px 16px;border-top:1px solid #243244;background:#0e1931;display:flex;justify-content:flex-end;gap:8px}
.btn.ghost{background:#1e293b;border:1px solid #334155;color:#e2e8f0;border-radius:6px;padding:8px 12px;cursor:pointer}
.btn.ghost:hover{background:#243446;border-color:#3d4e63}
</style>