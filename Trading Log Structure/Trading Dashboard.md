---
banner: "![[Banner.gif]]"
banner_y: 0
banner_x: 0.5
---
## Key Stats
```dataviewjs
// Get raw pages and handle potential non-array results
const rawPages = dv.pages('"Trades/Trade"') || [];
const allTrades = Array.isArray(rawPages)
  ? rawPages.where(p => !p.file.name.includes("Template"))
  : [];

// Dynamically extract available years
const yearsAvailable = allTrades.length > 0
  ? Array.from(new Set(allTrades
      .filter(p => p.exit_date)
      .map(p => {
        const date = moment.unix(Math.floor(p.exit_date / 1000));
        return date.isValid() ? date.year() : null;
      })
      .filter(year => year !== null)))
    .sort()
  : [];

// Dynamically extract available strategies with normalization
const strategiesAvailable = allTrades.length > 0
  ? Array.from(new Set(allTrades
      .filter(p => p.strategy && p.strategy.length > 0)
      .map(p => (p.strategy[0] || '').trim())))
    .sort()
  : [];

// Dynamically extract available directions
const directionsAvailable = allTrades.length > 0
  ? Array.from(new Set(allTrades
      .filter(p => p.Direction && p.Direction.length > 0)
      .map(p => (p.Direction[0] || '').trim())))
    .sort()
  : [];

const container = this.container;
container.innerHTML = '';

const mainDiv = container.createEl('div');

// UPDATED CSS for dropdownDiv with responsive styling and extra space between dropdown groups
const dropdownDiv = mainDiv.createEl('div', {
  attr: {
    id: 'dropdownDiv',
    style: `
      margin-bottom: 20px;
      display: flex;
      flex-wrap: wrap;
      gap: 15px;
      align-items: center;
    `
  }
});

// Add responsive CSS for mobile stacking and dark theme buttons
const style = document.createElement('style');
style.textContent = `
  @media (max-width: 600px) {
    #dropdownDiv > div {
      flex-basis: 100% !important;
      display: flex !important;
      justify-content: flex-start !important;
      gap: 5px !important;
      margin-bottom: 10px;
    }
  }
  .button-group button {
    padding: 5px 10px;
    margin-right: 5px;
    cursor: pointer;
    background-color: #000;
    border: 1px solid #333;
    border-radius: 3px;
    color: #fff;
    transition: background-color 0.3s;
  }
  .button-group button:hover {
    background-color: #222;
  }
  .button-group button.active {
    background-color: #007bff;
    color: #fff;
  }
