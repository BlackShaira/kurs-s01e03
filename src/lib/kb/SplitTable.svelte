<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, ticketCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // One row per problem the reports describe, counted the way this kb counts.
  //
  // The two cause rows come from `groupByCause`, so they are the desk's own view of its
  // own export. The third row is not a cause: `causes.ts` separates reports that were
  // "merged into the wrong place" from reports the desk "read differently", and `readAs`
  // counts the second kind where they are filed and footnotes them there. So the third
  // row is a subset of the row above it, and the two cause rows are what add up.
  //
  // The last column is the kb's own verdict, printed by the same rule `CauseTable` uses:
  // the cause this ticket is, or that this cause has no ticket of its own.
  let { ticket }: { ticket: number } = $props();

  const all = $derived(uniqueById(reports as Report[]));
  const merged = $derived(all.filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));
  const owner = $derived(ticketCause(groups, ticket));
  const counted = $derived(groups.reduce((n, g) => n + g.reports.length, 0));
  const oneDay = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)));
  const oneDayUnder = $derived(
    groups.filter((g) => readAs(g, notes.oneDayMove) > 0).map((g) => g.cause).join(', '),
  );

  const WHAT: Record<string, string> = {
    parser: 'the shared parser reads every date month first',
    hint: "booking-v3's date hint says MM/DD/YYYY and the form reads it the tenant's way",
  };
  const channelsIn = (rs: Report[]) =>
    Object.entries(rs.reduce<Record<string, number>>((n, r) => ({ ...n, [r.channel]: (n[r.channel] ?? 0) + 1 }), {}))
      .sort((a, b) => b[1] - a[1])
      .map(([c, n]) => `${c} ×${n}`)
      .join(' · ');
  const seen = (rs: Report[]) => {
    const days = rs.map((r) => r.receivedAt.slice(0, 10)).sort();
    return `${days[0]} → ${days[days.length - 1]}`;
  };
</script>

<p class="sum">
  <strong>{counted}</strong> reports merged into #{ticket} ·
  <strong>{groups.length}</strong> {groups.length === 1 ? 'cause' : 'causes'} ·
  {counted === merged.length ? 'every report counted once' : `${merged.length} in the ticket — this does not add up`}
</p>

<table>
  <thead>
    <tr><th>cause</th><th>reports</th><th>ticket</th><th>what it is</th><th>channels</th><th>tenants</th><th>seen</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.cause)}
      <tr>
        <td><code>{g.cause}</code></td>
        <td class="n">{g.reports.length}</td>
        <td>{g.cause === owner ? `#${ticket}` : 'no ticket of its own on the kb'}</td>
        <td>{WHAT[g.cause] ?? 'no cause the rules can name'}</td>
        <td class="muted">{channelsIn(g.reports)}</td>
        <td class="n">{g.tenants}</td>
        <td class="muted">{seen(g.reports)}</td>
      </tr>
    {/each}
    {#if oneDay.length}
      <tr class="subset">
        <td><code>one-day</code></td>
        <td class="n">{oneDay.length}</td>
        <td>counted under <code>{oneDayUnder}</code></td>
        <td>the date moves by one day, which is no swap at all</td>
        <td class="muted">{channelsIn(oneDay)}</td>
        <td class="n">{new Set(oneDay.map((r) => r.tenant)).size}</td>
        <td class="muted">{seen(oneDay)}</td>
      </tr>
    {/if}
  </tbody>
</table>

<p class="note">
  The <code>one-day</code> row is a subset of the row it sits under, not a share of its own, so
  the cause rows are what add up. Those reports are in the right ticket and read the wrong way:
  <code>readAs</code> counts them where they are filed and footnotes them there, and the desk
  keeps the list in <code>desk-notes.json</code>.
</p>

<p class="note">
  Intake's own tag disagrees with the cause the report belongs to in
  <strong>{groups.reduce((n, g) => n + g.reports.filter((r) => r.meta.intakeTag !== g.cause).length, 0)}</strong>
  of these {counted} reports, because intake guesses from the reporter's browser and the cause
  depends on the tenant. Counting by <code>meta.intakeTag</code> gives the wrong number for every
  row above.
</p>

<p class="note">
  The export ships {(reports as Report[]).length} report objects for {all.length} reports: its
  pages overlap, so some arrive twice. Every count here is after <code>uniqueById</code>.
</p>

<style>
  .sum { font-size: 1.1rem; }
  .n { text-align: right; font-variant-numeric: tabular-nums; }
  .muted { color: var(--muted); font-size: 0.85em; }
  .note { color: var(--muted); font-size: 0.85em; }
  .subset td { background: var(--note-bg); }
</style>
