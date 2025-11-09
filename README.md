# T-Mobile Customer Happiness Index Dashboard

Real-time, AI-assisted monitoring platform that forecasts T-Mobile customer sentiment, correlates network experience with emotional impact, and detects viral complaints before they explode. This guide explains the end-to-end build so Cursor (or any contributor) can spin up the full system quickly.

---

## 1. System Overview

- **Frontend:** React 18 + TypeScript, Tailwind CSS, Recharts visualisations, Lucide React icons, Socket.IO client for live updates.
- **Backend:** Node.js + Express (TypeScript), REST APIs, Socket.IO server, PostgreSQL (history), Redis (cache), optional message queue, Python ML microservice (optional) or JS-based ML utilities.
- **AI/ML Engines:**
  - Predictive happiness forecasting (weighted moving average + contextual adjustments).
  - Network-experience correlation engine (Pearson correlation + lag analysis).
  - Viral sentiment detection (velocity, influence, cross-platform spread, etc.).
- **Data Simulation:** Mock feeds for social media, network metrics, support tickets, app reviews, and historical scores.

---

## 2. Repository Layout

```
tmobile-happiness-index/
├── frontend/               # React + Tailwind dashboard
├── backend/                # Express + Socket.IO API server
├── shared/                 # Cross-cutting TypeScript types
└── README.md               # You are here
```

---

## 3. Phase-by-Phase Build Instructions

### Phase 1 — Project Bootstrap (≈15 min)
```bash
mkdir tmobile-happiness-index && cd tmobile-happiness-index

# Frontend
npm create vite@latest frontend -- --template react-ts
cd frontend
npm install
npm install recharts lucide-react socket.io-client
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p

# Backend
cd ..
mkdir backend && cd backend
npm init -y
npm install express cors socket.io
npm install -D typescript @types/node @types/express @types/cors ts-node nodemon
npx tsc --init
```

### Phase 2 — Mock Data Layer (≈30 min)
- Create `backend/src/utils/mockDataGenerator.ts`.
- Generate realistic:
  - Hourly happiness scores for 7 days.
  - Network metrics aligned with scores.
  - ≥1000 social mentions (positive/negative/mixed).
  - ≥50 correlation patterns with metadata.
- Include time-of-day/week/event/weather modifiers, randomness, and viral scenario simulators.
- Expose helper methods for scenario triggers (outage, billing issue, etc.).

### Phase 3 — Prediction Engine (≈45 min)
- File: `backend/src/services/predictionService.ts`.
- Implement algorithm:
  1. Weighted moving average baseline.
  2. Time-based adjustments (hour/day/weekend/holiday).
  3. Network impact factors.
  4. Event impact calculations.
  5. Confidence interval from historical variance.
- Return structured prediction results (score, confidence, contributing factors, alerts).
- API: `GET /api/predictions/forecast` with horizons `2h`, `6h`, `12h`, `24h`.

### Phase 4 — Correlation Engine (≈45 min)
- Files:
  - `backend/src/utils/statistics.ts` — Pearson correlation, lag analysis, strength labelling, matrix helpers.
  - `backend/src/services/correlationService.ts` — orchestrates insights, thresholds, fix recommendations.
- APIs:
  - `GET /api/correlations/top`
  - `GET /api/correlations/metric/:metricName`
  - `GET /api/correlations/heatmap`
  - `GET /api/correlations/insights`

### Phase 5 — Viral Detection Engine (≈45 min)
- File: `backend/src/services/viralService.ts`.
- Core pieces:
  - Mention velocity computation (15/30/60 min windows).
  - Influencer weighting (followers, verification, engagement).
  - Cross-platform spread scoring.
  - Emotional intensity and historical pattern match.
  - Risk level determination (`monitor`, `prepare`, `activate`, `critical`).
  - Stage detection (`spark`, `ignition`, `viral`, `containment`).
  - Alert generation with recommendations, templates, amplifier list.
