<script>
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';
  import { Plus, Download, TrendingUp, DollarSign, Wallet, Activity, Search, Filter, Calendar, Eye, Trash2, X, Check, AlertCircle } from 'lucide-svelte';

  // Stores for state management
  const transactions = writable([]);
  const notifications = writable([]);
  const metrics = writable({
    totalCash: 0,
    avgTransaction: 0,
    count: 0,
    cashInHand: 0,
  });
  const isLoading = writable(true);

  // UI state
  let currentPeriod = 'Today';
  let currentTime = new Date().toLocaleTimeString();
  let searchTerm = '';
  let typeFilter = 'All';
  let showNewTransactionModal = false;
  let showTransactionDetailsModal = false;
  let selectedTransaction = null;
  let isSubmitting = false;

  // Form state
  let form = {
    type: 'Inflow',
    amount: '',
    source: '',
    description: '',
    date: new Date().toISOString().split('T')[0],
  };

  const periodMultiplier = {
    Today: 1,
    'This Week': 7,
    'This Month': 30,
  };

  // Mock API functions
  async function fetchTransactions(period = 'Today') {
    $isLoading = true;
    const count = Math.floor(Math.random() * 10) + 5;
    const multiplier = periodMultiplier[period];
    const sources = ['Customer Payment', 'Cash Deposit', 'Refund', 'Sales', 'Miscellaneous'];
    const denominations = ['Rs. 50', 'Rs. 100', 'Rs. 500', 'Rs. 1000', 'Rs. 1 Coin', 'Rs. 2 Coin', 'Rs. 5 Coin', 'Rs. 10 Coin'];

    const mock = Array.from({ length: count }, () => {
      const isInflow = Math.random() > 0.5;
      return {
        id: crypto.randomUUID(),
        source: sources[Math.floor(Math.random() * sources.length)],
        type: isInflow ? 'Inflow' : 'Outflow',
        amount: (Math.random() * 5000 + 100).toFixed(2),
        denomination: denominations[Math.floor(Math.random() * denominations.length)],
        description: `Cash ${isInflow ? 'received' : 'disbursed'} for ${sources[Math.floor(Math.random() * sources.length)]}`,
        date: new Date(Date.now() - Math.floor(Math.random() * multiplier) * 86400000).toISOString().split('T')[0],
        status: Math.random() > 0.2 ? 'Completed' : Math.random() > 0.5 ? 'Pending' : 'Processing',
      };
    });

    await new Promise(r => setTimeout(r, 800));
    $transactions = mock;
    updateMetrics(mock);
    $isLoading = false;
  }

  function updateMetrics(data) {
    const completed = data.filter(t => t.status === 'Completed');
    const totalInflows = completed.filter(t => t.type === 'Inflow').reduce((a, b) => a + parseFloat(b.amount), 0);
    const totalOutflows = completed.filter(t => t.type === 'Outflow').reduce((a, b) => a + parseFloat(b.amount), 0);
    const totalCash = totalInflows - totalOutflows;
    $metrics = {
      totalCash: totalCash.toFixed(2),
      avgTransaction: completed.length > 0 ? ((totalInflows + totalOutflows) / completed.length).toFixed(2) : '0.00',
      count: data.length,
      cashInHand: totalCash >= 0 ? totalCash.toFixed(2) : '0.00',
    };
  }

  async function createTransaction() {
    isSubmitting = true;
    await new Promise(r => setTimeout(r, 1500));
    const transaction = {
      ...form,
      id: crypto.randomUUID(),
      status: 'Completed',
      denomination: form.amount >= 1000 ? 'Rs. 1000' : form.amount >= 500 ? 'Rs. 500' : form.amount >= 100 ? 'Rs. 100' : 'Rs. 50',
    };
    $transactions = [transaction, ...$transactions];
    updateMetrics($transactions);
    showNotification('Transaction added successfully!', 'success');
    form = {
      type: 'Inflow',
      amount: '',
      source: '',
      description: '',
      date: new Date().toISOString().split('T')[0],
    };
    isSubmitting = false;
    showNewTransactionModal = false;
  }

  function showNotification(message, type = 'info') {
    const id = crypto.randomUUID();
    $notifications = [...$notifications, { id, message, type }];
    setTimeout(() => {
      $notifications = $notifications.filter(note => note.id !== id);
    }, 5000);
  }

  function exportCSV() {
    const headers = ['Date', 'Source', 'Type', 'Amount', 'Denomination', 'Status', 'Description'];
    const rows = filteredTransactions.map(t => [t.date, t.source, t.type, t.amount, t.denomination, t.status, t.description]);
    const csv = [headers, ...rows].map(r => r.join(',')).join('\n');
    const uri = encodeURI('data:text/csv;charset=utf-8,' + csv);
    const link = document.createElement('a');
    link.setAttribute('href', uri);
    link.setAttribute('download', `cash_drawer_${new Date().toISOString().split('T')[0]}.csv`);
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    showNotification('CSV exported successfully!', 'success');
  }

  function deleteTransaction(id) {
    $transactions = $transactions.filter(t => t.id !== id);
    updateMetrics($transactions);
    showNotification('Transaction deleted successfully!', 'success');
  }

  // Reactive filtered transactions
  $: filteredTransactions = $transactions.filter(transaction => {
    const matchesSearch = transaction.source.toLowerCase().includes(searchTerm.toLowerCase()) ||
                         transaction.denomination.toLowerCase().includes(searchTerm.toLowerCase());
    const matchesType = typeFilter === 'All' || transaction.type === typeFilter;
    return matchesSearch && matchesType;
  });

  // Initialize data and time updates
  onMount(async () => {
    await fetchTransactions();
    const interval = setInterval(() => {
      currentTime = new Date().toLocaleTimeString();
    }, 1000);
    return () => clearInterval(interval);
  });

  // Fetch transactions when period changes
  $: if (currentPeriod) {
    fetchTransactions(currentPeriod);
  }

  function getStatusColor(status) {
    switch (status) {
      case 'Completed': return 'bg-green-600/20 text-green-400 border-green-600/50';
      case 'Pending': return 'bg-yellow-600/20 text-yellow-300 border-yellow-600/50';
      case 'Processing': return 'bg-blue-600/20 text-blue-400 border-blue-600/50';
      default: return 'bg-gray-600/20 text-gray-400 border-gray-600/50';
    }
  }

  function getNotificationColor(type) {
    switch (type) {
      case 'success': return 'bg-green-600/20 text-green-400 border-green-600/50';
      case 'error': return 'bg-red-600/20 text-red-400 border-red-600/50';
      case 'warning': return 'bg-yellow-600/20 text-yellow-300 border-yellow-600/50';
      default: return 'bg-blue-600/20 text-blue-400 border-blue-600/50';
    }
  }
