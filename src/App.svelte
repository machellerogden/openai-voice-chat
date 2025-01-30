<script>
  import './app.css';

  import EventLog from './components/EventLog.svelte';
  import SessionControls from './components/SessionControls.svelte';
  import ToolPanel from './components/ToolPanel.svelte';

  let isSessionActive = $state(false);
  let events = $state([]);
  let dataChannel = $state(null);
  let peerConnection = $state({});
  let audioElement = $state({});

  async function startSession() {
    // Get an ephemeral key from backend
    const tokenResponse = await fetch('/token');
    const data = await tokenResponse.json();
    const EPHEMERAL_KEY = data.client_secret.value;

    // Create a peer connection
    const pc = new RTCPeerConnection();

    // Set up to play remote audio from the model
    audioElement.current = document.createElement('audio');
    audioElement.current.autoplay = true;
    pc.ontrack = (e) => (audioElement.current.srcObject = e.streams[0]);

    // Add local audio track for microphone input in the browser
    const ms = await navigator.mediaDevices.getUserMedia({
      audio: true,
    });
    pc.addTrack(ms.getTracks()[0]);

    // Set up data channel for sending and receiving events
    const dc = pc.createDataChannel('oai-events');
    dataChannel = dc;

    // Start the session using the Session Description Protocol (SDP)
    const offer = await pc.createOffer();
    await pc.setLocalDescription(offer);

    const baseUrl = 'https://api.openai.com/v1/realtime';
    const model = 'gpt-4o-realtime-preview-2024-12-17';
    const sdpResponse = await fetch(`${baseUrl}?model=${model}`, {
      method: 'POST',
      body: offer.sdp,
      headers: {
        Authorization: `Bearer ${EPHEMERAL_KEY}`,
        'Content-Type': 'application/sdp',
      },
    });

    const answer = {
      type: 'answer',
      sdp: await sdpResponse.text(),
    };

    await pc.setRemoteDescription(answer);

    peerConnection.current = pc;
  }

  function stopSession() {
    if (dataChannel) {
      dataChannel.close();
    }
    if (peerConnection.current) {
      peerConnection.current.close();
    }

    isSessionActive = false;
    dataChannel = null;
    peerConnection.current = null;
  }

function sendClientEvent(message) {
  if (dataChannel) {
    message.event_id = message.event_id ?? crypto.randomUUID();
    dataChannel.send(JSON.stringify(message));
    events = [
      { event: message, timestamp: new Date().toISOString() },
      ...events,
    ];
  } else {
    console.error('No dataChannel available, cannot send event.');
  }
}

  // 4) Send a text message
  function sendTextMessage(text) {
    const event = {
      type: 'conversation.item.create',
      item: {
        type: 'message',
        role: 'user',
        content: [
          {
            type: 'input_text',
            text,
          },
        ],
      },
    };
    sendClientEvent(event);
    sendClientEvent({ type: 'response.create' });
  }

  function handleDataChannelMessage(e) {
      const serverEvent = JSON.parse(e.data);
      events = [
        { event: serverEvent, timestamp: new Date().toISOString() },
        ...events,
      ];
  }

  function handleDataChannelOpen() {
      isSessionActive = true;
      events = [];
  }

  $effect(() => {
    dataChannel?.addEventListener('message', handleDataChannelMessage);
    dataChannel?.addEventListener('open', handleDataChannelOpen);
    return () => {
      dataChannel?.removeEventListener('message', handleDataChannelMessage);
      dataChannel?.removeEventListener('open', handleDataChannelOpen);
    };
  });
</script>

<main class="absolute top-0 left-0 right-0 bottom-0">
  <section class="absolute top-0 left-0 right-[400px] bottom-0 flex">
    <section class="absolute top-0 left-0 right-0 bottom-32 px-4 overflow-y-auto w-full">
        <div class="flex gap-4 py-1 px-4 items-center border-b border-gray w-full">
          <img
            class="inline-block w-[24px] h-[24px]"
            src="/logo.svg"
            alt="Gateless"
          />
          <h1 class="inline-block text-2xl">Voice Chat</h1>
        </div>
        <EventLog {events} />
    </section>
    <section class="absolute h-32 left-0 right-0 bottom-0 p-4">
      <SessionControls
        {startSession}
        {stopSession}
        {sendClientEvent}
        {sendTextMessage}
        {events}
        {isSessionActive}
      />
    </section>
  </section>
  <section class="absolute top-0 w-[400px] right-0 bottom-0 p-4 pt-0 overflow-y-auto">
      <ToolPanel
        {sendClientEvent}
        {sendTextMessage}
        {events}
        {isSessionActive}
      />
  </section>
</main>
