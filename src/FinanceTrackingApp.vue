<template>
    <div class="app">
        <header class="header">
            <div class="header-inner">
                <h1> Finance Tracker</h1>
                <p class="subtitle">Personal Budget</p>
            </div>
        </header>

        <main class="main">
            <section class="cards">
                <div class="card card--balance" :class="{ negative: balance < 0 }">
                    <span class="card-label">Balance</span>
                    <span class="card-value">${{ balance.toFixed(2) }}</span>
                </div>
                <div class="card card--income">
                    <span class="card-label">Total Income</span>
                    <span class="card-value">${{ totalIncome.toFixed(2) }}</span>
                </div>
                <div class="card card--expense">
                    <span class="card-label">Total Expenses</span>
                    <span class="card-value">${{ totalExpense.toFixed(2) }}</span>
                </div>
            </section>

            <section class="budget-section">
                <div class="budget-header">
                    <span class="budget-label">Budget limit</span>
                    <div class="budget-limit-edit">
                        <span class="budget-status-badge" :class="budgetStatusClass">{{ budgetStatus }}</span>
                        <input v-model.number="budgetLimit" class="budget-input" min="0" placeholder="Set limit"
                            type="number" />
                    </div>
                </div>
                <div class="progress-track">
                    <div class="progress-fill" :style="{
                        width: expensePercentage + '%',
                        background: isOverBudget ? '#e53e3e' : expensePercentage > 75 ? '#d97706' : '#38a189'
                    }"></div>
                </div>
                <div class="progress-labels">
                    <span>${{ totalExpense.toFixed(2) }} spent</span>
                    <span>{{ expensePercentage.toFixed(0) }} % of ${{ budgetLimit }}</span>
                </div>
            </section>

            <section class="form-section">
                <h2>Add Transaction</h2>
                <div class="form-row">
                    <input v-model="form.desc" type="text" placeholder="Description (e.g. Groceries)" class="input" />
                    <input v-model.number="form.amount" type="number" placeholder="Amount" class="input input--amount"
                        min="0" />
                    <select v-model="form.type" class="input input--select">
                        <option value="income">Income</option>
                        <option value="expense">Expense</option>
                    </select>
                    <button @click="addTransaction" class="btn btn--add">Add</button>
                </div>
                <p v-if="formError" class="form-error">{{ formError }}</p>
            </section>

            <section class="list-section">
                <div class="list-header">
                    <h2>Transactions</h2>
                    <div class="filters">
                        <button v-for="f in ['all', 'income', 'expense']" :key="f" class="filter-btn"
                            :class="{ active: filterType === f }" @click="filterType = f">{{ f }}</button>
                    </div>
                    <button v-if="transactions.length" @click="clearAll" class="btn btn--clear">Clear all</button>
                </div>

                <div v-if="filteredTransactions.length === 0" class="empty">
                    No transactions yet. Add one above!
                </div>

                <ul class="tx-list">
                    <li v-for="tx in filteredTransactions" :key="tx.id" class="tx-item" :class="tx.type">
                        <div class="tx-info">
                            <span class="tx-desc">{{ tx.desc }}</span>
                            <span class="tx-date">{{ tx.date }}</span>
                        </div>
                        <div class="tx-right">
                            <span class="tx-amount">
                                {{ tx.type === 'income' ? '+' : '-' }}${{ tx.amount.toFixed(2) }}
                            </span>
                            <button @click="deleteTransaction(tx.id)" class="btn-delete">✕</button>
                        </div>
                    </li>
                </ul>
            </section>

            <!-- ── CATEGORY SUMMARY ── -->
            <section class="category-section">
                <h2>Category Summary</h2>
                <div class="category-grid">
                    <div v-for="(total, cat) in categorySummary" :key="cat" class="category-card" :class="cat">
                        <span class="cat-label">{{ cat }}</span>
                        <span class="cat-value">${{ total.toFixed(2) }}</span>
                    </div>
                </div>
            </section>

            <!-- ── NOTIFICATION LOG ── -->
            <section v-if="notificationLog.length" class="log-section">
                <h2>Notification Log</h2>
                <ul class="log-list">
                    <li v-for="(msg, i) in notificationLog" :key="i" class="log-item">{{ msg }}</li>
                </ul>
            </section>
        </main>

    </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue'
const transactions = ref([])
const filterType = ref('all')
const budgetLimit = ref(1000)

const form = ref({ desc: '', amount: '', type: 'expense' })
const formError = ref('')
const notificationLog = ref([])

// Methods

