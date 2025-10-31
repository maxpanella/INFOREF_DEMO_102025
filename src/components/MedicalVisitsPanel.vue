<template>
  <div class="mv-backdrop" @click.self="$emit('close')">
    <div class="mv-modal">
      <div class="mv-head">
        <strong>Visite mediche — {{ person?.nome || person?.id }}</strong>
        <div class="spacer"></div>
        <button class="link" @click="$emit('close')">Chiudi</button>
      </div>

      <div class="mv-body">
        <div v-if="(visits || []).length === 0" class="empty">
          Nessuna visita pianificata nel dataset. Aggiungi elementi in skills.js (todo/expiring).
        </div>
        <ul v-else class="list">
          <li v-for="(v,i) in visits" :key="i" class="row">
            <div class="name">🩺 {{ v.name }}</div>
            <div class="meta">
              <span v-if="v.date" class="badge">Fatta: {{ v.date }}</span>
              <span v-else-if="v.due" class="badge due">Scadenza: {{ v.due }}</span>
            </div>
          </li>
        </ul>
      </div>

      <div class="mv-foot">
        <button class="primary" @click="$emit('close')">Ok</button>
      </div>
    </div>
  </div>
</template>

<script setup>
defineProps({
  person: Object,
  visits: { type: Array, default: () => [] }
})
</script>

<style scoped>
.mv-backdrop{position:fixed;inset:0;background:rgba(0,0,0,.5);display:flex;align-items:center;justify-content:center;z-index:1000}
.mv-modal{width:min(720px,92vw);max-height:88vh;overflow:auto;background:#fff;border-radius:10px;box-shadow:0 10px 30px rgba(0,0,0,.25)}
.mv-head{display:flex;align-items:center;gap:8px;padding:10px 12px;border-bottom:1px solid #eee}
.mv-head .spacer{flex:1}
.link{background:none;border:none;color:#2563eb;cursor:pointer}
.mv-body{padding:12px}
.empty{color:#64748b}
.list{list-style:none;margin:0;padding:0;display:flex;flex-direction:column;gap:8px}
.row{display:flex;align-items:center;justify-content:space-between;border:1px solid #eef2f7;border-radius:8px;padding:8px 10px;background:#fafafa}
.badge{background:#e2e8f0;color:#111827;border-radius:999px;padding:2px 8px;font-size:12px}
.badge.due{background:#fde68a}
.mv-foot{display:flex;justify-content:flex-end;padding:10px 12px;border-top:1px solid #eee}
.primary{background:#2563eb;color:#fff;border:none;border-radius:6px;padding:8px 12px;cursor:pointer}
</style>