# DataTrack

A progressive web application (PWA) built with SvelteKit for tracking and managing events with customizable attributes.

## Features

- Create and manage events with custom attributes
- Edit and delete existing events
- Auto-suggest event names and attributes based on previous entries
- Export event data to JSON
- Works offline as a PWA
- Mobile-friendly responsive design

## Tech Stack

- [SvelteKit](https://kit.svelte.dev/) - Framework
- [Tailwind CSS](https://tailwindcss.com/) - Styling
- [Vite](https://vitejs.dev/) - Build tool
- [PWA Support](https://github.com/vite-pwa/sveltekit) - For offline capabilities

## Developing

Once you've cloned the project and installed dependencies with `npm install`, start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

## Building

To create a production version of your app:

```bash
npm run build
```

You can preview the production build with `npm run preview`.

## Data Storage

All event data is stored locally in the browser's localStorage. No data is sent to external servers.
