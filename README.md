# The Graying Globe: A World of Steady Population Shift

An interactive data visualization exploring global population demographic shifts from 1960 to 2050, built with Vue 3 and D3.js.

## Overview

This project visualizes the transformation of global population structure — from the youth boom of the 1960s to the projected "Silver Tsunami" of 2050, where people aged 65 and older become the fastest-growing group on Earth. The visualization uses a Martini Glass storytelling structure, beginning with a guided narrative and opening into a free-form interactive explorer.

---

## Features

### Scrollytelling Storyboard
The experience begins with a scrollable narrative that presents three key demographic milestones:
- **1960: The Youth Boom** — high birth rates and a classic wide-base pyramid
- **2017: Peak Child** — the point where the number of young children stopped growing
- **2050: The Silver Tsunami** — the projected aging of the global population

As you scroll, the population pyramid updates automatically to reflect each era.

### Interactive Population Pyramid
- Mirrored bar chart showing male (left) and female (right) population by age group
- Expressed as a percentage of total population for cross-country comparability
- **Ghost bars** show the population distribution from 10 years prior, making change immediately visible
- Fixed x-axis scale (-10% to +10%) ensures consistent comparisons across countries and years

### Year Slider and Play Button
- Drag the slider to explore any year from 1960 to 2050
- Press **Play** to animate the pyramid automatically through all 90 years
- Press **Pause** to stop at any point

### Country Dropdown
- Select any of 235 countries and economies with available data
- The pyramid and line graph update instantly on country change
- Countries with no available data are automatically excluded from the list

### Line Graph
- Shows total population from 1960 to 2050
- Solid line for historical data (1960–2020), dashed line for projections (2020–2050)
- Annotated milestones: Youth Boom (1960), Peak Child (2017), and 9.7B projected peak (2050)
- Click anywhere on the graph to jump the pyramid to that year

### World Map
- Color-coded map showing whether each country's 0–4 age group population is increasing (light gray) or decreasing (dark gray) in the selected year
- Updates in sync with the year slider