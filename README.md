# OpenAI Realtime Console

This is an example application showing how to use the [OpenAI Realtime API](https://platform.openai.com/docs/guides/realtime) with [WebRTC](https://platform.openai.com/docs/guides/realtime-webrtc).

## Installation and usage

Before you begin, you'll need an OpenAI API key - [create one in the dashboard here](https://platform.openai.com/settings/api-keys). Create a `.env` file from the example file and set your API key in there:

```bash
cp .env.example .env
```

Running this application locally requires [Node.js](https://nodejs.org/) to be installed. Install dependencies for the application with:

```bash
npm install
```

Start the backend server with:

```bash
node server.js
```

Host the web app for development with:

```bash
npm run dev
```

This should start the console application on [http://localhost:5173](http://localhost:5173).

## Credits

This is a port of [https://github.com/openai/openai-realtime-console](https://github.com/openai/openai-realtime-console) into a basic Svelte app.

## License

MIT
