<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  // One id, one report. Counted where the desk files them (by tenant and channel), footnoted if read differently.
  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));
  const NAMES: Record<string, string> = {
    parser: 'Parser reads day-first dates month-first',
    hint: 'booking-v3 form hints MM/DD/YYYY to day-first tenants',
  };
</script>

<p><strong>{merged.length}</strong> reports merged into #{ticket} · <strong>{groups.length}</strong> problems</p>

<table>
  <thead><tr><th>problem</th><th>reports</th><th>tenants</th></tr></thead>
  <tbody>
    {#each groups as g (g.cause)}
      <tr><td>{NAMES[g.cause] ?? g.cause}</td><td>{g.reports.length}</td><td>{g.tenants}</td></tr>
    {/each}
  </tbody>
</table>

{#each groups as g (g.cause)}
  {@const moved = readAs(g, notes.oneDayMove)}
  {#if moved}
    <p class="note">{moved} reports read as a one-day move; filed under <code>{g.cause}</code> by tenant.</p>
  {/if}
{/each}

<style>
  .note { color: var(--muted); font-size: 0.85em; }
</style>