`;
document.head.appendChild(style);

// Year dropdown group
const yearGroup = dropdownDiv.createEl('div', { attr: { style: 'display: flex; align-items: center; gap: 5px;' } });
yearGroup.createEl('label', { text: 'Year:', attr: { for: 'yearSelect' } });
const yearSelect = yearGroup.createEl('select', { attr: { id: 'yearSelect', style: 'margin-right: 10px;' } });
yearSelect.createEl('option', { value: 'all', text: 'All Years' });
yearsAvailable.forEach(year => yearSelect.createEl('option', { value: year, text: year }));
yearSelect.value = 'all';

// Month dropdown group
const monthGroup = dropdownDiv.createEl('div', { attr: { style: 'display: flex; align-items: center; gap: 5px;' } });
monthGroup.createEl('label', { text: 'Month:', attr: { for: 'monthSelect' } });
const monthSelect = monthGroup.createEl('select', { attr: { id: 'monthSelect' } });
monthSelect.createEl('option', { value: 'all', text: 'All Months' });
Array.from({ length: 12 }, (_, i) =>
  monthSelect.createEl('option', { value: i + 1, text: moment().month(i).format("MMMM") })
);
monthSelect.createEl('option', { value: 'Q1', text: 'Q1' });
monthSelect.createEl('option', { value: 'Q2', text: 'Q2' });
monthSelect.createEl('option', { value: 'Q3', text: 'Q3' });
monthSelect.createEl('option', { value: 'Q4', text: 'Q4' });

// Days of the week dropdown group
const dayGroup = dropdownDiv.createEl('div', { attr: { style: 'display: flex; align-items: center; gap: 5px;' } });
dayGroup.createEl('label', { text: 'Day:', attr: { for: 'daySelect' } });
const daySelect = dayGroup.createEl('select', { attr: { id: 'daySelect', style: 'margin-right: 10px;' } });
daySelect.createEl('option', { value: 'all', text: 'All Days' });
['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday', 'Weekdays', 'Weekends'].forEach(day =>
  daySelect.createEl('option', { value: day.toLowerCase(), text: day })
);
daySelect.value = 'all';

// Date range group
const dateRangeGroup = dropdownDiv.createEl('div', { attr: { style: 'display: flex; align-items: center; gap: 5px;' } });
dateRangeGroup.createEl('label', { text: 'Start Date:', attr: { for: 'startDate' } });
const startDateInput = dateRangeGroup.createEl('input', { attr: { type: 'date', id: 'startDate' } });
const endDateGroup = dropdownDiv.createEl('div', { attr: { style: 'display: flex; align-items: center; gap: 5px;' } });
endDateGroup.createEl('label', { text: 'End Date:', attr: { for: 'endDate' } });
const endDateInput = endDateGroup.createEl('input', { attr: { type: 'date', id: 'endDate' } });

// System dropdown group
const systemGroup = dropdownDiv.createEl('div', { attr: { style: 'display: flex; align-items: center; gap: 5px;' } });
systemGroup.createEl('label', { text: 'System:', attr: { for: 'systemSelect' } });
const systemSelect = systemGroup.createEl('select', { attr: { id: 'systemSelect', style: 'margin-right: 10px;' } });
systemSelect.createEl('option', { value: 'all', text: 'All Systems' });
strategiesAvailable.forEach(strategy => systemSelect.createEl('option', { value: strategy, text: strategy }));
systemSelect.value = 'all';

// Direction dropdown group
const directionGroup = dropdownDiv.createEl('div', { attr: { style: 'display: flex; align-items: center; gap: 5px;' } });
directionGroup.createEl('label', { text: 'Direction:', attr: { for: 'directionSelect' } });
const directionSelect = directionGroup.createEl('select', { attr: { id: 'directionSelect', style: 'margin-right: 10px;' } });
directionSelect.createEl('option', { value: 'all', text: 'Long / Short' });
directionsAvailable.forEach(direction => directionSelect.createEl('option', { value: direction, text: direction }));
directionSelect.value = 'all';

// Add Remove Filters button
const removeFiltersBtn = dropdownDiv.createEl('button', { text: 'Remove Filters', attr: { style: 'padding: 5px 10px; cursor: pointer; background-color: #000; border: 1px solid #333; border-radius: 3px; color: #fff; transition: background-color 0.3s;' } });
removeFiltersBtn.addEventListener('click', () => {
  yearSelect.value = 'all';
  monthSelect.value = 'all';
  daySelect.value = 'all';
  startDateInput.value = '';
  endDateInput.value = '';
  systemSelect.value = 'all';
  directionSelect.value = 'all';
  activeButton = 'monthly';
  yearlyBtn.classList.remove('active');
  monthlyBtn.classList.add('active');
  dailyBtn.classList.remove('active');
  generateMetrics();
});

const filterInfoDiv = mainDiv.createEl('div', { attr: { id: 'filterInfo', style: 'margin-bottom: 10px; font-size: 0.9em; color: #ccc;' } });
const metricsDiv = mainDiv.createEl('div', { attr: { id: 'metrics', style: 'margin-bottom: 30px;' } });
const chartsContainerDiv = mainDiv.createEl('div', {
  attr: {
    style: `
      display: flex;
      flex-wrap: wrap;
      justify-content: space-between;
      width: 100%;
      box-sizing: border-box;
      margin-bottom: 30px;
    `
  }
});

// Button group for period selection moved above PnL chart
const buttonGroup = chartsContainerDiv.createEl('div', { cls: 'button-group', attr: { style: 'display: flex; gap: 5px; margin-bottom: 10px; width: 100%;' } });
const yearlyBtn = buttonGroup.createEl('button', { text: 'Yearly' });
const monthlyBtn = buttonGroup.createEl('button', { text: 'Monthly' });
const dailyBtn = buttonGroup.createEl('button', { text: 'Daily' });
let activeButton = 'monthly'; // Default to monthly
yearlyBtn.addEventListener('click', () => {
  activeButton = 'yearly';
  yearlyBtn.classList.add('active');
  monthlyBtn.classList.remove('active');
  dailyBtn.classList.remove('active');
  generateMetrics();
});
monthlyBtn.addEventListener('click', () => {
  activeButton = 'monthly';
  monthlyBtn.classList.add('active');
  yearlyBtn.classList.remove('active');
  dailyBtn.classList.remove('active');
  generateMetrics();
});
dailyBtn.addEventListener('click', () => {
  activeButton = 'daily';
  dailyBtn.classList.add('active');
  yearlyBtn.classList.remove('active');
  monthlyBtn.classList.remove('active');
  generateMetrics();
});
monthlyBtn.classList.add('active'); // Set default active button to monthly

const tradesTableDiv = mainDiv.createEl('div', { attr: { id: 'tradesTable' } });

yearSelect.addEventListener("change", generateMetrics);
monthSelect.addEventListener("change", generateMetrics);
systemSelect.addEventListener("change", generateMetrics);
daySelect.addEventListener("change", generateMetrics);
startDateInput.addEventListener("change", generateMetrics);
endDateInput.addEventListener("change", generateMetrics);
directionSelect.addEventListener("change", generateMetrics);

function getDuration(entryDate, exitDate) {
  const start = moment.unix(Math.floor(entryDate / 1000));
  const end = moment.unix(Math.floor(exitDate / 1000));
  if (!start.isValid() || !end.isValid()) return 0;
  return Math.abs(end.diff(start, "days"));
}

async function generateMetrics() {
  // Clear all content before re-rendering
  filterInfoDiv.innerHTML = '';
  metricsDiv.innerHTML = '';
  chartsContainerDiv.innerHTML = '';
  tradesTableDiv.innerHTML = '';
  mainDiv.querySelectorAll('.no-trades-message').forEach(el => el.remove());

  // Re-create button group to ensure it stays above charts
  const buttonGroup = chartsContainerDiv.createEl('div', { cls: 'button-group', attr: { style: 'display: flex; gap: 5px; margin-bottom: 10px; width: 100%;' } });
  const yearlyBtn = buttonGroup.createEl('button', { text: 'Yearly' });
  const monthlyBtn = buttonGroup.createEl('button', { text: 'Monthly' });
  const dailyBtn = buttonGroup.createEl('button', { text: 'Daily' });
  yearlyBtn.addEventListener('click', () => {
    activeButton = 'yearly';
    yearlyBtn.classList.add('active');
    monthlyBtn.classList.remove('active');
    dailyBtn.classList.remove('active');
    generateMetrics();
  });
  monthlyBtn.addEventListener('click', () => {
    activeButton = 'monthly';
    monthlyBtn.classList.add('active');
    yearlyBtn.classList.remove('active');
    dailyBtn.classList.remove('active');
    generateMetrics();
  });
  dailyBtn.addEventListener('click', () => {
    activeButton = 'daily';
    dailyBtn.classList.add('active');
    yearlyBtn.classList.remove('active');
    monthlyBtn.classList.remove('active');
    generateMetrics();
  });
  if (activeButton === 'yearly') yearlyBtn.classList.add('active');
  else if (activeButton === 'monthly') monthlyBtn.classList.add('active');
  else dailyBtn.classList.add('active');

  const selectedYear = yearSelect.value;
  const selectedMonth = monthSelect.value;
  const selectedDay = daySelect.value;
  const startDate = startDateInput.value ? moment(startDateInput.value) : null;
  const endDate = endDateInput.value ? moment(endDateInput.value) : null;
  const selectedDirection = directionSelect.value;

  const tradesArray = Array.from(allTrades.filter(p => {
    if (!p.entry_date || !p.exit_date) return false;
    const date = moment.unix(Math.floor(p.exit_date / 1000));
    const strategy = p.strategy && p.strategy.length > 0 ? p.strategy[0].trim() : '';
    const direction = p.Direction && p.Direction.length > 0 ? p.Direction[0].trim() : '';

    // Year filter
    const yearMatch = selectedYear === 'all' || date.year() === Number(selectedYear);

    // Month filter
    let monthMatch = true;
    if (selectedMonth !== 'all') {
      if (['Q1', 'Q2', 'Q3', 'Q4'].includes(selectedMonth)) {
        const quarterStartMonth = { 'Q1': 0, 'Q2': 3, 'Q3': 6, 'Q4': 9 }[selectedMonth];
        monthMatch = date.month() >= quarterStartMonth && date.month() < quarterStartMonth + 3;
      } else {
        monthMatch = date.month() + 1 === Number(selectedMonth);
      }
    }

    // System filter
    const systemMatch = systemSelect.value === 'all' || strategy === systemSelect.value;

    // Direction filter
    const directionMatch = selectedDirection === 'all' || direction === selectedDirection;

    // Date range logic
    let dateInRange = true;
    if (startDate && endDate) {
      dateInRange = date.isBetween(startDate, endDate, null, '[]');
    } else if (startDate && !endDate) {
      dateInRange = date.isSameOrAfter(startDate);
    } else if (!startDate && endDate) {
      const earliestDate = allTrades.reduce((min, p) => {
        const d = moment.unix(Math.floor(p.exit_date / 1000));
        return d.isValid() && (!min || d.isBefore(min)) ? d : min;
      }, null);
      dateInRange = earliestDate && date.isSameOrAfter(earliestDate) && date.isSameOrBefore(endDate);
    }

    // Day of the week logic (based on exit date)
    let dayMatch = true;
    if (selectedDay !== 'all') {
      const dayOfWeek = date.format('dddd').toLowerCase();
      if (selectedDay === 'weekdays') {
        dayMatch = ['monday', 'tuesday', 'wednesday', 'thursday', 'friday'].includes(dayOfWeek);
      } else if (selectedDay === 'weekends') {
        dayMatch = ['saturday', 'sunday'].includes(dayOfWeek);
      } else {
        dayMatch = dayOfWeek === selectedDay;
      }
    }

    return date.isValid() && yearMatch && monthMatch && systemMatch && dateInRange && dayMatch && directionMatch;
  }));

  // Update filter info text only if at least one filter is applied or trades are filtered
  let filterText = '';
  let hasFilters = false;
  if (selectedYear !== 'all') { filterText += `Year ${selectedYear}, `; hasFilters = true; }
  if (selectedMonth !== 'all') {
    filterText += ['Q1', 'Q2', 'Q3', 'Q4'].includes(selectedMonth) ? `${selectedMonth}, ` :
                  `${moment().month(Number(selectedMonth) - 1).format("MMMM")}, `;
    hasFilters = true;
  }
  if (selectedDay !== 'all') { filterText += `Day ${selectedDay.charAt(0).toUpperCase() + selectedDay.slice(1)}, `; hasFilters = true; }
  if (startDate && endDate) { filterText += `from ${startDate.format("DD/MM/YYYY")} to ${endDate.format("DD/MM/YYYY")}, `; hasFilters = true; }
  else if (startDate) { filterText += `from ${startDate.format("DD/MM/YYYY")} onwards, `; hasFilters = true; }
  else if (endDate) {
    const earliestDate = allTrades.reduce((min, p) => {
      const d = moment.unix(Math.floor(p.exit_date / 1000));
      return d.isValid() && (!min || d.isBefore(min)) ? d : min;
    }, null);
    filterText += `from ${earliestDate ? earliestDate.format("DD/MM/YYYY") : 'start of data'} to ${endDate.format("DD/MM/YYYY")}, `;
    hasFilters = true;
  }
  if (systemSelect.value !== 'all') { filterText += `System ${systemSelect.value}, `; hasFilters = true; }
  if (selectedDirection !== 'all') { filterText += `Direction ${selectedDirection}, `; hasFilters = true; }
  filterText = filterText.replace(/, $/, ''); // Remove trailing comma and space

  if (hasFilters || tradesArray.length > 0) {
    filterInfoDiv.innerHTML = filterText || (tradesArray.length > 0 ? "Showing all trades." : '');
  }

  if (tradesArray.length === 0 && hasFilters) {
    mainDiv.createEl('p', {
      cls: 'no-trades-message',
      text: "No trades found for the selected filters.",
      attr: { style: 'margin-top: 10px; color: #ccc;' }
    });
    return;
  }

  const totalPnL = tradesArray.reduce((sum, p) => sum + (Number(p.PnL) || 0), 0);
  const rrValues = tradesArray.map(p => Number(p.RR)).filter(v => !isNaN(v));
  const totalRR = rrValues.length > 0 ? rrValues.reduce((a, b) => a + b, 0).toFixed(2) : "N/A";
  const durations = tradesArray.map(p => getDuration(p.entry_date, p.exit_date));
  const avgDuration = durations.length > 0 ? (durations.reduce((a, b) => a + b, 0) / durations.length).toFixed(2) : "N/A";
  const winningTrades = tradesArray.filter(p => Number(p.PnL) > 0).length;
  const winrate = ((winningTrades / tradesArray.length) * 100).toFixed(2) + "%";

  // Calculate Long and Short trade counts
  const longTrades = tradesArray.filter(p => p.Direction && p.Direction[0] && p.Direction[0].toLowerCase() === 'long').length;
  const shortTrades = tradesArray.filter(p => p.Direction && p.Direction[0] && p.Direction[0].toLowerCase() === 'short').length;

  // Calculate avgWin and avgLoss from RR
  const avgWinRR = rrValues.length > 0 ? (rrValues.filter(rr => rr > 0).reduce((sum, rr) => sum + rr, 0) / winningTrades).toFixed(2) : "N/A";
  const avgWin = avgWinRR !== "N/A" ? Number(avgWinRR) : "N/A";
  const losingTrades = tradesArray.filter(p => Number(p.PnL) <= 0).length;
  const avgLossRR = rrValues.length > 0 ? (rrValues.filter(rr => rr <= 0).reduce((sum, rr) => sum + rr, 0) / losingTrades).toFixed(2) : "N/A";
  const avgLoss = avgLossRR !== "N/A" ? Number(avgLossRR) : "N/A";

  const ev = avgWin !== "N/A" && avgLoss !== "N/A"
    ? (((Number(winrate.replace("%", "")) / 100) * avgWin) - ((1 - (Number(winrate.replace("%", "")) / 100)) * Math.abs(avgLoss))).toFixed(2)
    : "N/A";

  metricsDiv.innerHTML = `
    <table class="metrics-table" style="width: 100%; border-collapse: collapse; text-align: left; color: #ccc; background-color: #222;">
      <thead><tr><th style="padding: 8px; border: 1px solid #444;">Metric</th><th style="padding: 8px; border: 1px solid #444;">Value</th></tr></thead>
      <tbody>
        <tr><td style="padding: 8px; border: 1px solid #444;">Total Trades</td><td style="padding: 8px; border: 1px solid #444;">${tradesArray.length}</td></tr>
        <tr><td style="padding: 8px; border: 1px solid #444;">Long Trades</td><td style="padding: 8px; border: 1px solid #444;">${longTrades}</td></tr>
        <tr><td style="padding: 8px; border: 1px solid #444;">Short Trades</td><td style="padding: 8px; border: 1px solid #444;">${shortTrades}</td></tr>
        <tr><td style="padding: 8px; border: 1px solid #444;">Total PnL</td><td style="padding: 8px; border: 1px solid #444;">${totalPnL.toFixed(1)}$</td></tr>
        <tr><td style="padding: 8px; border: 1px solid #444;">Total RR</td><td style="padding: 8px; border: 1px solid #444;">${totalRR}</td></tr>
        <tr><td style="padding: 8px; border: 1px solid #444;">Average Duration</td><td style="padding: 8px; border: 1px solid #444;">${avgDuration} day(s)</td></tr>
        <tr><td style="padding: 8px; border: 1px solid #444;">Winrate</td><td style="padding: 8px; border: 1px solid #444;">${winrate}</td></tr>
        <tr><td style="padding: 8px; border: 1px solid #444;">EV</td><td style="padding: 8px; border: 1px solid #444;">${ev}</td></tr>
      </tbody>
    </table>
  `;

  // Calculate period-based data for charts
  let effectiveActiveButton = activeButton;
  if (selectedMonth !== 'all' && !['Q1', 'Q2', 'Q3', 'Q4'].includes(selectedMonth)) {
    effectiveActiveButton = 'daily';
  }

  let periodPnL = {};
  let periodRR = {};
  tradesArray.forEach(p => {
    const date = moment.unix(Math.floor(p.exit_date / 1000));
    const key = effectiveActiveButton === 'yearly' ? date.year() :
                effectiveActiveButton === 'monthly' ? date.format('YYYY-MM') :
                date.format('YYYY-MM-DD');
    if (!periodPnL[key]) periodPnL[key] = [];
    if (!periodRR[key]) periodRR[key] = [];
    periodPnL[key].push(Number(p.PnL) || 0);
    periodRR[key].push(Number(p.RR) || 0);
  });

  // Aggregated PnL chart: cumulative sum per period
  const sortedPeriods = Object.keys(periodPnL).sort((a, b) => moment(a).diff(moment(b)));
  let cumulativePnL = 0;
  const aggregatedData = sortedPeriods.map(period => {
    const periodSum = periodPnL[period].reduce((a, b) => a + b, 0);
    cumulativePnL += periodSum;
    return { period, value: cumulativePnL };
  });

  const periodMap = {
    'yearly': 'year',
    'monthly': 'month',
    'daily': 'day'
  };
  const periodLabel = periodMap[effectiveActiveButton];

  // Get the first value of the selected period for the grey line
  const firstPnLValue = aggregatedData.length > 0 ? aggregatedData[0].value : 0;

  const aggregatedChartConfig = {
    type: 'line',
    data: {
      labels: aggregatedData.map(d => 
        effectiveActiveButton === 'yearly' ? d.period :
        effectiveActiveButton === 'monthly' ? moment(d.period).format('MMMM YYYY') :
        moment(d.period).format('DD-MM-YYYY')
      ),
      datasets: [{
        label: `Cumulative PnL ($) by ${periodLabel}`,
        data: aggregatedData.map(d => d.value),
        backgroundColor: 'rgba(54, 162, 235, 0.2)',
        borderColor: 'rgba(54, 162, 235, 1)',
        borderWidth: 2,
        fill: false
      }]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      scales: {
        y: { beginAtZero: true, title: { display: true, text: 'Cumulative PnL ($)' } },
        x: { title: { display: true, text: periodLabel } }
      },
      plugins: {
        annotation: {
          annotations: {
            firstValueLine: {
              type: 'line',
              yMin: firstPnLValue,
              yMax: firstPnLValue,
              borderColor: '#808080',
              borderWidth: 2,
              borderDash: [5, 5],
              label: {
                content: `${firstPnLValue.toFixed(1)}`,
                enabled: true,
                position: 'end',
                backgroundColor: 'rgba(128, 128, 128, 0.7)',
                color: '#fff',
                font: {
                  size: 12
                }
              }
            }
          }
        }
      }
    }
  };

  if (window.renderChart) {
    const chartStyle = `
      flex-grow: 1;
      flex-shrink: 1;
      flex-basis: 100%;
      min-width: 100%;
      max-width: 100%;
      margin-bottom: 20px;
      height: 400px;
      box-sizing: border-box;
      @media (min-width: 768px) {
        flex-basis: 49.5%;
        min-width: 450px;
        max-width: 49.5%;
      }
    `;

    const aggChartWrapper = chartsContainerDiv.createEl('div', { attr: { style: chartStyle } });

    await window.renderChart(aggregatedChartConfig, aggChartWrapper);

    // R chart: can toggle between average and total
    const rChartWrapper = chartsContainerDiv.createEl('div', { attr: { style: chartStyle } });
    const rToggleGroup = rChartWrapper.createEl('div', { cls: 'button-group', attr: { style: 'display: flex; gap: 5px; margin-bottom: 10px;' } });
    const avgRBtn = rToggleGroup.createEl('button', { text: 'Average R' });
    const totalRBtn = rToggleGroup.createEl('button', { text: 'Total R' });
    let rChartMode = 'average'; // Default to average
    avgRBtn.classList.add('active');

    const updateRChart = () => {
      const dataValues = sortedPeriods.map(period => {
        const periodRRs = periodRR[period] || [];
        return rChartMode === 'average'
          ? (periodRRs.reduce((a, b) => a + b, 0) / (periodRRs.length || 1) || 0).toFixed(2)
          : periodRRs.reduce((a, b) => a + b, 0).toFixed(2);
      });

      const rChartConfig = {
        type: 'line',
        data: {
          labels: sortedPeriods.map(period =>
            effectiveActiveButton === 'yearly' ? period :
            effectiveActiveButton === 'monthly' ? moment(period).format('MMMM YYYY') :
            moment(period).format('DD-MM-YYYY')),
          datasets: [{
            label: `${rChartMode.charAt(0).toUpperCase() + rChartMode.slice(1)} R by ${periodLabel}`,
            data: dataValues,
            backgroundColor: 'rgba(75, 192, 192, 0.2)',
            borderColor: 'rgba(75, 192, 192, 1)',
            borderWidth: 2,
            fill: false
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          scales: {
            y: {
              beginAtZero: false,
              title: { display: true, text: `${rChartMode.charAt(0).toUpperCase() + rChartMode.slice(1)} R` }
            },
            x: { title: { display: true, text: periodLabel } }
          },
          plugins: {
            annotation: {
              annotations: {
                zeroLine: {
                  type: 'line',
                  yMin: 0,
                  yMax: 0,
                  borderColor: '#808080',
                  borderWidth: 2,
                  borderDash: [5, 5],
                  label: {
                    content: '0R',
                    enabled: true,
                    position: 'end',
                    backgroundColor: 'rgba(128, 128, 128, 0.7)',
                    color: '#fff',
                    font: {
                      size: 12
                    }
                  }
                }
              }
            }
          }
        }
      };

      rChartContainer.innerHTML = '';
      window.renderChart(rChartConfig, rChartContainer);
    };

    const rChartContainer = rChartWrapper.createEl('div', { attr: { style: 'height: 100%;' } });

    avgRBtn.addEventListener('click', () => {
      rChartMode = 'average';
      avgRBtn.classList.add('active');
      totalRBtn.classList.remove('active');
      updateRChart();
    });

    totalRBtn.addEventListener('click', () => {
      rChartMode = 'total';
      totalRBtn.classList.add('active');
      avgRBtn.classList.remove('active');
      updateRChart();
    });

    updateRChart();
  } else {
    chartsContainerDiv.createEl('p', {
      text: "Charts plugin's renderChart function not found. Please ensure the Charts plugin is installed and enabled.",
      attr: { style: 'color: #ccc;' }
    });
  }

  if (tradesArray.length > 0) {
    let tableHTML = `
      <h3 style="margin-top: 20px; margin-bottom: 15px; color: #ccc;">Filtered Trades (${tradesArray.length})</h3>
      <table class="trades-data-table" style="width: 100%; border-collapse: collapse; color: #ccc; background-color: #222;">
        <thead><tr>
          <th style="padding: 8px; border: 1px solid #444;">Ticker</th>
          <th style="padding: 8px; border: 1px solid #444;">Strategy</th>
          <th style="padding: 8px; border: 1px solid #444;">Entry Date</th>
          <th style="padding: 8px; border: 1px solid #444;">Exit Date</th>
          <th style="padding: 8px; border: 1px solid #444;">Duration (Days)</th>
          <th style="padding: 8px; border: 1px solid #444;">PnL ($)</th>
          <th style="padding: 8px; border: 1px solid #444;">RR</th>
          <th style="padding: 8px; border: 1px solid #444;">Direction</th>
          <th style="padding: 8px; border: 1px solid #444;">File</th>
        </tr></thead><tbody>
    `;

    // Sort trades from newest to oldest based on exit_date
    tradesArray.sort((a, b) => moment.unix(Math.floor(b.exit_date / 1000)).valueOf() - moment.unix(Math.floor(a.exit_date / 1000)).valueOf()).forEach(trade => {
      const pnl = (Number(trade.PnL) || 0).toFixed(1);
      const rr = (Number(trade.RR) || 0).toFixed(2);
      const ticker = trade.file.name.match(/^([A-Za-z0-9]+)\s/)?.[1] || 'N/A';
      const entry = moment.unix(Math.floor(trade.entry_date / 1000)).format("DD-MM-YYYY");
      const exit = moment.unix(Math.floor(trade.exit_date / 1000)).format("DD-MM-YYYY");
      const duration = getDuration(trade.entry_date, trade.exit_date);
      const direction = trade.Direction && trade.Direction.length > 0 ? trade.Direction[0] : 'N/A';
      const link = `obsidian://open?vault=${encodeURIComponent(app.vault.getName())}&file=${encodeURIComponent(trade.file.path)}`;
      const pnlColor = pnl > 0 ? 'background-color: rgba(54, 162, 235, 0.2); color: rgba(54, 162, 235, 1);' : 'background-color: rgba(255, 99, 132, 0.2); color: rgba(255, 99, 132, 1);';
      const rrColor = rr > 0 ? 'background-color: rgba(54, 162, 235, 0.2); color: rgba(54, 162, 235, 1);' : 'background-color: rgba(255, 99, 132, 0.2); color: rgba(255, 99, 132, 1);';

      tableHTML += `
        <tr>
          <td style="padding: 8px; border: 1px solid #444;">${ticker}</td>
          <td style="padding: 8px; border: 1px solid #444;">${trade.strategy ? trade.strategy.join(', ') : 'N/A'}</td>
          <td style="padding: 8px; border: 1px solid #444;">${entry}</td>
          <td style="padding: 8px; border: 1px solid #444;">${exit}</td>
          <td style="padding: 8px; border: 1px solid #444;">${duration} (Days)</td>
          <td style="padding: 8px; border: 1px solid #444; ${pnlColor}">${pnl}$</td>
          <td style="padding: 8px; border: 1px solid #444; ${rrColor}">${rr}</td>
          <td style="padding: 8px; border: 1px solid #444;">${direction}</td>
          <td style="padding: 8px; border: 1px solid #444;"><a href="${link}" target="_blank" rel="noopener" style="color: #007bff;">${trade.file.name}</a></td>
        </tr>
      `;
    });

    tableHTML += '</tbody></table>';
    tradesTableDiv.innerHTML = tableHTML;
  }
}

// Initial render
generateMetrics();
```