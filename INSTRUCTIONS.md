# Calendar App — Build Instructions

## Overview
A web-based calendar app with Material 3 design, pastel colour scheme, dark/light mode, and iOS compatibility.

---

## Tech Stack

- **Framework:** React (Vite)
- **UI Library:** Material UI v6 (MUI) — Material 3 support via `@mui/material-next` or custom theming
- **Styling:** Emotion (included with MUI)
- **Date handling:** `date-fns` or `dayjs`
- **PWA (iOS compatibility):** Vite PWA plugin (`vite-plugin-pwa`)

---

## Setup

```bash
npm create vite@latest calendar-app -- --template react-ts
cd calendar-app
npm install @mui/material @mui/icons-material @emotion/react @emotion/styled
npm install dayjs
npm install -D vite-plugin-pwa
```

---

## Material 3 Theming with Pastel Colours

Create `src/theme.ts`:

```ts
import { createTheme } from '@mui/material/styles';

const pastelTokens = {
  primary: '#A8C5DA',       // pastel blue
  secondary: '#C3B1E1',     // pastel purple
  error: '#F4A9A8',         // pastel red
  background: '#F9F6FF',    // soft lavender white
  surface: '#FFFFFF',
};

const darkPastelTokens = {
  primary: '#7BA7BC',
  secondary: '#9D8EC4',
  error: '#D98080',
  background: '#1E1B2E',
  surface: '#2A2640',
};

export const getTheme = (mode: 'light' | 'dark') =>
  createTheme({
    palette: {
      mode,
      primary:    { main: mode === 'light' ? pastelTokens.primary    : darkPastelTokens.primary },
      secondary:  { main: mode === 'light' ? pastelTokens.secondary  : darkPastelTokens.secondary },
      error:      { main: mode === 'light' ? pastelTokens.error      : darkPastelTokens.error },
      background: {
        default: mode === 'light' ? pastelTokens.background : darkPastelTokens.background,
        paper:   mode === 'light' ? pastelTokens.surface    : darkPastelTokens.surface,
      },
    },
    shape: { borderRadius: 16 },   // rounded corners — M3 style
    typography: {
      fontFamily: '"Google Sans", "Roboto", sans-serif',
    },
  });
```

---

## Dark / Light Mode Toggle

In `src/App.tsx`:

```tsx
import React, { useState, useMemo } from 'react';
import { ThemeProvider, CssBaseline, IconButton } from '@mui/material';
import LightModeIcon from '@mui/icons-material/LightMode';
import DarkModeIcon from '@mui/icons-material/DarkMode';
import { getTheme } from './theme';
import Calendar from './components/Calendar';

export default function App() {
  const [mode, setMode] = useState<'light' | 'dark'>('light');
  const theme = useMemo(() => getTheme(mode), [mode]);

  return (
    <ThemeProvider theme={theme}>
      <CssBaseline />
      <IconButton onClick={() => setMode(m => m === 'light' ? 'dark' : 'light')}>
        {mode === 'light' ? <DarkModeIcon /> : <LightModeIcon />}
      </IconButton>
      <Calendar />
    </ThemeProvider>
  );
}
```

---

## Calendar Component

Create `src/components/Calendar.tsx`:

```tsx
import React, { useState } from 'react';
import { Box, Typography, Paper, Grid, IconButton } from '@mui/material';
import ChevronLeftIcon from '@mui/icons-material/ChevronLeft';
import ChevronRightIcon from '@mui/icons-material/ChevronRight';
import dayjs, { Dayjs } from 'dayjs';

export default function Calendar() {
  const [current, setCurrent] = useState<Dayjs>(dayjs());

  const startOfMonth = current.startOf('month');
  const daysInMonth  = current.daysInMonth();
  const startDay     = startOfMonth.day(); // 0 = Sunday

  const prev = () => setCurrent(c => c.subtract(1, 'month'));
  const next = () => setCurrent(c => c.add(1, 'month'));

  const cells = Array.from({ length: startDay + daysInMonth }, (_, i) =>
    i < startDay ? null : i - startDay + 1
  );

  return (
    <Paper elevation={0} sx={{ p: 3, maxWidth: 420, mx: 'auto', mt: 4, borderRadius: 4 }}>
      {/* Header */}
      <Box display="flex" alignItems="center" justifyContent="space-between" mb={2}>
        <IconButton onClick={prev}><ChevronLeftIcon /></IconButton>
        <Typography variant="h6" fontWeight={600}>
          {current.format('MMMM YYYY')}
        </Typography>
        <IconButton onClick={next}><ChevronRightIcon /></IconButton>
      </Box>

      {/* Day labels */}
      <Grid container columns={7} mb={1}>
        {['Su','Mo','Tu','We','Th','Fr','Sa'].map(d => (
          <Grid key={d} size={1} textAlign="center">
            <Typography variant="caption" color="text.secondary">{d}</Typography>
          </Grid>
        ))}
      </Grid>

      {/* Day cells */}
      <Grid container columns={7} gap={0}>
        {cells.map((day, i) => (
          <Grid key={i} size={1} textAlign="center" py={0.5}>
            {day && (
              <Box
                sx={{
                  width: 36, height: 36,
                  display: 'flex', alignItems: 'center', justifyContent: 'center',
                  borderRadius: '50%', mx: 'auto', cursor: 'pointer',
                  bgcolor: day === dayjs().date() && current.isSame(dayjs(), 'month')
                    ? 'primary.main' : 'transparent',
                  '&:hover': { bgcolor: 'primary.light' },
                }}
              >
                <Typography variant="body2">{day}</Typography>
              </Box>
            )}
          </Grid>
        ))}
      </Grid>
    </Paper>
  );
}
```

---

## iOS / PWA Compatibility

### `vite.config.ts`

```ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { VitePWA } from 'vite-plugin-pwa';

export default defineConfig({
  plugins: [
    react(),
    VitePWA({
      registerType: 'autoUpdate',
      manifest: {
        name: 'Calendar App',
        short_name: 'Calendar',
        theme_color: '#A8C5DA',
        background_color: '#F9F6FF',
        display: 'standalone',
        icons: [
          { src: '/icon-192.png', sizes: '192x192', type: 'image/png' },
          { src: '/icon-512.png', sizes: '512x512', type: 'image/png' },
        ],
      },
    }),
  ],
});
```

### `index.html` — add inside `<head>`:

```html
<meta name="apple-mobile-web-app-capable" content="yes" />
<meta name="apple-mobile-web-app-status-bar-style" content="default" />
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
<link rel="apple-touch-icon" href="/icon-192.png" />
```

> Add `icon-192.png` and `icon-512.png` to `/public/`.

---

## Running the App

```bash
npm run dev       # development
npm run build     # production build
npm run preview   # preview production build
```

---

## Folder Structure

```
calendar-app/
├── public/
│   ├── icon-192.png
│   └── icon-512.png
├── src/
│   ├── components/
│   │   └── Calendar.tsx
│   ├── App.tsx
│   ├── main.tsx
│   └── theme.ts
├── index.html
├── vite.config.ts
└── package.json
```
