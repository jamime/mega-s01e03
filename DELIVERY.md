# S01E03 delivery

The support desk's kb export, plus one added page about ticket #4471.

- The page source: `src/routes/kb/incidents/4471-split/+page.svx`
- The page as the desk will see it: `rendered/kb/incidents/4471-split.html`, picture at `rendered/4471-split.png`
- The component that computes its counts: `src/lib/kb/TicketSplit.svelte`
- The tickets: issues #1, #2 and #3 in this repo

The export ships `src/` only. `package.json`, `svelte.config.js`, `vite.config.js`, `src/app.html` and
`src/routes/+layout.js` are the smallest setup that builds it. To rebuild: `npm install`, then `npm run build`.
