---
type: health-dashboard
description: Dashboard de saúde da memória (M1-M13) — gerado por health_dashboard.py
generated_at: 2026-06-19 11:39
---
# Dashboard de Saúde da Memória

> Gerado em 2026-06-19 11:39 por `health_dashboard.py --generate-dashboard`.
> Requer plugin **Dataview** ativo no Obsidian para renderizar os blocos abaixo.

## Últimos 7 dias — métricas principais

```dataviewjs
const saude = '"cortex/insula/saude"';
const pages = dv.pages(saude)
  .filter(p => /^\d{4}-\d{2}-\d{2}$/.test(p.file.name))
  .sort(p => p.file.name, 'desc')
  .slice(0, 7);

function extractMetric(content, label) {
  const re = new RegExp('\\| ' + label + ' \\| ([^|]+) \\|');
  const m = content.match(re);
  return m ? m[1].trim() : 'n/a';
}

const rows = [];
for (const page of pages) {
  const file = app.vault.getAbstractFileByPath(page.file.path);
  const content = await app.vault.read(file);
  rows.push([
    page.file.name,
    extractMetric(content, 'M1 átomos/dia'),
    extractMetric(content, 'M2 daily logs \\(7d\\)'),
    extractMetric(content, 'M4 órfãos %'),
    extractMetric(content, 'M9 dream survival'),
    extractMetric(content, 'M12 conflitos abertos'),
    extractMetric(content, 'M13 alertas despachados \\(hoje\\)'),
  ]);
}

dv.table(
  ['Data', 'M1 átomos', 'M2 daily(7d)', 'M4 órfãos%', 'M9 dream', 'M12 conflitos', 'M13 alertas'],
  rows
);
```

## Histórico de alertas — últimos 30 dias

```dataviewjs
const saude = '"cortex/insula/saude"';
const pages = dv.pages(saude)
  .filter(p => /^\d{4}-\d{2}-\d{2}$/.test(p.file.name))
  .sort(p => p.file.name, 'desc')
  .slice(0, 30);

function extractAlerts(content) {
  const m = content.match(/## Alertas\n([\s\S]*?)(?=\n## |$)/);
  if (!m) return [];
  return m[1].split('\n')
    .filter(l => l.includes('⚠️'))
    .map(l => l.replace(/^-\s*⚠️\s*/, '').trim())
    .filter(Boolean);
}

const alertRows = [];
for (const page of pages) {
  const file = app.vault.getAbstractFileByPath(page.file.path);
  const content = await app.vault.read(file);
  for (const alert of extractAlerts(content)) {
    alertRows.push([page.file.name, alert]);
  }
}

if (alertRows.length === 0) {
  dv.paragraph('✅ Nenhum alerta nos últimos 30 dias.');
} else {
  dv.table(['Data', 'Alerta'], alertRows);
}
```

## Tendência de M1 (átomos/dia) — últimos 30 dias

```dataviewjs
const saude = '"cortex/insula/saude"';
const pages = dv.pages(saude)
  .filter(p => /^\d{4}-\d{2}-\d{2}$/.test(p.file.name))
  .sort(p => p.file.name, 'asc')
  .slice(-30);

const rows = [];
for (const page of pages) {
  const file = app.vault.getAbstractFileByPath(page.file.path);
  const content = await app.vault.read(file);
  const m = content.match(/\| M1 átomos\/dia \| ([^|]+) \|/);
  const v = m ? m[1].trim() : 'n/a';
  rows.push([page.file.name, v === 'n/a' ? v : Number(v)]);
}

dv.table(['Data', 'M1 átomos/dia'], rows);
```