</script>

<svelte:head>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800;900&display=swap" rel="stylesheet">
  <script src="https://cdn.tailwindcss.com"></script>
  <script>
    tailwind.config = {
      theme: {
        extend: {
          fontFamily: {
            'inter': ['Inter', 'sans-serif'],
          },
          animation: {
            'fade-in-up': 'fadeInUp 0.6s ease-out forwards',
            'pulse': 'pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite',
          }
        }
      }
    }
  </script>
</svelte:head>

<div class="min-h-screen bg-gradient-to-br from-black to-gray-950 text-gray-100 p-6 font-inter">
  <!-- Notifications -->
  <div class="fixed top-4 right-4 z-50 space-y-2">
    {#each $notifications as notification (notification.id)}
      <div class="p-4 rounded-xl border backdrop-blur-md shadow-lg transform transition-all duration-300 animate-pulse {getNotificationColor(notification.type)}">
        <div class="flex items-center space-x-2">
          {#if notification.type === 'success'}
            <Check class="w-5 h-5" />
          {:else if notification.type === 'error' || notification.type === 'warning'}
            <AlertCircle class="w-5 h-5" />
          {/if}
          <span class="text-sm font-medium">{notification.message}</span>
        </div>
      </div>
    {/each}
  </div>

  <div class="max-w-7xl mx-auto">
    <!-- Header -->
    <div class="flex flex-col lg:flex-row justify-between items-start lg:items-center mb-8">
      <div>
        <h1 class="text-5xl font-bold mb-2 bg-gradient-to-r from-blue-400 to-purple-400 bg-clip-text text-transparent">
          Cash Drawer Dashboard
        </h1>
        <p class="text-gray-400">Real-time cash drawer monitoring and analytics</p>
        <p class="text-sm text-gray-500 mt-1">Last updated: {currentTime}</p>
      </div>
      <div class="flex items-center space-x-4 mt-4 lg:mt-0">
        <select 
          bind:value={currentPeriod}
          class="bg-black/50 backdrop-blur-md border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 focus:outline-none focus:ring-2 focus:ring-blue-600"
        >
          <option value="Today">Today</option>
          <option value="This Week">This Week</option>
          <option value="This Month">This Month</option>
        </select>
        <button 
          on:click={() => showNewTransactionModal = true}
          class="bg-gradient-to-r from-blue-600 to-purple-600 hover:from-blue-700 hover:to-purple-700 px-6 py-2 rounded-lg font-medium transition-all duration-200 flex items-center space-x-2 shadow-lg"
        >
          <Plus class="w-5 h-5" />
          <span>New Transaction</span>
        </button>
      </div>
    </div>

    <!-- Metrics Grid -->
    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
      <div class="bg-gray-900/80 backdrop-blur-md p-6 rounded-2xl shadow-xl border border-gray-800/50 hover:bg-gray-800/80 transition-all duration-300 animate-fade-in-up">
        <div class="flex items-center justify-between">
          <div>
            <p class="text-sm font-medium text-gray-400 mb-1">Total Cash</p>
            <h2 class="text-3xl font-bold text-green-400">Rs. {$metrics.totalCash}</h2>
          </div>
          <DollarSign class="w-8 h-8 text-green-400" />
        </div>
      </div>
      <div class="bg-gray-900/80 backdrop-blur-md p-6 rounded-2xl shadow-xl border border-gray-800/50 hover:bg-gray-800/80 transition-all duration-300 animate-fade-in-up" style="animation-delay: 0.1s;">
        <div class="flex items-center justify-between">
          <div>
            <p class="text-sm font-medium text-gray-400 mb-1">Average Transaction</p>
            <h2 class="text-3xl font-bold text-blue-400">Rs. {$metrics.avgTransaction}</h2>
          </div>
          <TrendingUp class="w-8 h-8 text-blue-400" />
        </div>
      </div>
      <div class="bg-gray-900/80 backdrop-blur-md p-6 rounded-2xl shadow-xl border border-gray-800/50 hover:bg-gray-800/80 transition-all duration-300 animate-fade-in-up" style="animation-delay: 0.2s;">
        <div class="flex items-center justify-between">
          <div>
            <p class="text-sm font-medium text-gray-400 mb-1">Transactions</p>
            <h2 class="text-3xl font-bold text-purple-400">{$metrics.count}</h2>
          </div>
          <Activity class="w-8 h-8 text-purple-400" />
        </div>
      </div>
      <div class="bg-gray-900/80 backdrop-blur-md p-6 rounded-2xl shadow-xl border border-gray-800/50 hover:bg-gray-800/80 transition-all duration-300 animate-fade-in-up" style="animation-delay: 0.3s;">
        <div class="flex items-center justify-between">
          <div>
            <p class="text-sm font-medium text-gray-400 mb-1">Cash in Hand</p>
            <h2 class="text-3xl font-bold text-orange-400">Rs. {$metrics.cashInHand}</h2>
          </div>
          <Wallet class="w-8 h-8 text-orange-400" />
        </div>
      </div>
    </div>

    <!-- Filters and Actions -->
    <div class="flex flex-col lg:flex-row justify-between items-start lg:items-center gap-4 mb-6">
      <div class="flex flex-col sm:flex-row gap-4 w-full lg:w-auto">
        <div class="relative">
          <Search class="absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400 w-5 h-5" />
          <input
            type="text"
            placeholder="Search transactions..."
            bind:value={searchTerm}
            class="bg-black/50 backdrop-blur-md border border-gray-800/50 rounded-lg pl-10 pr-4 py-2 text-gray-100 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-600 w-full sm:w-64"
          />
        </div>
        <select 
          bind:value={typeFilter}
          class="bg-black/50 backdrop-blur-md border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 focus:outline-none focus:ring-2 focus:ring-blue-600"
        >
          <option value="All">All Types</option>
          <option value="Inflow">Inflow</option>
          <option value="Outflow">Outflow</option>
        </select>
      </div>
      <button 
        on:click={exportCSV}
        class="bg-gray-800 hover:bg-gray-700 border border-gray-800/50 px-4 py-2 rounded-lg font-medium transition-all duration-200 flex items-center space-x-2"
      >
        <Download class="w-5 h-5" />
        <span>Export CSV</span>
      </button>
    </div>

    <!-- Transactions Table -->
    <div class="bg-gray-900/50 backdrop-blur-md rounded-2xl border border-gray-800/50 shadow-xl overflow-hidden">
      {#if $isLoading}
        <div class="flex items-center justify-center p-12">
          <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600"></div>
        </div>
      {:else}
        <div class="overflow-x-auto">
          <table class="min-w-full text-sm">
            <thead class="bg-gray-900/30 border-b border-gray-800/50">
              <tr>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Date</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Source</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Type</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Amount</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Denomination</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Status</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Actions</th>
              </tr>
            </thead>
            <tbody>
              {#each filteredTransactions as transaction, index}
                <tr 
                  class="border-b border-gray-800/50 hover:bg-gray-800/30 transition-all duration-200"
                  style="animation-delay: {index * 0.1}s;"
                >
                  <td class="px-6 py-4 text-gray-400">{transaction.date}</td>
                  <td class="px-6 py-4 font-medium text-gray-100">{transaction.source}</td>
                  <td class="px-6 py-4 text-gray-400">{transaction.type}</td>
                  <td class="px-6 py-4 font-semibold text-green-400">Rs. {transaction.amount}</td>
                  <td class="px-6 py-4 text-gray-400">{transaction.denomination}</td>
                  <td class="px-6 py-4">
                    <span class="px-3 py-1 rounded-full text-xs font-semibold border {getStatusColor(transaction.status)}">
                      {transaction.status}
                    </span>
                  </td>
                  <td class="px-6 py-4">
                    <div class="flex items-center space-x-2">
                      <button 
                        on:click={() => {
                          selectedTransaction = transaction;
                          showTransactionDetailsModal = true;
                        }}
                        class="p-2 hover:bg-gray-800/50 rounded-lg transition-colors"
                        title="View Details"
                      >
                        <Eye class="w-4 h-4 text-blue-400" />
                      </button>
                      <button 
                        on:click={() => deleteTransaction(transaction.id)}
                        class="p-2 hover:bg-gray-800/50 rounded-lg transition-colors"
                        title="Delete"
                      >
                        <Trash2 class="w-4 h-4 text-red-400" />
                      </button>
                    </div>
                  </td>
                </tr>
              {/each}
            </tbody>
          </table>
          {#if filteredTransactions.length === 0}
            <div class="text-center py-12 text-gray-400">
              <Activity class="w-12 h-12 mx-auto mb-4 opacity-50" />
              <p>No transactions found</p>
            </div>
          {/if}
        </div>
      {/if}
    </div>

    <!-- New Transaction Modal -->
    {#if showNewTransactionModal}
      <div class="fixed inset-0 bg-black/60 backdrop-blur-sm flex items-center justify-center z-50 p-4" on:click|self={() => showNewTransactionModal = false}>
        <div class="bg-gray-900/80 backdrop-blur-md rounded-2xl border border-gray-800/50 p-8 w-full max-w-md shadow-2xl">
          <div class="flex justify-between items-center mb-6">
            <h3 class="text-2xl font-bold text-gray-100">New Cash Transaction</h3>
            <button 
              on:click={() => showNewTransactionModal = false}
              class="p-2 hover:bg-gray-800/50 rounded-lg transition-colors"
            >
              <X class="w-6 h-6 text-gray-400" />
            </button>
          </div>
          <div class="space-y-4">
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Type</label>
              <select 
                bind:value={form.type}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 focus:outline-none focus:ring-2 focus:ring-blue-600"
              >
                <option value="Inflow">Inflow</option>
                <option value="Outflow">Outflow</option>
              </select>
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Source</label>
              <input 
                type="text" 
                bind:value={form.source}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-600"
                placeholder="Source of transaction"
                required
              />
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Amount (Rs.)</label>
              <input 
                type="number" 
                bind:value={form.amount}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-600"
                placeholder="0.00"
                required
                min="0"
                step="0.01"
              />
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Description</label>
              <textarea 
                bind:value={form.description}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-600 h-20 resize-none"
                placeholder="Transaction description"
              ></textarea>
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Date</label>
              <input 
                type="date" 
                bind:value={form.date}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 focus:outline-none focus:ring-2 focus:ring-blue-600"
                required
              />
            </div>
            <div class="flex gap-4 pt-4">
              <button 
                on:click={() => showNewTransactionModal = false}
                class="flex-1 bg-gray-800 hover:bg-gray-700 border border-gray-800/50 px-4 py-2 rounded-lg font-medium transition-all duration-200 text-gray-100"
              >
                Cancel
              </button>
              <button 
                on:click={createTransaction}
                disabled={isSubmitting}
                class="flex-1 bg-gradient-to-r from-blue-600 to-purple-600 hover:from-blue-700 hover:to-purple-700 disabled:opacity-50 disabled:cursor-not-allowed px-4 py-2 rounded-lg font-medium transition-all duration-200"
              >
                {isSubmitting ? 'Saving...' : 'Save Transaction'}
              </button>
            </div>
          </div>
        </div>
      </div>
    {/if}

    <!-- Transaction Details Modal -->
    {#if showTransactionDetailsModal && selectedTransaction}
      <div class="fixed inset-0 bg-black/60 backdrop-blur-sm flex items-center justify-center z-50 p-4" on:click|self={() => showTransactionDetailsModal = false}>
        <div class="bg-gray-900/80 backdrop-blur-md rounded-2xl border border-gray-800/50 p-8 w-full max-w-md shadow-2xl">
          <div class="flex justify-between items-center mb-6">
            <h3 class="text-2xl font-bold text-gray-100">Transaction Details</h3>
            <button 
              on:click={() => showTransactionDetailsModal = false}
              class="p-2 hover:bg-gray-800/50 rounded-lg transition-colors"
            >
              <X class="w-6 h-6 text-gray-400" />
            </button>
          </div>
          <div class="space-y-4">
            <div class="flex justify-between items-center py-3 border-b border-gray-800/50">
              <span class="text-gray-400">Source:</span>
              <span class="font-medium text-gray-100">{selectedTransaction.source}</span>
            </div>
            <div class="flex justify-between items-center py-3 border-b border-gray-800/50">
              <span class="text-gray-400">Type:</span>
              <span class="font-medium text-gray-100">{selectedTransaction.type}</span>
            </div>
            <div class="flex justify-between items-center py-3 border-b border-gray-800/50">
              <span class="text-gray-400">Amount:</span>
              <span class="font-semibold text-green-400">Rs. {selectedTransaction.amount}</span>
            </div>
            <div class="flex justify-between items-center py-3 border-b border-gray-800/50">
              <span class="text-gray-400">Denomination:</span>
              <span class="font-medium text-gray-100">{selectedTransaction.denomination}</span>
            </div>
            <div class="flex justify-between items-center py-3 border-b border-gray-800/50">
              <span class="text-gray-400">Date:</span>
              <span class="font-medium text-gray-100">{selectedTransaction.date}</span>
            </div>
            <div class="flex justify-between items-center py-3 border-b border-gray-800/50">
              <span class="text-gray-400">Status:</span>
              <span class="px-3 py-1 rounded-full text-xs font-semibold border {getStatusColor(selectedTransaction.status)}">
                {selectedTransaction.status}
              </span>
            </div>
            <div class="py-3">
              <span class="text-gray-400 block mb-2">Description:</span>
              <p class="text-gray-100 bg-gray-800/50 p-3 rounded-lg">{selectedTransaction.description || 'No description provided'}</p>
            </div>
          </div>
        </div>
      </div>
    {/if}
  </div>
</div>

<style>
  @keyframes fadeInUp {
    from {
      opacity: 0;
      transform: translateY(20px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  @keyframes pulse {
    0%, 100% {
      opacity: 1;
    }
    50% {
      opacity: 0.5;
    }
  }

  :global(html) {
    scroll-behavior: smooth;
  }

  :global(body) {
    font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', sans-serif;
  }

  :global(::-webkit-scrollbar) {
    width: 8px;
  }

  :global(::-webkit-scrollbar-track) {
    background: transparent;
  }

  :global(::-webkit-scrollbar-thumb) {
    background: #374151;
    border-radius: 4px;
  }

  :global(::-webkit-scrollbar-thumb:hover) {
    background: #4b5563;
  }
</style>