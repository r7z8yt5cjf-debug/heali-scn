/* SCN Desk service worker — bump CACHE version when shipping updates */
const CACHE = 'scn-desk-v2';
const SHELL = [
  './',
  './index.html',
  './manifest.webmanifest',
  './logo.png',
  './icon-192.png',
  './icon-512.png',
  './icon-maskable-512.png',
  './apple-touch-icon.png',
  './versions.json',
  './v9.json',
  './v8.json'
];

self.addEventListener('install', (e) => {
  e.waitUntil(caches.open(CACHE).then((c) => c.addAll(SHELL)).then(() => self.skipWaiting()));
});

self.addEventListener('activate', (e) => {
  e.waitUntil(
    caches.keys()
      .then((keys) => Promise.all(keys.filter((k) => k !== CACHE).map((k) => caches.delete(k))))
      .then(() => self.clients.claim())
  );
});

self.addEventListener('fetch', (e) => {
  const url = new URL(e.request.url);
  if (e.request.method !== 'GET' || url.origin !== location.origin) return;

  // Data files: serve cached immediately, refresh in background (stale-while-revalidate)
  if (url.pathname.includes('/')) {
    e.respondWith(
      caches.open(CACHE).then(async (c) => {
        const cached = await c.match(e.request);
        const refresh = fetch(e.request)
          .then((res) => { if (res.ok) c.put(e.request, res.clone()); return res; })
          .catch(() => cached);
        return cached || refresh;
      })
    );
    return;
  }

  // App shell: cache-first, network fallback; navigations fall back to index.html
  e.respondWith(
    caches.match(e.request).then((cached) =>
      cached ||
      fetch(e.request)
        .then((res) => {
          if (res.ok) caches.open(CACHE).then((c) => c.put(e.request, res.clone()));
          return res;
        })
        .catch(() => (e.request.mode === 'navigate' ? caches.match('./index.html') : undefined))
    )
  );
});