function addTransaction() {
    formError.value = ''
    if (!form.value.desc.trim()) { formError.value = 'Please enter the description.'; return }
    if (!form.value.amount || form.value.amount <= 0) { formError.value = "Please enter a valid amount."; return }

    transactions.value.push({
        id: Date.now(),
        desc: form.value.desc.trim(),
        amount: Number(form.value.amount),
        type: form.value.type,
        date: new Date().toLocaleDateString()
    })
    form.value.desc = ''
    form.value.amount = ''
}

function deleteTransaction(id) {
    transactions.value = transactions.value.filter(t => t.id !== id)
}

function clearAll() {
    if (confirm('Clear all transactions?')) transactions.value = []
}

// Computed

const filteredTransactions = computed(() => {
    return filterType.value === 'all'
        ? transactions.value
        : transactions.value.filter(t => t.type === filterType.value)
})

const totalIncome = computed(() =>
    transactions.value
        .filter(t => t.type === 'income')
        .reduce((sum, t) => sum + t.amount, 0)
)

const totalExpense = computed(() =>
    transactions.value
        .filter(t => t.type === 'expense')
        .reduce((sum, t) => sum + t.amount, 0)
)

const balance = computed(() => totalIncome.value - totalExpense.value)

const isOverBudget = computed(() => totalExpense.value > budgetLimit.value)

const expensePercentage = computed(() =>
    budgetLimit.value > 0
        ? Math.min((totalExpense.value / budgetLimit.value) * 100, 100)
        : 0
)

const categorySummary = computed(() => {
    const s = {}
    transactions.value.forEach(t => {
        s[t.type] = (s[t.type] || 0) + t.amount
    })
    return s
})

const budgetStatus = computed(() => {
    if (isOverBudget.value) return 'Over budget!'
    if (expensePercentage.value > 75) return 'Approaching limit'
    return 'Within budget'
})

const budgetStatusClass = computed(() => {
    if (isOverBudget.value) return 'status--danger'
    if (expensePercentage.value > 75) return 'status--warning'
    return 'status--okay'
})

// watcher

watch(transactions, (newVal) => {
    localStorage.setItem('fit_transactions', JSON.stringify(newVal))
}, { deep: true, flush: 'post' })

watch(balance, (val) => {
    if (val < 0) {
        notificationLog.value.push(`Balance went negative ($${val.toFixed(2)}) at ${new Date().toLocaleTimeString()}`)
    }
})

watch(isOverBudget, (val) => {
    if (val) {
        notificationLog.value.push(`Over budget! Expenses hit $${totalExpense.value.toFixed(2)} at ${new Date().toLocaleTimeString()}`)
    }
})

// load from localstorage on mount
onMounted(() => {
    const saved = localStorage.getItem('fit_transactions')
    if (saved) transactions.value = JSON.parse(saved)
})

</script>

<style scoped>
* { box-sizing: border-box; margin: 0; padding: 0; }

.app {
  min-height: 100vh;
  background: #f0f4f8;
  font-family: 'Segoe UI', system-ui, sans-serif;
  color: #1a202c;
}

/* HEADER */
.header {
  background: #1a365d;
  color: white;
  padding: 1.5rem 2rem;
}
.header h1 { font-size: 1.6rem; font-weight: 700; }
.subtitle { font-size: 0.85rem; opacity: 0.65; margin-top: 4px; }

/* MAIN */
.main {
  max-width: 820px;
  margin: 0 auto;
  padding: 2rem 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 1.75rem;
}

