<script>
    let { children, onclick } = $props();
</script>

<button
    class="bg-gray-800 text-white rounded-full p-4 flex items-center gap-1 hover:opacity-90"
    onclick={onclick}
>
    {@render children?.()}
</button>