- APIs:
  - `GET /api/viral/monitor`
  - `GET /api/viral/alerts`
  - `GET /api/viral/trends`
  - `GET /api/viral/issue/:issueId`
  - `POST /api/viral/simulate`

### Phase 6 — Real-Time Socket Layer (≈30 min)
- File: `backend/src/services/realtimeService.ts`.
- Initialise Socket.IO, emit `dashboard:update` payloads every 5 seconds.
- Aggregate latest happiness, predictions, correlations, viral alerts.
- Optionally emit specialised channels (`happiness:update`, `prediction:alert`, etc.).
- Ensure cleanup on shutdown (clear intervals).

### Phase 7 — Backend REST API (≈30 min)
- Routes under `backend/src/routes/`:
  - `happiness.ts`, `predictions.ts`, `correlations.ts`, `viral.ts`.
- Implement Express app in `backend/src/app.ts`:
  - Middleware: `cors`, `express.json()`, request logging.
  - Attach routes, error handlers, Socket.IO integration hook.
- `backend/src/server.ts`:
  - HTTP server on port `3001`.
  - Attach Socket.IO and boot `RealtimeService`.
  - Graceful shutdown handling.

### Phase 8 — Frontend Foundation (≈45 min)
- Tailwind config (`frontend/tailwind.config.{cjs,js}`):
  ```js
  export default {
    content: ['./index.html', './src/**/*.{js,ts,jsx,tsx}'],
    theme: {
      extend: {
        colors: {
          'tmobile-magenta': '#E20074',
          'tmobile-cyan': '#00A79D',
          'tmobile-yellow': '#FFB81C',
          'tmobile-blue': '#00558C',
          positive: '#10b981',
          neutral: '#FFB81C',
          negative: '#ef4444',
        },
      },
    },
    plugins: [],
  };
  ```
- Types: `frontend/src/types/index.ts` (mirror backend + shared enums/interfaces).
- API wrapper: `frontend/src/services/api.ts` (Axios/fetch with interceptors).
- Hooks: `useRealtime`, `usePredictions`, `useCorrelations`, `useViralAlerts`.
- Utility helpers (formatting, sentiment colouring, etc.).

### Phase 9 — Dashboard Components (≈2 h)
- Structure under `frontend/src/components/`.
- Key modules:
  - `Dashboard/Header.tsx`: hero band with score, trend, selectors, live indicator.
  - `Dashboard/AlertBanner.tsx`: consolidated alerts.
  - `Predictions/ForecastChart.tsx`, `PredictionCard.tsx`, `FactorBreakdown.tsx`: trend lines, confidence, factor bars, alerts list.
  - `Correlations/CorrelationHeatmap.tsx`, `TopDriversList.tsx`, `SankeyDiagram.tsx`.
  - `Viral/ViralMonitor.tsx`, `RiskGauge.tsx`, `AlertFeed.tsx`, `ResponseCenter.tsx`.
  - Shared charts: `Charts/LineChart.tsx`, `BarChart.tsx`, `PieChart.tsx` (Recharts wrappers).
- Aim for interactive, responsive layouts (3-column desktop, stack on mobile).

### Phase 10 — Dashboard Integration (≈45 min)
- Implement `Dashboard.tsx` layout grid:
  - Header, optional alert banner, main grid (charts, correlation list, viral panel), secondary grid (sentiment distribution, channel mix, category radar, highlights).
- Wire hooks to populate state, handle loading & error states.
- Add `App.tsx` wrappers (routing optional) and mount in `main.tsx`.

### Phase 11 — Testing & Polish (≈60 min)
- Validate each service with mock data snapshots.
- WebSocket soak test (memory/leak checks).
- UI responsiveness across breakpoints.
- Add loading skeletons, error boundaries, fallback UI for empty datasets.
- Optimise performance:
  - Memoise expensive selectors.
  - Debounce rapid updates if needed.
  - Lazy load heavy visualisations.

