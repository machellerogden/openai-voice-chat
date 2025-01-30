<script>
  import Event from './Event.svelte';

  let { events } = $props();
  let deltaEvents = $state({});
  let eventsToDisplay = $derived(events?.reduce((acc, { event, timestamp }) => {
    if (event.type.endsWith('delta')) {
      if (deltaEvents[event.type]) return acc;
      acc[event.type] = { event, timestamp };
    } else {
      acc.push({ event, timestamp });
    }
    return acc;
  }, []));
</script>

<div class="w-full">
  {#each eventsToDisplay as { event, timestamp } (event.event_id)}
    <Event {event} {timestamp} />
  {/each}
</div>
