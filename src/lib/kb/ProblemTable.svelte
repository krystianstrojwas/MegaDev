<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { causeOf, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  // One id, one report. The desk's one-day-move reads (desk-notes.json) are a problem of their own.
  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const oneDay = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)));
  const rest = $derived(merged.filter((r) => !notes.oneDayMove.includes(r.id)));
  const by = (c: string) => rest.filter((r) => causeOf(r, tenantOf(r)) === c);

  const rows = $derived([
    { problem: 'Parser reads day-first dates month-first', reports: by('parser') },
    { problem: 'booking-v3 form hints MM/DD/YYYY to day-first tenants', reports: by('hint') },
    { problem: 'Invites land one day off (likely time zone)', reports: oneDay },
  ]);
  const tenants = (rs: Report[]) => new Set(rs.map((r) => r.tenant)).size;
</script>

<p><strong>{merged.length}</strong> reports merged into #{ticket} · <strong>{rows.length}</strong> problems</p>

<table>
  <thead><tr><th>problem</th><th>reports</th><th>tenants</th></tr></thead>
  <tbody>
    {#each rows as row (row.problem)}
      <tr><td>{row.problem}</td><td>{row.reports.length}</td><td>{tenants(row.reports)}</td></tr>
    {/each}
  </tbody>
</table>