### Phase 12 — Demo Mode Enhancements (≈30 min)
- Floating `DemoControls` overlay to trigger:
  - Viral outage scenario.
  - Happiness drop.
  - Positive event boost.
- Data export buttons (CSV/JSON/PDF) for key panels.
- Keyboard shortcuts: `R` refresh, `P` toggle predictions, `V` toggle viral monitor, `Esc` close modals.

---

## 4. Environment Configuration

### Backend `.env`
```
PORT=3001
NODE_ENV=development
CORS_ORIGIN=http://localhost:5173
DATABASE_URL=postgresql://user:pass@localhost:5432/happiness_index
REDIS_URL=redis://localhost:6379
TWITTER_API_KEY=your_key_here
SENTIMENT_API_KEY=your_key_here
```

### Frontend `.env`
```
VITE_API_URL=http://localhost:3001
VITE_WS_URL=ws://localhost:3001
VITE_ENABLE_DEMO_MODE=true
```

Use `.env.local` for secrets during development; never commit real credentials.

---

## 5. Running the Stack

### Development
```bash
# Terminal 1
cd backend
npm run dev           # via nodemon/ts-node

# Terminal 2
cd frontend
npm run dev           # Vite dev server on http://localhost:5173
```

### Production Build
```bash
cd frontend && npm run build
cd ../backend && npm run build
npm start             # Serve compiled backend (ensure static hosting or deploy frontend separately)
```

Deploy options:
- Containerise with Docker Compose (one service per directory + optional Postgres/Redis).
- Vercel/Netlify for frontend, Render/Fly.io/Heroku for backend.
- Use Railway/Supabase for managed Postgres/Redis.

---

## 6. Implementation Priorities & Guidelines

- **Realistic Mock Data:** Mirror daily/weekly behaviour, include event/weather overlays, ensure correlated series.
- **Type Safety:** Share TypeScript interfaces via `shared/types.ts`; import from both frontend/back.
- **Interactive Charts:** Tooltips, click-to-drill, range selectors, responsive breakpoints.
- **Real-Time Experience:** Socket-driven updates every 5 seconds, visual pulse indicators, smooth transitions.
- **Responsiveness & Accessibility:** Mobile-first layout, semantic HTML, ARIA labels, keyboard navigation paths.
- **Demo Mode:** Predefined scenarios, toggle controls, emphasise storytelling for presentations.
- **Logging & Monitoring:** Console logs in dev; structured logger in production; handle API/WS errors gracefully.
- **Comments & Docs:** Document ML/statistical logic, thresholds, and assumptions inline.
- **Performance:** Avoid unnecessary renders, cache heavy calculations, throttle WS updates if necessary.

---

## 7. Success Criteria

- Dashboard shows live happiness score with trend.
- Forecast panel surfaces 2h–24h predictions with confidence and factor breakdown.
- Correlation engine highlights top drivers, heatmaps, and actionable insights.
- Viral detection monitors social chatter, classifies risk levels, and provides playbooks.
- WebSocket pushes fresh data every 5 seconds without errors or leaks.
- Charts feel polished, interactive, and responsive with T-Mobile branding.
- Demo controls trigger simulated scenarios convincingly.
- No console errors/warnings; Lighthouse/Performance checks pass reasonable thresholds.

---

## 8. Optional Enhancements

- Dark mode toggle.
- Automated PDF reporting and CSV exports.
- Email/SMS alerting for `critical` viral events.
- Historical playback scrubber.
- Control panel for tuning ML/threshold parameters.
- Twitter API integration for live data (replace mock generator).
- Localization & accessibility upgrades.
- Performance observability dashboard (metrics + tracing).

---

Happy building! This README should be sufficient for Cursor AI (or any engineer) to implement the full T-Mobile Customer Happiness Index experience end-to-end. For questions or improvements, open an issue or start a discussion in the repository.
