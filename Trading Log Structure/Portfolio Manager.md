```dataviewjs
// Calculate total trade PnL
function calculateTotalTradePnL(allTrades) {
    return allTrades.reduce((sum, trade) => {
        const pnl = Number(trade.PnL) || 0;
        return sum + pnl;
    }, 0);
}

// Calculate Maximum Drawdown from trades with initial balance
function calculateMaxDrawdown(trades, initialBalance) {
    if (!trades || trades.length === 0) {
        return 0;
    }
    let peak = initialBalance;
    let maxDrawdown = 0;
    let currentPortfolioValue = initialBalance;
    const sortedTrades = [...trades].sort((a, b) => {
        const dateA = moment.unix(Math.floor(a.exit_date / 1000));
        const dateB = moment.unix(Math.floor(b.exit_date / 1000));
        return dateA.diff(dateB);
    });
    const pnlOnly = sortedTrades.map(t => Number(t.PnL) || 0).filter(pnl => !isNaN(pnl));
    for (let i = 0; i < pnlOnly.length; i++) {
        currentPortfolioValue += pnlOnly[i];
        if (currentPortfolioValue > peak) {
            peak = currentPortfolioValue;
        }
        if (peak > 0) {
            const drawdown = (peak - currentPortfolioValue) / peak;
            if (drawdown > maxDrawdown) {
                maxDrawdown = drawdown;
            }
        }
    }
    return maxDrawdown;
}

// Calculate Return on Investment
function calculateROI(initialBalance, deposits, withdrawals, totalPnL) {
    const totalInvestment = initialBalance + deposits - withdrawals;
    if (totalInvestment === 0) {
        return 0;
    }
    return (totalPnL / totalInvestment) * 100;
}

// Calculate Sharpe Ratio
function computeSharpe(events, initialBalance, startDate = null, endDate = null) {
    if (events.length === 0) return 0;
    events = events.filter(e => moment(e.date, 'DD-MM-YYYY', true).isValid());
    if (events.length === 0) return 0;
    events.sort((a, b) => moment(a.date, 'DD-MM-YYYY').diff(moment(b.date, 'DD-MM-YYYY')));
    const monthlyBalances = new Map();
    let tempBal = initialBalance;
    let lastMonth = null;
    for (let i = 0; i < events.length; i++) {
        const event = events[i];
        const em = moment(event.date, 'DD-MM-YYYY');
        const mk = em.format('YYYY-MM');
        if (mk !== lastMonth) {
            if (lastMonth !== null) {
                monthlyBalances.set(lastMonth, tempBal);
            }
            lastMonth = mk;
        }
        const amountToAdd = event.type === 'Withdrawal' ? -event.amount : event.amount;
        tempBal += amountToAdd;
    }
    if (lastMonth !== null) {
        monthlyBalances.set(lastMonth, tempBal);
    }
    let startM, endM;
    if (!startDate || !endDate) {
        startM = moment(events[0].date, 'DD-MM-YYYY').startOf('month');
        endM = moment(events[events.length - 1].date, 'DD-MM-YYYY').endOf('month');
    } else {
        startM = moment(startDate).startOf('month');
        endM = moment(endDate).endOf('month');
    }
    const monthlyReturns = [];
    let prevBalance = initialBalance;
    let currentM = startM.clone();
    while (currentM.isSameOrBefore(endM)) {
        const mk = currentM.format('YYYY-MM');
        let thisBalance = monthlyBalances.get(mk) || prevBalance;
        if (prevBalance > 0) {
            monthlyReturns.push((thisBalance - prevBalance) / prevBalance);
        }
        prevBalance = thisBalance;
        currentM.add(1, 'month');
    }
    if (monthlyReturns.length < 2) return 0;
    const mean = monthlyReturns.reduce((a, b) => a + b, 0) / monthlyReturns.length;
    const variance = monthlyReturns.reduce((a, b) => a + Math.pow(b - mean, 2), 0) / (monthlyReturns.length - 1);
    const std = Math.sqrt(variance);
    if (std === 0) return 0;
    return (mean / std) * Math.sqrt(12);
}

// Calculate Sortino Ratio
function computeSortino(events, initialBalance, startDate = null, endDate = null) {
    if (events.length === 0) return 0;
    events = events.filter(e => moment(e.date, 'DD-MM-YYYY', true).isValid());
    if (events.length === 0) return 0;
    events.sort((a, b) => moment(a.date, 'DD-MM-YYYY').diff(moment(b.date, 'DD-MM-YYYY')));
    const monthlyBalances = new Map();
    let tempBal = initialBalance;
    let lastMonth = null;
    for (let i = 0; i < events.length; i++) {
        const event = events[i];
        const em = moment(event.date, 'DD-MM-YYYY');
        const mk = em.format('YYYY-MM');
        if (mk !== lastMonth) {
            if (lastMonth !== null) {
                monthlyBalances.set(lastMonth, tempBal);
            }
            lastMonth = mk;
        }
        const amountToAdd = event.type === 'Withdrawal' ? -event.amount : event.amount;
        tempBal += amountToAdd;
    }
    if (lastMonth !== null) {
        monthlyBalances.set(lastMonth, tempBal);
    }
    let startM, endM;
    if (!startDate || !endDate) {
        startM = moment(events[0].date, 'DD-MM-YYYY').startOf('month');
        endM = moment(events[events.length - 1].date, 'DD-MM-YYYY').endOf('month');
    } else {
        startM = moment(startDate).startOf('month');
        endM = moment(endDate).endOf('month');
    }
    const monthlyReturns = [];
    let prevBalance = initialBalance;
    let currentM = startM.clone();
    while (currentM.isSameOrBefore(endM)) {
        const mk = currentM.format('YYYY-MM');
        let thisBalance = monthlyBalances.get(mk) || prevBalance;
        if (prevBalance > 0) {
            monthlyReturns.push((thisBalance - prevBalance) / prevBalance);
        }
        prevBalance = thisBalance;
        currentM.add(1, 'month');
    }
    if (monthlyReturns.length < 2) return 0;
    const mean = monthlyReturns.reduce((a, b) => a + b, 0) / monthlyReturns.length;
    const downsideReturns = monthlyReturns.filter(r => r < 0);
    let downsideStd = 0;
    if (downsideReturns.length > 0) {
        const downsideMean = downsideReturns.reduce((a, b) => a + b, 0) / downsideReturns.length;
        const downsideVariance = downsideReturns.reduce((a, b) => a + Math.pow(b - downsideMean, 2), 0) / (downsideReturns.length - 1);
        downsideStd = Math.sqrt(downsideVariance);
    }
    if (downsideStd === 0) return 0;
    return (mean / downsideStd) * Math.sqrt(12);
}

const container = this.container;
container.innerHTML = '';
const mainDiv = container.createEl('div');
mainDiv.classList.add('portfolio-container');
const portfolioFilePath = 'Trades/Portfolio Movements.md';
let currentAction = '';
let currentTimeframe = 'monthly';
let currentYear = 'all';
let currentChartType = 'balance';
let data = { initialBalance: 0, movements: [] };
let trades = [];
let computed = {};

function getPortfolioData() {
    return new Promise(async (resolve) => {
        try {
            const portfolioFile = app.vault.getAbstractFileByPath(portfolioFilePath);
            if (portfolioFile) {
                const content = await app.vault.read(portfolioFile);
                let initialBalance = 0;
                let movements = [];
                const initialMatch = content.match(/\*\*Initial Portfolio Size:\*\*\s*\$?([0-9,.]+(?![0-9,.]))/);
                if (initialMatch) {
                    initialBalance = parseFloat(initialMatch[1].replace(/,/g, '.'));
                }
                const tableRowsMatch = content.match(/\|([^|]*)\|([^|]*)\|([^|]*)\|([^|]*)\|([^|]*)\|/g);
                if (tableRowsMatch) {
                    const dataRows = tableRowsMatch.slice(2);
                    dataRows.forEach(row => {
                        const cols = row.split('|').map(col => col.trim());
                        if (cols.length >= 6) {
                            let date = cols[1];
                            if (moment(date, 'YYYY-MM-DD', true).isValid()) {
                                date = moment(date, 'YYYY-MM-DD').format('DD-MM-YYYY');
                            }
                            movements.push({
                                date: date,
                                type: cols[2],
                                amount: parseFloat(cols[3].replace(/[$,]/g, '').replace(/,/g, '.')),
                                balance: parseFloat(cols[4].replace(/[$,]/g, '').replace(/,/g, '.')),
                                description: cols[5]
                            });
                        }
                    });
                }
                resolve({ initialBalance, movements });
            } else {
                resolve({ initialBalance: 0, movements: [] });
            }
        } catch (error) {
            console.log('Portfolio file not found or parsing error:', error);
            resolve({ initialBalance: 0, movements: [] });
        }
    });
}

function savePortfolioData(initialBalance, movements) {
    return new Promise(async (resolve) => {
        const tableRows = movements.map(m => '| ' + m.date + ' | ' + m.type + ' | $' + m.amount.toFixed(2) + ' | $' + m.balance.toFixed(2) + ' | ' + m.description + ' |').join('\n');
        const content = '# Portfolio Movements\n\n**Initial Portfolio Size:** $' + initialBalance.toFixed(2) + '\n\n## Movement History\n\n| Date | Type | Amount | Balance | Description |\n|------|------|--------|---------|-------------|\n' + tableRows + '\n\n---\n\n*This file is automatically managed by the Portfolio Manager*';
        try {
            const portfolioFile = app.vault.getAbstractFileByPath(portfolioFilePath);
            if (portfolioFile) {
                await app.vault.modify(portfolioFile, content);
            } else {
                await app.vault.create(portfolioFilePath, content);
            }
            resolve();
        } catch (error) {
            console.error('Error saving portfolio file:', error);
            resolve();
        }
    });
}

function loadData() {
    return new Promise(async (resolve) => {
        data = await getPortfolioData();
        try {
            if (typeof dv !== 'undefined' && dv.pages) {
                const rawPages = dv.pages('"Trades/Trade"') || [];
                trades = Array.from(rawPages).filter(p => p && p.file && !p.file.name.includes("Template") && p.exit_date);
            }
        } catch (error) {
            console.error('Error loading trades:', error);
            trades = [];
        }
        computed = computeStats(data, trades);
        resolve();
    });
}

function computeStats(data, trades) {
    const { initialBalance, movements } = data;
    const totalTradePnL = calculateTotalTradePnL(trades);
    const manualPnLMovements = movements.filter(m => m.type === 'Investment closed' && moment(m.date, 'DD-MM-YYYY', true).isValid());
    const totalManualPnl = manualPnLMovements.reduce((sum, m) => sum + m.amount, 0);
    const maxDrawdown = calculateMaxDrawdown([
        ...trades,
        ...manualPnLMovements.map(m => ({
            exit_date: moment(m.date, 'DD-MM-YYYY').unix() * 1000,
            PnL: m.amount
        }))
    ], initialBalance);
    const depositsTotal = movements.filter(m => m.type === 'Deposit').reduce((sum, m) => sum + m.amount, 0);
    const withdrawalsTotal = movements.filter(m => m.type === 'Withdrawal').reduce((sum, m) => sum + m.amount, 0);
    const totalPnL = totalTradePnL + totalManualPnl;
    const currentBalance = initialBalance + depositsTotal - withdrawalsTotal + totalPnL;
    const totalChange = currentBalance - initialBalance;
    const percentChange = initialBalance > 0 ? ((totalChange / initialBalance) * 100) : 0;
    const roi = calculateROI(initialBalance, depositsTotal, withdrawalsTotal, totalPnL);
    let allEvents = movements.map(m => ({ date: m.date, type: m.type, amount: m.amount }));
    trades.forEach(t => {
        const d = moment.unix(Math.floor(t.exit_date / 1000)).format('DD-MM-YYYY');
        if (moment(d, 'DD-MM-YYYY', true).isValid()) {
            allEvents.push({ date: d, type: 'Trade P&L', amount: Number(t.PnL) || 0 });
        }
    });
    const sharpe = computeSharpe(allEvents, initialBalance);
    const sortino = computeSortino(allEvents, initialBalance);
    return { totalTradePnL, totalManualPnl, maxDrawdown, depositsTotal, withdrawalsTotal, totalPnL, currentBalance, totalChange, percentChange, roi, initialBalance, sharpe, sortino };
}

function computeYearStats(year, data, trades) {
    const yearStart = moment(`${year}-01-01`, 'YYYY-MM-DD');
    const yearEnd = moment(`${year}-12-31`, 'YYYY-MM-DD');
    const previousMovements = data.movements.filter(m => moment(m.date, 'DD-MM-YYYY', true).isValid() && moment(m.date, 'DD-MM-YYYY').isBefore(yearStart));
    const previousTrades = trades.filter(t => moment.unix(Math.floor(t.exit_date / 1000)).isValid() && moment.unix(Math.floor(t.exit_date / 1000)).isBefore(yearStart));
    const previousManualPnL = previousMovements.filter(m => m.type === 'Investment closed');
    const initialForYear = data.initialBalance +
        previousMovements.reduce((sum, m) => {
            if (m.type === 'Deposit') return sum + m.amount;
            if (m.type === 'Withdrawal') return sum - m.amount;
            return sum;
        }, 0) +
        calculateTotalTradePnL(previousTrades) +
        previousManualPnL.reduce((sum, m) => sum + m.amount, 0);
    const yearMovements = data.movements.filter(m => moment(m.date, 'DD-MM-YYYY', true).isValid() && moment(m.date, 'DD-MM-YYYY').year() === year);
    const yearTrades = trades.filter(t => moment.unix(Math.floor(t.exit_date / 1000)).isValid() && moment.unix(Math.floor(t.exit_date / 1000)).year() === year);
    const yearManualPnL = yearMovements.filter(m => m.type === 'Investment closed');
    const depositsTotal = yearMovements.filter(m => m.type === 'Deposit').reduce((sum, m) => sum + m.amount, 0);
    const withdrawalsTotal = yearMovements.filter(m => m.type === 'Withdrawal').reduce((sum, m) => sum + m.amount, 0);
    const totalManualPnl = yearManualPnL.reduce((sum, m) => sum + m.amount, 0);
    const totalTradePnL = calculateTotalTradePnL(yearTrades);
    const maxDrawdown = calculateMaxDrawdown([
        ...yearTrades,
        ...yearManualPnL.map(m => ({
            exit_date: moment(m.date, 'DD-MM-YYYY').unix() * 1000,
            PnL: m.amount
        }))
    ], initialForYear) || 0;
    const totalPnL = totalTradePnL + totalManualPnl;
    const currentBalance = initialForYear + depositsTotal - withdrawalsTotal + totalPnL;
    const totalChange = currentBalance - initialForYear;
    const percentChange = initialForYear > 0 ? ((totalChange / initialForYear) * 100) : 0;
    const roi = calculateROI(initialForYear, depositsTotal, withdrawalsTotal, totalPnL);
    let yearEvents = yearMovements.map(m => ({ date: m.date, type: m.type, amount: m.amount }));
    yearTrades.forEach(t => {
        const d = moment.unix(Math.floor(t.exit_date / 1000)).format('DD-MM-YYYY');
        if (moment(d, 'DD-MM-YYYY', true).isValid()) {
            yearEvents.push({ date: d, type: 'Trade P&L', amount: Number(t.PnL) || 0 });
        }
    });
    const sharpe = computeSharpe(yearEvents, initialForYear, `${year}-01-01`, `${year}-12-31`);
    const sortino = computeSortino(yearEvents, initialForYear, `${year}-01-01`, `${year}-12-31`);
    return { initialBalance: initialForYear, depositsTotal, withdrawalsTotal, totalTradePnL, totalManualPnl, maxDrawdown, roi, totalPnL, currentBalance, totalChange, percentChange, sharpe, sortino };
}

function renderPortfolio() {
    mainDiv.innerHTML = '';
    let portfolioValueEl, portfolioChangeEl;
    const headerDiv = mainDiv.createEl('div', { cls: 'portfolio-header' });
    const valueDiv = headerDiv.createEl('div');
    valueDiv.createEl('div', { text: 'Portfolio Value', attr: { style: 'font-size: 0.9em; color: #888; margin-bottom: 5px;' } });
    portfolioValueEl = valueDiv.createEl('div', { text: '$' + computed.currentBalance.toFixed(2), cls: 'portfolio-value' });
    const changeDiv = headerDiv.createEl('div', { attr: { style: 'text-align: right;' } });
    changeDiv.createEl('div', { text: 'Total Change', attr: { style: 'font-size: 0.9em; color: #888; margin-bottom: 5px;' } });
    const changeSign = computed.totalChange >= 0 ? '+' : '';
    const percentSign = computed.percentChange >= 0 ? '+' : '';
    portfolioChangeEl = changeDiv.createEl('div', {
        text: changeSign + '$' + computed.totalChange.toFixed(2) + ' (' + percentSign + computed.percentChange.toFixed(2) + '%)',
        cls: `portfolio-change ${computed.totalChange >= 0 ? 'positive' : 'negative'}`
    });
    function updateHeader() {
        portfolioValueEl.innerText = '$' + computed.currentBalance.toFixed(2);
        const changeSign = computed.totalChange >= 0 ? '+' : '';
        const percentSign = computed.percentChange >= 0 ? '+' : '';
        const changeText = changeSign + '$' + computed.totalChange.toFixed(2) + ' (' + percentSign + computed.percentChange.toFixed(2) + '%)';
        portfolioChangeEl.innerText = changeText;
        portfolioChangeEl.className = `portfolio-change ${computed.totalChange >= 0 ? 'positive' : 'negative'}`;
    }
    const actionsDiv = mainDiv.createEl('div', { cls: 'portfolio-actions' });
    if (data.initialBalance === 0) {
        const setInitialBtn = actionsDiv.createEl('button', { text: 'Set initial balance', cls: 'action-btn' });
        setInitialBtn.addEventListener('click', () => {
            currentAction = 'initial';
            inputDiv.style.display = 'flex';
            amountInput.focus();
            amountInput.value = data.initialBalance.toString();
            descInput.style.display = 'none';
            inputDiv.querySelector('label').textContent = 'Amount:';
            errorMessage.textContent = '';
        });
    }
    const depositBtn = actionsDiv.createEl('button', { text: 'Deposit', cls: 'action-btn deposit' });
    depositBtn.addEventListener('click', () => {
        currentAction = 'deposit';
        inputDiv.style.display = 'flex';
        amountInput.focus();
        amountInput.value = '';
        descInput.value = '';
        descInput.style.display = 'block';
        inputDiv.querySelector('label').textContent = 'Amount:';
        errorMessage.textContent = '';
    });
    const withdrawBtn = actionsDiv.createEl('button', { text: 'Withdrawal', cls: 'action-btn withdraw' });
    withdrawBtn.addEventListener('click', () => {
            currentAction = 'withdraw';
            inputDiv.style.display = 'flex';
            amountInput.focus();
            amountInput.value = '';
            descInput.value = '';
            descInput.style.display = 'block';
            inputDiv.querySelector('label').textContent = 'Amount:';
            errorMessage.textContent = '';
    });
    const tradeBtn = actionsDiv.createEl('button', { text: 'Investment closed', cls: 'action-btn trade' });
    tradeBtn.addEventListener('click', () => {
        currentAction = 'investment closed';
        inputDiv.style.display = 'flex';
        amountInput.focus();
        amountInput.value = '';
        descInput.value = '';
        descInput.style.display = 'block';
        inputDiv.querySelector('label').textContent = 'P&L:';
        errorMessage.textContent = '';
    });
    const inputDiv = mainDiv.createEl('div', { cls: 'input-group', attr: { style: 'display: none;' } });
    inputDiv.createEl('label', { text: 'Amount:' });
    const amountInput = inputDiv.createEl('input', { attr: { type: 'number', step: '0.01', placeholder: 'Enter amount' } });
    const descInput = inputDiv.createEl('input', { attr: { type: 'text', placeholder: 'Description (optional)' } });
    const errorMessage = inputDiv.createEl('div', { cls: 'error-message' });
    const confirmBtn = inputDiv.createEl('button', { text: 'Confirm', cls: 'action-btn' });
    const cancelBtn = inputDiv.createEl('button', { text: 'Cancel', cls: 'action-btn', attr: { style: 'background-color: #6c757d;' } });
    cancelBtn.addEventListener('click', () => {
        inputDiv.style.display = 'none';
        amountInput.value = '';
        descInput.value = '';
        errorMessage.textContent = '';
    });
    confirmBtn.addEventListener('click', async () => {
        const amount = parseFloat(amountInput.value.replace(/,/g, '.'));
        const description = descInput.value || '';
        if (isNaN(amount) || amount === 0) {
            errorMessage.textContent = 'Please enter a valid amount';
            return;
        }
        if (currentAction !== 'investment closed' && amount <= 0) {
            errorMessage.textContent = 'Amount must be positive for this action';
            return;
        }
        errorMessage.textContent = '';
        const { initialBalance: currentInitial, movements: currentMovements } = data;
        let newInitialBalance = currentInitial;
        let newMovements = [...currentMovements];

        let newBalance = currentInitial;
        if (currentMovements.length > 0) {
            newBalance = currentMovements[currentMovements.length - 1].balance;
        }

        if (currentAction === 'initial') {
            newInitialBalance = amount;
            newMovements = [];
            newBalance = amount;
        } else {
            if (currentAction === 'deposit') {
                newBalance += amount;
            } else if (currentAction === 'withdraw') {
                newBalance -= amount; // Correctly subtract withdrawal amount
            } else { // 'investment closed'
                newBalance += amount;
            }
            newMovements.push({
                date: moment().format('DD-MM-YYYY'),
                type: currentAction === 'deposit' ? 'Deposit' : (currentAction === 'withdraw' ? 'Withdrawal' : 'Investment closed'),
                amount: amount,
                balance: newBalance,
                description: description
            });
        }
        inputDiv.style.display = 'none';
        amountInput.value = '';
        descInput.value = '';
        await savePortfolioData(newInitialBalance, newMovements);
        data = await getPortfolioData();
        trades = [];
        try {
            if (typeof dv !== 'undefined' && dv.pages) {
                const rawPages = dv.pages('"Trades/Trade"') || [];
                trades = Array.from(rawPages).filter(p => p && p.file && !p.file.name.includes("Template") && p.exit_date);
            }
        } catch (error) {
            console.error('Error loading trades:', error);
            trades = [];
        }
        computed = computeStats(data, trades);
        renderPortfolio();
    });
    const yearDiv = mainDiv.createEl('div', { cls: 'year-selector' });
    yearDiv.createEl('label', { text: 'Year:' });
    const yearSelect = yearDiv.createEl('select');
    const yearsSet = new Set();
    data.movements.forEach(m => {
        const y = moment(m.date, 'DD-MM-YYYY').year();
        if (!isNaN(y)) yearsSet.add(y);
    });
    trades.forEach(t => {
        const y = moment.unix(Math.floor(t.exit_date / 1000)).year();
        if (!isNaN(y)) yearsSet.add(y);
    });
    yearsSet.add(moment().year());
    const allYears = Array.from(yearsSet).sort((a, b) => a - b);
    yearSelect.createEl('option', { text: 'All', attr: { value: 'all' } }).selected = true;
    allYears.forEach(y => {
        yearSelect.createEl('option', { text: y.toString(), attr: { value: y.toString() } });
    });
    let isUpdating = false;
    function debounceUpdate(callback) {
        return function (...args) {
            if (!isUpdating) {
                isUpdating = true;
                setTimeout(() => {
                    callback(...args);
                    isUpdating = false;
                }, 100);
            }
        };
    }
    yearSelect.addEventListener('change', debounceUpdate(async () => {
        currentYear = yearSelect.value;
        data = await getPortfolioData(); // Reload data
        trades = [];
        try {
            if (typeof dv !== 'undefined' && dv.pages) {
                const rawPages = dv.pages('"Trades/Trade"') || [];
                trades = Array.from(rawPages).filter(p => p && p.file && !p.file.name.includes("Template") && p.exit_date);
            }
        } catch (error) {
            console.error('Error loading trades:', error);
            trades = [];
        }
        computed = currentYear === 'all' ? computeStats(data, trades) : computeYearStats(parseInt(currentYear), data, trades);
        updateHeader();
        renderStats();
        updateChartSafely();
    }));
    const timeframeDiv = mainDiv.createEl('div', { cls: 'timeframe-buttons' });
    const timeframes = ['daily', 'monthly', 'yearly'];
    const timeframeButtons = {};
    timeframes.forEach(tf => {
        const btn = timeframeDiv.createEl('button', {
            text: tf.charAt(0).toUpperCase() + tf.slice(1),
            cls: `timeframe-btn ${tf === currentTimeframe ? 'active' : ''}`
        });
        timeframeButtons[tf] = btn;
        btn.addEventListener('click', debounceUpdate(() => {
            timeframes.forEach(t => {
                timeframeButtons[t].classList.toggle('active', t === tf);
            });
            currentTimeframe = tf;
            updateChartSafely();
        }));
    });
    const chartTypeDiv = mainDiv.createEl('div', { cls: 'chart-type-buttons' });
    const chartTypes = ['balance', 'roi'];
    const chartTypeButtons = {};
    chartTypes.forEach(ct => {
        let btnText = ct.charAt(0).toUpperCase() + ct.slice(1);
        if (ct === 'roi') {
            btnText = 'ROI';
        }
        const btn = chartTypeDiv.createEl('button', {
            text: btnText,
            cls: `chart-type-btn ${ct === currentChartType ? 'active' : ''}`
        });
        chartTypeButtons[ct] = btn;
        btn.addEventListener('click', debounceUpdate(() => {
            chartTypes.forEach(t => {
                chartTypeButtons[t].classList.toggle('active', t === ct);
            });
            currentChartType = ct;
            updateChartSafely();
        }));
    });
    const chartDiv = mainDiv.createEl('div', { cls: 'chart-container' });
    const statsDiv = mainDiv.createEl('div', { cls: 'section-container' });
    statsDiv.createEl('h3', { text: 'Portfolio Summary', attr: { style: 'color: #fff; margin-bottom: 15px;' } });
    const statsTable = statsDiv.createEl('table', { cls: 'stats-table' });
    const movementsDiv = mainDiv.createEl('div', { cls: 'section-container' });
    if (data.movements.length > 0) {
        movementsDiv.createEl('h3', { text: 'Recent Movements', attr: { style: 'color: #fff; margin-bottom: 0; margin-top: 0;' } });
        const table = movementsDiv.createEl('table', { cls: 'movements-table' });
        const thead = table.createEl('thead');
        const headerRow = thead.createEl('tr');
        ['Date', 'Type', 'Amount', 'Balance After', 'Description'].forEach(header => {
            headerRow.createEl('th', { text: header });
        });
        const tbody = table.createEl('tbody');
        [...data.movements].reverse().slice(0, 10).forEach(movement => {
            const row = tbody.createEl('tr');
            row.createEl('td', { text: movement.date });
            row.createEl('td', {
                text: movement.type,
                attr: {
                    style: `color: ${movement.type === 'Deposit' ? '#28a745' : (movement.type === 'Withdrawal' ? '#dc3545' : '#E65100')};`
                }
            });
            row.createEl('td', { text: '$' + movement.amount.toFixed(2) });
            row.createEl('td', { text: '$' + movement.balance.toFixed(2) });
            row.createEl('td', { text: movement.description });
        });
        if (data.movements.length > 10) {
            movementsDiv.createEl('p', {
                text: '... and ' + (data.movements.length - 10) + ' more movements',
                attr: { style: 'color: #888; margin-top: 10px; font-style: italic;' }
            });
        }
    }
    function renderStats() {
        statsTable.innerHTML = '';
        let stats;
        try {
            stats = computed;
            if (!stats || Object.keys(stats).length === 0) {
                statsTable.createEl('tr').createEl('td', { text: 'No data available for this year', attr: { colspan: 2, style: 'padding: 8px; color: #ccc; text-align: center;' } });
                return;
            }
        } catch (error) {
            console.error('Error computing stats:', error);
            statsTable.createEl('tr').createEl('td', { text: 'Error loading stats for this year', attr: { colspan: 2, style: 'padding: 8px; color: #ccc; text-align: center;' } });
            return;
        }
        const statsData = [
            ['Initial Balance', '$' + stats.initialBalance.toFixed(2)],
            ['Final Balance', '$' + (stats.currentBalance !== undefined ? stats.currentBalance.toFixed(2) : 'N/A')],
            ['Total Deposits', '$' + stats.depositsTotal.toFixed(2)],
            ['Total Withdrawals', '$' + stats.withdrawalsTotal.toFixed(2)],
            ['Trading P&L', '$' + stats.totalTradePnL.toFixed(2)],
            ['Investing P&L', '$' + stats.totalManualPnl.toFixed(2)],
            ['Max Drawdown', (stats.maxDrawdown * 100).toFixed(2) + '%'],
            ['Sharpe Ratio', stats.sharpe.toFixed(2)],
            ['Sortino Ratio', stats.sortino.toFixed(2)],
            ['ROI', stats.roi.toFixed(2) + '%'],
            ['PnL', (stats.totalPnL >= 0 ? '+' : '') + '$' + stats.totalPnL.toFixed(2)]
        ];
        statsData.forEach(([label, value]) => {
            const row = statsTable.createEl('tr');
            row.createEl('td', { text: label, attr: { style: 'padding: 8px; color: #ccc; border-bottom: 1px solid #444;' } });
            let colorStyle = 'color: #fff;';
            if (label === 'PnL') {
                colorStyle = stats.totalPnL >= 0 ? 'color: #28a745;' : 'color: #dc3545;';
            } else if (label === 'ROI') {
                colorStyle = stats.roi >= 0 ? 'color: #28a745;' : 'color: #dc3545;';
            } else if (label === 'Sharpe Ratio') {
                colorStyle = stats.sharpe >= 0 ? 'color: #28a745;' : 'color: #dc3545;';
            } else if (label === 'Sortino Ratio') {
                colorStyle = stats.sortino >= 0 ? 'color: #28a745;' : 'color: #dc3545;';
            }
            row.createEl('td', {
                text: value,
                attr: {
                    style: 'padding: 8px; text-align: right; border-bottom: 1px solid #444; ' + colorStyle
                }
            });
        });
    }
    function updateChartSafely() {
        try {
            chartDiv.innerHTML = '';
            const initialBalance = computed.initialBalance;
            let filteredEvents = [...data.movements];
            if (currentYear !== 'all') {
                const selectedYear = parseInt(currentYear);
                filteredEvents = data.movements.filter(m => moment(m.date, 'DD-MM-YYYY', true).isValid() && moment(m.date, 'DD-MM-YYYY').year() === selectedYear);
                trades.forEach(trade => {
                    const tradeDate = moment.unix(Math.floor(trade.exit_date / 1000));
                    if (tradeDate.isValid() && tradeDate.year() === selectedYear) {
                        filteredEvents.push({
                            date: tradeDate.format('DD-MM-YYYY'),
                            type: 'Trade P&L',
                            amount: Number(trade.PnL) || 0,
                            description: 'Closed trade on ' + (trade.Symbol || 'Unknown')
                        });
                    }
                });
            } else {
                trades.forEach(trade => {
                    const tradeDate = moment.unix(Math.floor(trade.exit_date / 1000));
                    if (tradeDate.isValid()) {
                        filteredEvents.push({
                            date: tradeDate.format('DD-MM-YYYY'),
                            type: 'Trade P&L',
                            amount: Number(trade.PnL) || 0,
                            description: 'Closed trade on ' + (trade.Symbol || 'Unknown')
                        });
                    }
                });
            }
            filteredEvents.sort((a, b) => moment(a.date, 'DD-MM-YYYY').diff(moment(b.date, 'DD-MM-YYYY')));
            const aggregatedData = {};
            let currentCumPnL = 0;
            let currentCumInvestment = initialBalance;
            let currentBalance = initialBalance;
            let dataField = 'balance';
            let chartLabel = 'Portfolio Balance ($)';
            let yTitle = 'Balance ($)';
            let yCallback = (value) => '$' + value.toFixed(2);
            let beginAtZero = false;
            let tooltipLabelFunc = (context) => 'Balance: $' + context.parsed.y.toFixed(2);
            if (currentChartType === 'roi') {
                dataField = 'roi';
                chartLabel = 'Cumulative ROI (%)';
                yTitle = 'ROI (%)';
                yCallback = (value) => value.toFixed(2) + '%';
                beginAtZero = true;
                tooltipLabelFunc = (context) => 'ROI: ' + context.parsed.y.toFixed(2) + '%';
                currentCumPnL = 0;
                currentCumInvestment = initialBalance;
            }
            filteredEvents.forEach(event => {
                if (!moment(event.date, 'DD-MM-YYYY', true).isValid()) {
                    console.warn('Invalid date format for event: ' + event.date);
                    return;
                }
                let key;
                if (currentTimeframe === 'daily') {
                    key = event.date;
                } else if (currentTimeframe === 'monthly') {
                    key = moment(event.date, 'DD-MM-YYYY').format('MMM YYYY');
                } else {
                    key = moment(event.date, 'DD-MM-YYYY').format('YYYY');
                }
                if (!aggregatedData[key]) {
                    aggregatedData[key] = {
                        date: key
                    };
                    if (currentChartType === 'balance') {
                        aggregatedData[key][dataField] = currentBalance;
                    } else {
                        let roi = currentCumInvestment > 0 ? (currentCumPnL / currentCumInvestment) * 100 : 0;
                        aggregatedData[key][dataField] = roi;
                    }
                }
                const amountToAdd = (event.type === 'Withdrawal') ? -event.amount : event.amount;
                let pnlToAdd = 0;
                let investToAdd = 0;
                if (event.type === 'Trade P&L' || event.type === 'Investment closed') {
                    pnlToAdd = event.amount;
                } else if (event.type === 'Deposit') {
                    investToAdd = event.amount;
                } else if (event.type === 'Withdrawal') {
                    investToAdd = -event.amount;
                }
                if (currentChartType === 'balance') {
                    currentBalance += amountToAdd;
                    aggregatedData[key][dataField] = currentBalance;
                } else { // roi
                    currentCumPnL += pnlToAdd;
                    currentCumInvestment += investToAdd;
                    let roi = currentCumInvestment > 0 ? (currentCumPnL / currentCumInvestment) * 100 : 0;
                    aggregatedData[key][dataField] = roi;
                }
            });
            let chartData = Object.values(aggregatedData);
            if (chartData.length === 0) {
                let emptyDateFormat = currentTimeframe === 'daily' ? 'DD-MM-YYYY' : currentTimeframe === 'monthly' ? 'MMM YYYY' : 'YYYY';
                const emptyValue = currentChartType === 'balance' ? initialBalance : 0;
                chartData.push({
                    date: moment().format(emptyDateFormat),
                    [dataField]: emptyValue
                });
            }
            if (window.renderChart) {
                const tooltipLabel = (context) => tooltipLabelFunc(context);
                const chartConfig = {
                    type: 'line',
                    data: {
                        labels: chartData.map(d => d.date),
                        datasets: [{
                            label: chartLabel,
                            data: chartData.map(d => d[dataField]),
                            backgroundColor: 'rgba(54, 162, 235, 0.1)',
                            borderColor: 'rgba(54, 162, 235, 1)',
                            borderWidth: 2,
                            fill: true,
                            tension: 0.1
                        }]
                    },
                    options: {
                        responsive: true,
                        maintainAspectRatio: false,
                        scales: {
                            y: {
                                beginAtZero: beginAtZero,
                                title: { display: true, text: yTitle },
                                ticks: {
                                    callback: yCallback
                                }
                            },
                            x: {
                                title: { display: true, text: 'Date' },
                                ticks: {
                                    callback: function(value, index, values) {
                                        const label = this.getLabelForValue(value);
                                        let formattedLabel = label;
                                        if (currentTimeframe === 'daily') {
                                            formattedLabel = moment(label, 'DD-MM-YYYY').format('MMM DD, YYYY');
                                        } else if (currentTimeframe === 'monthly') {
                                            formattedLabel = moment(label, 'MMM YYYY').format('MMM YYYY');
                                        } else {
                                            formattedLabel = moment(label, 'YYYY').format('YYYY');
                                        }
                                        return formattedLabel;
                                    }
                                }
                            }
                        },
                        plugins: {
                            tooltip: {
                                callbacks: {
                                    label: tooltipLabel
                                }
                            }
                        }
                    }
                };
                window.renderChart(chartConfig, chartDiv);
            } else {
                chartDiv.createEl('p', {
                    text: "Charts plugin's renderChart function not found. Please ensure the Charts plugin is installed and enabled.",
                    attr: { style: 'color: #ccc; text-align: center; padding: 50px;' }
                });
            }
        } catch (error) {
            console.error('Error updating chart:', error);
            chartDiv.innerHTML = 'Error updating chart: ' + error.message;
        }
    }
    renderStats();
    updateChartSafely();
}

// Initial setup
loadData().then(() => {
    renderPortfolio();
}).catch(error => {
    console.error('Error rendering portfolio:', error);
    mainDiv.innerHTML = 'Error loading portfolio: ' + error.message;
});
```