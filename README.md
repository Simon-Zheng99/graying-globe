# The Graying Globe: A World of Steady Population Shift

**Live demo: [simon-zheng99.github.io/graying-globe](https://simon-zheng99.github.io/graying-globe/)**

An interactive data visualization of how the world's population structure changes from 1960 to 2050, built with Vue 3, TypeScript, and D3.js.

<!-- Optional: add a screenshot. Save one as docs/screenshot.png and uncomment the line below. -->
<!-- ![The Graying Globe screenshot](docs/screenshot.png) -->

## Overview

This project shows the shift in global population from the youth boom of the 1960s to the projected "Silver Tsunami" of 2050, when people aged 65 and older become the fastest-growing age group on Earth. It follows a Martini Glass storytelling structure: a guided narrative first, then a free-form explorer where you can dig into any country and year.

## Tech Stack

- **Vue 3** with **TypeScript** (type-checked with vue-tsc)
- **D3.js** and **TopoJSON** for the charts and world map
- **Pinia** for shared state, so all views stay in sync
- **Vuetify** for UI components
- **Vite** for development and builds
- **GitHub Actions** for CI/CD, building and deploying to **GitHub Pages** on every push to `main`

## Features

### Scrollytelling Storyboard

The experience opens with a scrollable narrative built around three milestones:

- **1960: The Youth Boom.** High birth rates and a classic wide-base pyramid.
- **2017: Peak Child.** The point where the number of young children stopped growing.
- **2050: The Silver Tsunami.** The projected aging of the global population.

As you scroll, the population pyramid updates to match each era.

### Interactive Population Pyramid

- Mirrored bar chart with male (left) and female (right) population by age group
- Shown as a percentage of total population, so countries of different sizes can be compared
- Ghost bars show the distribution from 10 years earlier
- Fixed x-axis scale (-10% to +10%)

### Year Slider and Play Button

- Drag the slider to any year from 1960 to 2050
- Press **Play** to animate through all 90 years, and **Pause** to stop at any point

### Country Dropdown

- Choose from 235 countries and economies
- The pyramid and line graph update instantly
- Countries with no available data are left out of the list automatically

### Line Graph

- Total population from 1960 to 2050
- Solid line for historical data (1960–2020), dashed line for projections (2020–2050)
- Annotated milestones: Youth Boom (1960), Peak Child (2017), and the projected 9.7B peak (2050)
- Click anywhere on the graph to jump the pyramid to that year

### World Map

- Colors each country by whether its 0–4 age group is increasing (white) or decreasing (black) in the selected year
- Stays in sync with the year slider

## Data

Population data comes from [DATA SOURCE NAME](DATA SOURCE LINK). Values from 2020 onward are projections. The processed data files are in `public/data/`.

## Running Locally

Requires [Node.js](https://nodejs.org/) 20 or later.

```bash
git clone https://github.com/Simon-Zheng99/graying-globe.git
cd graying-globe
npm install
npm run dev
```

Then open `http://localhost:3000/graying-globe/` in your browser.

To build and preview the production version:

```bash
npm run build
npm run preview
```

## Deployment

Every push to `main` runs the GitHub Actions workflow in `.github/workflows/deploy.yml`. The workflow installs dependencies, type-checks and builds the app, and deploys the `dist` folder to GitHub Pages.