<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { causeOf, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  type Problem = 'parser' | 'hint' | 'shift';
  let { ticket, links }: { ticket: number; links: Record<Problem, string> } = $props();

  const LABEL: Record<Problem, string> = {
    parser: 'Month-first parser flips day and month',
    hint: 'booking-v3 hint asks for month first, form reads day first',
    shift: 'Dates move by one day, not flipped',
  };
  const KEYS = Object.keys(LABEL) as Problem[];

  // The desk's cause (causes.ts), except the reports it footnotes as a one-day move (desk-notes.json).
  const problemOf = (r: Report): Problem => (notes.oneDayMove.includes(r.id) ? 'shift' : causeOf(r, tenantOf(r)) === 'hint' ? 'hint' : 'parser');

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const rows = $derived(
    KEYS.map((key) => {
      const rs = merged.filter((r) => problemOf(r) === key);
      const days = rs.map((r) => r.receivedAt.slice(0, 10)).sort();
      return { key, count: rs.length, tenants: new Set(rs.map((r) => r.tenant)).size, first: days[0], last: days[days.length - 1] };
    }),
  );
  const months = $derived.by(() => {
    const all = [...new Set(merged.map((r) => r.receivedAt.slice(0, 7)))].sort();
    return all.map((month) => ({
      month,
      counts: KEYS.map((key) => merged.filter((r) => r.receivedAt.startsWith(month) && problemOf(r) === key).length),
    }));
  });
  const max = $derived(Math.max(...months.map((m) => m.counts.reduce((a, b) => a + b, 0))));
  const h = (n: number) => (n / max) * 100;
</script>

<p class="sum"><strong>{merged.length}</strong> reports · <strong>{rows.length}</strong> problems · each report counted once</p>

<table>
  <thead><tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th></tr></thead>
  <tbody>
    {#each rows as r, i (r.key)}
      <tr>
        <td><span class="key k{i}"></span> {LABEL[r.key]}</td>
        <td>{r.count}</td>
        <td>{r.tenants}</td>
        <td>{r.first} → {r.last}</td>
        <td><a href={links[r.key]}>{links[r.key].split('/').pop() ? `#${links[r.key].split('/').pop()}` : ''}</a></td>
      </tr>
    {/each}
  </tbody>
</table>

<figure>
  <svg viewBox="0 0 {months.length * 60} 136" role="img" aria-label="reports received per month, by problem">
    {#each months as m, i (m.month)}
      {@const stack = m.counts.map((n, k) => ({ n, k, below: m.counts.slice(0, k).reduce((a, b) => a + b, 0) }))}
      {#each stack as s (s.k)}
        <rect class="k{s.k}" x={i * 60 + 10} y={110 - h(s.below + s.n)} width="40" height={h(s.n)} />
      {/each}
      <text x={i * 60 + 30} y="126">{m.month}</text>
    {/each}
  </svg>
  <figcaption>Reports received per month, by problem. Colors match the table.</figcaption>
</figure>

<style>
  .sum { font-size: 1.1rem; }
  svg { width: 100%; max-width: 640px; }
  text { font: 8px var(--mono); fill: var(--muted); text-anchor: middle; }
  figcaption { color: var(--muted); font-size: 0.85rem; }
  .key { display: inline-block; width: 0.7em; height: 0.7em; border-radius: 2px; }
  .k0 { fill: var(--accent); background: var(--accent); }
  .k1 { fill: var(--warn); background: var(--warn); }
  .k2 { fill: var(--muted); background: var(--muted); }
</style>
