<script>
  let { event, timestamp } = $props();
  let isExpanded = $state(false);
  let isClient = $derived(event?.event_id && !event.event_id.startsWith('event_'));
</script>

<div class="flex flex-col gap-2 p-2 rounded-md bg-gray-50">
    <div
        class="flex items-center gap-2 cursor-pointer"
        onclick={() => isExpanded = !isExpanded}
    >
    {#if isClient}&darr;{:else}&uarr;{/if}
    <div class="text-sm text-gray-500">
        {isClient ? 'client:' : 'server:'}
        &nbsp;{event.type} | {timestamp}
    </div>
    </div>
    <div
    class={`text-gray-500 bg-gray-200 p-2 rounded-md overflow-x-auto ${
        isExpanded ? 'block' : 'hidden'
    }`}
    >
        <pre class="text-xs">{JSON.stringify(event, null, 2)}</pre>
    </div>
</div>
