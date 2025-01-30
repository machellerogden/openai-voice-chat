<script>
  let {
    isSessionActive,
    sendClientEvent,
    events,
  } = $props();

  let functionAdded = $state(false);
  let functionCallOutput = $state(null);
  let { theme, colors } = $derived(JSON.parse(functionCallOutput?.arguments ?? '{}'));

  const functionDescription = `
  Call this function when a user asks for a color palette.
  `;

  const sessionUpdate = {
    type: "session.update",
    session: {
      tools: [
        {
          type: "function",
          name: "display_color_palette",
          description: functionDescription,
          parameters: {
            type: "object",
            strict: true,
            properties: {
              theme: {
                type: "string",
                description: "Description of the theme for the color scheme.",
              },
              colors: {
                type: "array",
                description: "Array of five hex color codes based on the theme.",
                items: {
                  type: "string",
                  description: "Hex color code",
                },
              },
            },
            required: ["theme", "colors"],
          },
        },
      ],
      tool_choice: "auto",
    },
  };

  $effect(() => {
    if (!events || events.length === 0) return;

    const firstEvent = events.at(-1);
    if (!functionAdded && firstEvent.event.type === "session.created") {
      console.log("Adding function to session...", sessionUpdate);
      sendClientEvent(sessionUpdate);
      functionAdded = true;
    }

    const [ mostRecentEvent ] = events;
    if (
      mostRecentEvent.event.type === "response.done" &&
      mostRecentEvent.event.response.output
    ) {
      mostRecentEvent.event.response.output.forEach((output) => {
        if (
          output.type === "function_call" &&
          output.name === "display_color_palette"
        ) {
          functionCallOutput = output;
          setTimeout(() => {
            sendClientEvent({
              type: "response.create",
              response: {
                instructions: `
                ask for feedback about the color palette - don't repeat
                the colors, just ask if they like the colors.
              `,
              },
            });
          }, 500);
        }
      });
    }
  });

  $effect(() => {
    if (!isSessionActive) {
      functionAdded = false;
      functionCallOutput = null;
    }
  });
</script>

<section class="h-full w-full flex flex-col gap-4">
  <div class="h-full bg-gray-50 p-4 text-black">
    <h2 class="text-lg font-bold">Color Palette Tool</h2>
    {#if isSessionActive}
      {#if functionCallOutput}
        <div class="flex flex-col gap-2">
          <p>Theme: {theme}</p>
          {#each colors as color}
            <div
              key={color}
              class="w-full h-16 rounded-md flex items-center justify-center border border-gray-200"
              style={`background-color: ${color}`}
            >
              <p class="text-sm font-bold text-black bg-slate-100 rounded-md p-2 border border-black">
                {color}
              </p>
            </div>
          {/each}
          <pre class="text-xs bg-gray-100 rounded-md p-2 overflow-x-auto">
{JSON.stringify(functionCallOutput, null, 2)}
          </pre>
        </div>
      {:else}
        <p>Ask for advice on a color palette...</p>
      {/if}
    {:else}
      <p>Start the session to use this tool...</p>
    {/if}
  </div>
</section>
