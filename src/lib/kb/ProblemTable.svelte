<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import tickets from '#lib/data/tickets-4471.json';
  import { uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';
  import { groupByProblem } from '#lib/kb/problems';

  let { ticket }: { ticket: number } = $props();

  const WHAT: Record<string, string> = {
    parser: 'the shared parser reads day-first dates month first',
    hint: 'the booking-v3 form hints MM/DD/YYYY, then reads the date day first',
    'one-day': 'the booking moves by one day, not by a day/month swap',
    unplaced: 'no tenant on file',
  };

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByProblem(merged, tenantOf, notes.oneDayMove));
  type Ticket = { url: string; state: string; by: string };
  const ticketOf = (p: string): Ticket | undefined => (tickets as Record<string, Ticket>)[p];
  const isOpen = (p: string) => (ticketOf(p)?.state ?? 'open') === 'open';
  const stillOpen = $derived(groups.filter((g) => isOpen(g.problem)));
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} ·
  <strong>{groups.length}</strong>
  {groups.length === 1 ? 'problem' : 'problems'} ·
  <strong class:open={stillOpen.length}>{stillOpen.reduce((n, g) => n + g.reports.length, 0)}</strong>
  still open{stillOpen.length ? ` (${stillOpen.map((g) => g.problem).join(', ')})` : ''}
</p>

<table>
  <thead>
    <tr><th>problem</th><th>what the reports show</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th><th>state</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.problem)}
      {@const t = ticketOf(g.problem)}
      <tr class:open={isOpen(g.problem)}>
        <td><code>{g.problem}</code></td>
        <td>{WHAT[g.problem]}</td>
        <td>{g.reports.length}</td>
        <td>{g.tenants}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td>
          {#if t}
            <a href={t.url}>#{t.url.split('/').pop()}</a>
          {:else}
            <span class="none">no ticket</span>
          {/if}
        </td>
        <td>{t ? t.state : 'no ticket'}{#if t}<br /><span class="by">{t.by}</span>{/if}</td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
  .none { color: var(--warn); font-weight: 600; }
  .by { color: var(--muted); font-size: 0.85em; }
  .open td { background: var(--warn-bg); }
  .open td:last-child, strong.open { color: var(--warn); font-weight: 600; }
</style>