h2 { font-size: 1rem; font-weight: 600; color: #2d3748; margin-bottom: 0.75rem; }

/* SUMMARY CARDS */
.cards { display: grid; grid-template-columns: repeat(3, 1fr); gap: 1rem; }
.card {
  background: white;
  border-radius: 12px;
  padding: 1.25rem;
  display: flex;
  flex-direction: column;
  gap: 6px;
  border-left: 4px solid #cbd5e0;
}
.card--balance { border-left-color: #3182ce; }
.card--balance.negative { border-left-color: #e53e3e; }
.card--income  { border-left-color: #38a169; }
.card--expense { border-left-color: #e53e3e; }
.card-label { font-size: 0.75rem; text-transform: uppercase; letter-spacing: 0.05em; color: #718096; }
.card-value { font-size: 1.5rem; font-weight: 700; color: #1a202c; }
.card--balance.negative .card-value { color: #e53e3e; }

/* BUDGET */
.budget-section { background: white; border-radius: 12px; padding: 1.25rem; }
.budget-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 0.75rem; }
.budget-label { font-size: 0.9rem; font-weight: 600; color: #2d3748; }
.budget-limit-edit { display: flex; align-items: center; gap: 0.5rem; }
.budget-input {
  width: 110px;
  padding: 5px 10px;
  border: 1px solid #e2e8f0;
  border-radius: 6px;
  font-size: 0.85rem;
}
.budget-status-badge {
  font-size: 0.8rem;
  padding: 3px 10px;
  border-radius: 20px;
  font-weight: 500;
}
.status--ok      { background: #f0fff4; color: #276749; }
.status--warning { background: #fffbeb; color: #92400e; }
.status--danger  { background: #fff5f5; color: #c53030; }
.progress-track {
  height: 10px;
  background: #edf2f7;
  border-radius: 99px;
  overflow: hidden;
}
.progress-fill {
  height: 100%;
  border-radius: 99px;
  transition: width 0.4s ease, background 0.4s ease;
}
.progress-labels {
  display: flex;
  justify-content: space-between;
  font-size: 0.75rem;
  color: #718096;
  margin-top: 6px;
}

/* FORM */
.form-section { background: white; border-radius: 12px; padding: 1.25rem; }
.form-row { display: flex; gap: 0.5rem; flex-wrap: wrap; }
.input {
  flex: 1;
  min-width: 140px;
  padding: 9px 12px;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  font-size: 0.9rem;
  outline: none;
  transition: border-color 0.15s;
}
.input:focus { border-color: #3182ce; }
.input--amount { max-width: 130px; }
.input--select { max-width: 130px; cursor: pointer; }
.btn {
  padding: 9px 18px;
  border: none;
  border-radius: 8px;
  font-size: 0.9rem;
  font-weight: 600;
  cursor: pointer;
  transition: opacity 0.15s;
}
.btn:hover { opacity: 0.85; }
.btn--add   { background: #3182ce; color: white; }
.btn--clear { background: #fed7d7; color: #c53030; font-size: 0.8rem; padding: 6px 14px; }
.form-error { margin-top: 8px; font-size: 0.82rem; color: #e53e3e; }

/* LIST */
.list-section { background: white; border-radius: 12px; padding: 1.25rem; }
.list-header { display: flex; align-items: center; gap: 0.75rem; flex-wrap: wrap; margin-bottom: 0.75rem; }
.list-header h2 { margin: 0; }
.filters { display: flex; gap: 4px; margin-left: auto; }
.filter-btn {
  padding: 4px 12px;
  border: 1px solid #e2e8f0;
  border-radius: 20px;
  background: white;
  font-size: 0.8rem;
  cursor: pointer;
  text-transform: capitalize;
  transition: all 0.15s;
}
.filter-btn.active { background: #1a365d; color: white; border-color: #1a365d; }
.empty { text-align: center; padding: 2rem; color: #a0aec0; font-size: 0.9rem; }
.tx-list { list-style: none; display: flex; flex-direction: column; gap: 6px; }
.tx-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 14px;
  border-radius: 8px;
  border-left: 3px solid #e2e8f0;
  background: #f7fafc;
}
.tx-item.income  { border-left-color: #38a169; }
.tx-item.expense { border-left-color: #e53e3e; }
.tx-info { display: flex; flex-direction: column; gap: 2px; }
.tx-desc { font-size: 0.9rem; font-weight: 500; }
.tx-date { font-size: 0.75rem; color: #a0aec0; }
.tx-right { display: flex; align-items: center; gap: 12px; }
.tx-amount { font-weight: 700; font-size: 0.95rem; }
.income  .tx-amount { color: #38a169; }
.expense .tx-amount { color: #e53e3e; }
.btn-delete {
  background: none;
  border: none;
  color: #cbd5e0;
  cursor: pointer;
  font-size: 0.8rem;
  padding: 2px 6px;
  border-radius: 4px;
  transition: color 0.15s, background 0.15s;
}
.btn-delete:hover { color: #e53e3e; background: #fff5f5; }

/* CATEGORY SUMMARY */
.category-section { background: white; border-radius: 12px; padding: 1.25rem; }
.category-grid { display: flex; gap: 1rem; }
.category-card {
  flex: 1;
  padding: 1rem;
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  gap: 4px;
}
.category-card.income  { background: #f0fff4; }
.category-card.expense { background: #fff5f5; }
.cat-label { font-size: 0.75rem; text-transform: capitalize; font-weight: 600; color: #718096; }
.cat-value { font-size: 1.3rem; font-weight: 700; }
.income  .cat-value { color: #276749; }
.expense .cat-value { color: #c53030; }

/* LOG */
.log-section { background: white; border-radius: 12px; padding: 1.25rem; }
.log-list { list-style: none; display: flex; flex-direction: column; gap: 6px; }
.log-item {
  font-size: 0.82rem;
  padding: 8px 12px;
  background: #fffbeb;
  border-radius: 6px;
  border-left: 3px solid #d97706;
  color: #744210;
}

@media (max-width: 600px) {
  .cards { grid-template-columns: 1fr; }
  .form-row { flex-direction: column; }
  .input--amount, .input--select { max-width: 100%; }
}
</style>

