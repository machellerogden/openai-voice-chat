<script>
    import Button from "./Button.svelte";

    let {
        startSession,
        stopSession,
        sendClientEvent,
        sendTextMessage,
        serverEvents,
        isSessionActive
    } = $props();

    let isActivating = $state(false);

    function handleStartSession() {
        if (isActivating) return;
        isActivating = true;
        startSession();
    }

    function handleStopSession() {
        isActivating = false;
        stopSession();
    }

    let message = $state('');

    function handleSendClientEvent() {
        sendTextMessage(message);
        message = '';
    }
</script>

<div class="flex gap-4 border-t-2 border-gray-200 h-full rounded-md">
    {#if isSessionActive}
        <div class="flex items-center justify-center w-full h-full gap-4">
            <input
                onkeydown={(e) => {
                if (e.key === "Enter" && message.trim()) {
                    handleSendClientEvent();
                }
                }}
                type="text"
                placeholder="send a text message..."
                class="border border-gray-200 rounded-full p-4 flex-1"
                value={message}
                onchange={(e) => {
                    message = e.target.value;
                }}
            />
            <Button
                onclick={() => {
                if (message.trim()) {
                    handleSendClientEvent();
                }
                }}
                class="bg-blue-400"
            >
                send text
            </Button>
            <Button onclick={handleStopSession}>
                disconnect
            </Button>
        </div>
    {:else}
        <div class="flex items-center justify-center w-full h-full">
            <Button
                onclick={handleStartSession}
                class={isActivating ? "bg-gray-600" : "bg-red-600"}>
                {isActivating ? "starting session..." : "start session"}
            </Button>
        </div>
    {/if}
</div>
