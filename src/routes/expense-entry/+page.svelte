<script>
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';
  import { Plus, Download, TrendingUp, DollarSign, ShoppingBag, Activity, Search, Filter, Calendar, Eye, Trash2, X, Check, AlertCircle } from 'lucide-svelte';

  // API Configuration - Update this when connecting to a real backend
  const BASE_URL = 'http://localhost:3000/api'; // Replace with your backend URL, e.g., 'https://your-backend.com/api'

  // Optional: Add authentication headers if required
  const getAuthHeaders = () => ({
    // 'Authorization': 'Bearer YOUR_TOKEN_HERE', // Uncomment and replace with actual token if needed
    'Content-Type': 'application/json'
  });

  // Stores for state management
  const expenses = writable([]);
  const notifications = writable([]);
  const metrics = writable({
    totalExpenses: 0,
    avgExpense: 0,
    count: 0,
    budgetUtilization: '0%',
  });
  const isLoading = writable(true);

  // UI state
  let currentPeriod = 'Today';
  let currentTime = new Date().toLocaleTimeString();
  let searchTerm = '';
  let categoryFilter = 'All';
  let showNewExpenseModal = false;
  let showExpenseDetailsModal = false;
  let selectedExpense = null;
  let isSubmitting = false;

  // Form state
  let form = {
    category: 'Operational',
    amount: '',
    recipient: '',
    description: '',
    date: new Date().toISOString().split('T')[0],
    budget: 10000, // Default budget
  };

  const periodMultiplier = {
    Today: 1,
    'This Week': 7,
    'This Month': 30,
  };

  // API functions with static data and backend integration comments
  /**
   * Fetches expenses for a given period
   * Backend Endpoint: GET /api/expenses
   * Query Parameters:
   *   - period (string): 'Today', 'This Week', or 'This Month'
   * Expected Response (JSON):
   *   - Status: 200 OK
   *   - Body: Array of expense objects
   *   - Example:
   *     [
   *       {
   *         "id": "uuid-string",
   *         "recipient": "VendorSync Ltd",
   *         "category": "Operational",
   *         "amount": "1250.50",
   *         "description": "Expense for Operational costs",
   *         "date": "2025-07-28",
   *         "status": "Paid"
   *       },
   *       ...
   *     ]
   * Error Handling:
   *   - 400 Bad Request: Invalid period parameter
   *   - 500 Internal Server Error: Server-side issue
   * Backend Notes:
   *   - Filter expenses based on the 'period' query parameter
   *   - Return an empty array [] if no expenses are found
   *   - Ensure amount is a string with two decimal places for client-side display
   *   - Generate unique id (e.g., UUID) for each expense
   */
  async function fetchExpenses(period = 'Today') {
    $isLoading = true;
    try {
      // Static data for now
      await new Promise(r => setTimeout(r, 800));
      const count = Math.floor(Math.random() * 10) + 5;
      const multiplier = periodMultiplier[period];
      const categories = ['Operational', 'Utilities', 'Supplies', 'Travel', 'Miscellaneous'];
      const recipients = ['VendorSync Ltd', 'Utility Co', 'SupplyChain Inc', 'Travel Agency', 'Miscellaneous Payee'];

      const mock = [];
      Array.from({ length: count }).forEach(() => {
        mock.push({
          id: crypto.randomUUID(),
          recipient: recipients[Math.floor(Math.random() * recipients.length)],
          category: categories[Math.floor(Math.random() * categories.length)],
          amount: (Math.random() * 5000 + 100).toFixed(2),
          description: `Expense for ${categories[Math.floor(Math.random() * categories.length)]}`,
          date: new Date(Date.now() - Math.floor(Math.random() * multiplier) * 86400000).toISOString().split('T')[0],
          status: Math.random() > 0.2 ? 'Paid' : Math.random() > 0.5 ? 'Pending' : 'Processing',
        });
      });

      // For backend integration, uncomment the following and comment out static data:
      /*
      const response = await fetch(`${BASE_URL}/expenses?period=${period}`, {
        method: 'GET',
        headers: getAuthHeaders()
      });
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      const mock = await response.json();
      */

      if (!Array.isArray(mock)) {
        throw new Error('Invalid expense data');
      }

      $expenses = mock;
      updateMetrics(mock);
    } catch (error) {
      showNotification(`Failed to fetch expenses: ${error.message}`, 'error');
    } finally {
      $isLoading = false;
    }
  }

  /**
   * Updates metrics based on expense data
   * Client-side function, no backend endpoint required
   * Notes:
   *   - Calculates totalExpenses, avgExpense, count, and budgetUtilization from expense data
   *   - Filters for paid expenses only to ensure accuracy
   *   - Updates metrics store with formatted values
   */
  function updateMetrics(data) {
    const paid = [];
    data.forEach(t => {
      if (t.status === 'Paid') {
        paid.push(t);
      }
    });
    let total = 0;
    paid.forEach(t => {
      total += parseFloat(t.amount);
    });
    $metrics = {
      totalExpenses: total.toFixed(2),
      avgExpense: paid.length > 0 ? (total / paid.length).toFixed(2) : '0.00',
      count: data.length,
      budgetUtilization: `${Math.min(Math.round((total / form.budget) * 100), 100)}%`,
    };
  }

  /**
   * Creates a new expense
   * Backend Endpoint: POST /api/expenses
   * Request Body (JSON):
   *   - category (string): 'Operational', 'Utilities', 'Supplies', 'Travel', 'Miscellaneous'
   *   - amount (string): Expense amount with two decimal places
   *   - recipient (string): Recipient of the expense
   *   - description (string): Optional description
   *   - date (string): ISO date string (e.g., '2025-07-28')
   *   - budget (number): Budget for utilization calculation
   *   - Example:
   *     {
   *       "category": "Operational",
   *       "amount": "1000.00",
   *       "recipient": "VendorSync Ltd",
   *       "description": "Payment for operational costs",
   *       "date": "2025-07-28",
   *       "budget": 10000
   *     }
   * Expected Response (JSON):
   *   - Status: 201 Created
   *   - Body: Created expense object
   *   - Example:
   *     {
   *       "id": "uuid-string",
   *       "category": "Operational",
   *       "amount": "1000.00",
   *       "recipient": "VendorSync Ltd",
   *       "description": "Payment for operational costs",
   *       "date": "2025-07-28",
   *       "status": "Paid",
   *       "budget": 10000
   *     }
   * Error Handling:
   *   - 400 Bad Request: Missing or invalid fields
   *   - 500 Internal Server Error: Server-side issue
   * Backend Notes:
   *   - Validate required fields (category, amount, recipient, date, budget)
   *   - Ensure amount is a string with two decimal places
   *   - Generate unique id (e.g., UUID)
   *   - Return the full expense object
   */
  async function createExpense() {
    isSubmitting = true;
    try {
      // Static data for now
      await new Promise(r => setTimeout(r, 1500));
      const expense = {
        ...form,
        id: crypto.randomUUID(),
        status: 'Paid',
      };

      // For backend integration, uncomment the following and comment out static data:
      /*
      const response = await fetch(`${BASE_URL}/expenses`, {
        method: 'POST',
        headers: getAuthHeaders(),
        body: JSON.stringify({
          category: form.category,
          amount: parseFloat(form.amount).toFixed(2),
          recipient: form.recipient,
          description: form.description,
          date: form.date,
          budget: parseFloat(form.budget)
        })
      });
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      const expense = await response.json();
      */

      $expenses = [expense, ...$expenses];
      updateMetrics($expenses);
      showNotification('Expense added successfully!', 'success');
      form = {
        category: 'Operational',
        amount: '',
        recipient: '',
        description: '',
        date: new Date().toISOString().split('T')[0],
        budget: form.budget,
      };
      showNewExpenseModal = false;
    } catch (error) {
      showNotification(`Failed to create expense: ${error.message}`, 'error');
    } finally {
      isSubmitting = false;
    }
  }

  /**
   * Deletes an expense by ID
   * Backend Endpoint: DELETE /api/expenses/:id
   * URL Parameters:
   *   - id (string): Expense ID
   * Expected Response (JSON):
   *   - Status: 200 OK
   *   - Body: { success: true }
   *   - Example:
   *     {
   *       "success": true
   *     }
   * Error Handling:
   *   - 404 Not Found: Expense ID not found
   *   - 500 Internal Server Error: Server-side issue
   * Backend Notes:
   *   - Verify the expense ID exists before deletion
   *   - Return { success: true } on successful deletion
   */
  async function deleteExpense(id) {
    try {
      // Static data for now
      await new Promise(r => setTimeout(r, 500));

      // For backend integration, uncomment the following:
      /*
      const response = await fetch(`${BASE_URL}/expenses/${id}`, {
        method: 'DELETE',
        headers: getAuthHeaders()
      });
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      const result = await response.json();
      if (!result.success) {
        throw new Error('Failed to delete expense');
      }
      */

      $expenses = $expenses.filter(t => t.id !== id);
      updateMetrics($expenses);
      showNotification('Expense deleted successfully!', 'success');
    } catch (error) {
      showNotification(`Failed to delete expense: ${error.message}`, 'error');
    }
  }

  /**
   * Exports expenses as CSV
   * Client-side function, but can leverage GET /api/expenses/csv for data
   * Backend Endpoint (optional): GET /api/expenses/csv
   * Query Parameters:
   *   - period (string): 'Today', 'This Week', or 'This Month'
   *   - category (string): 'All', 'Operational', 'Utilities', 'Supplies', 'Travel', 'Miscellaneous'
   *   - search (string): Search term for recipient or category
   * Expected Response (optional server-side CSV):
   *   - Status: 200 OK
   *   - Content-Type: text/csv
   *   - Body: CSV content with headers: Date,Recipient,Category,Amount,Status,Description
   * Client-side Notes:
   *   - Currently generates CSV from filteredExpenses
   *   - Uses client-side filtering based on searchTerm and categoryFilter
   * Backend Notes:
   *   - Optionally implement server-side CSV generation to reduce client-side processing
   *   - Ensure CSV headers match client-side expectations
   *   - Apply same filtering logic (recipient, category) server-side if implemented
   */
  function exportCSV() {
    try {
      const headers = ['Date', 'Recipient', 'Category', 'Amount', 'Status', 'Description'];
      const rows = [];
      filteredExpenses.forEach(t => {
        rows.push([t.date, t.recipient, t.category, t.amount, t.status, t.description]);
      });

      // For server-side CSV generation, uncomment the following and comment out client-side logic:
      /*
      const response = await fetch(`${BASE_URL}/expenses/csv?period=${currentPeriod}&category=${categoryFilter}&search=${encodeURIComponent(searchTerm)}`, {
        method: 'GET',
        headers: getAuthHeaders()
      });
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      const csv = await response.text();
      */

      const csv = [headers, ...rows].map(r => r.join(',')).join('\n');
      const uri = encodeURI('data:text/csv;charset=utf-8,' + csv);
      const link = document.createElement('a');
      link.setAttribute('href', uri);
      link.setAttribute('download', `expenses_${new Date().toISOString().split('T')[0]}.csv`);
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
      showNotification('CSV exported successfully!', 'success');
    } catch (error) {
      showNotification(`Failed to export CSV: ${error.message}`, 'error');
    }
  }

  function showNotification(message, type = 'info') {
    const id = crypto.randomUUID();
    $notifications = [...$notifications, { id, message, type }];
    setTimeout(() => {
      $notifications = $notifications.filter(note => note.id !== id);
    }, 5000);
  }

  // Reactive filtered expenses
  $: filteredExpenses = $expenses.filter(expense => {
    const matchesSearch = expense.recipient.toLowerCase().includes(searchTerm.toLowerCase()) ||
                         expense.category.toLowerCase().includes(searchTerm.toLowerCase());
    const matchesCategory = categoryFilter === 'All' || expense.category === categoryFilter;
    return matchesSearch && matchesCategory;
  });

  // Initialize data and time updates
  onMount(async () => {
    await fetchExpenses();
    const interval = setInterval(() => {
      currentTime = new Date().toLocaleTimeString();
    }, 1000);
    return () => clearInterval(interval);
  });

  // Fetch expenses when period changes
  $: if (currentPeriod) {
    fetchExpenses(currentPeriod);
  }

  function getStatusColor(status) {
    switch (status) {
      case 'Paid': return 'bg-green-600/20 text-green-400 border-green-600/50';
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
          Expense Dashboard
        </h1>
        <p class="text-gray-400">Real-time expense tracking and analytics</p>
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
          on:click={() => showNewExpenseModal = true}
          class="bg-gradient-to-r from-blue-600 to-purple-600 hover:from-blue-700 hover:to-purple-700 px-6 py-2 rounded-lg font-medium transition-all duration-200 flex items-center space-x-2 shadow-lg"
        >
          <Plus class="w-5 h-5" />
          <span>New Expense</span>
        </button>
      </div>
    </div>

    <!-- Metrics Grid -->
    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
      <div class="bg-gray-900/80 backdrop-blur-md p-6 rounded-2xl shadow-xl border border-gray-800/50 hover:bg-gray-800/80 transition-all duration-300 animate-fade-in-up">
        <div class="flex items-center justify-between">
          <div>
            <p class="text-sm font-medium text-gray-400 mb-1">Total Expenses</p>
            <h2 class="text-3xl font-bold text-green-400">Rs. {$metrics.totalExpenses}</h2>
          </div>
          <DollarSign class="w-8 h-8 text-green-400" />
        </div>
      </div>
      <div class="bg-gray-900/80 backdrop-blur-md p-6 rounded-2xl shadow-xl border border-gray-800/50 hover:bg-gray-800/80 transition-all duration-300 animate-fade-in-up" style="animation-delay: 0.1s;">
        <div class="flex items-center justify-between">
          <div>
            <p class="text-sm font-medium text-gray-400 mb-1">Average Expense</p>
            <h2 class="text-3xl font-bold text-blue-400">Rs. {$metrics.avgExpense}</h2>
          </div>
          <TrendingUp class="w-8 h-8 text-blue-400" />
        </div>
      </div>
      <div class="bg-gray-900/80 backdrop-blur-md p-6 rounded-2xl shadow-xl border border-gray-800/50 hover:bg-gray-800/80 transition-all duration-300 animate-fade-in-up" style="animation-delay: 0.2s;">
        <div class="flex items-center justify-between">
          <div>
            <p class="text-sm font-medium text-gray-400 mb-1">Expense Entries</p>
            <h2 class="text-3xl font-bold text-purple-400">{$metrics.count}</h2>
          </div>
          <Activity class="w-8 h-8 text-purple-400" />
        </div>
      </div>
      <div class="bg-gray-900/80 backdrop-blur-md p-6 rounded-2xl shadow-xl border border-gray-800/50 hover:bg-gray-800/80 transition-all duration-300 animate-fade-in-up" style="animation-delay: 0.3s;">
        <div class="flex items-center justify-between">
          <div>
            <p class="text-sm font-medium text-gray-400 mb-1">Budget Utilization</p>
            <h2 class="text-3xl font-bold text-orange-400">{$metrics.budgetUtilization}</h2>
          </div>
          <ShoppingBag class="w-8 h-8 text-orange-400" />
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
            placeholder="Search expenses..."
            bind:value={searchTerm}
            class="bg-black/50 backdrop-blur-md border border-gray-800/50 rounded-lg pl-10 pr-4 py-2 text-gray-100 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-600 w-full sm:w-64"
          />
        </div>
        <select 
          bind:value={categoryFilter}
          class="bg-black/50 backdrop-blur-md border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 focus:outline-none focus:ring-2 focus:ring-blue-600"
        >
          <option value="All">All Categories</option>
          <option value="Operational">Operational</option>
          <option value="Utilities">Utilities</option>
          <option value="Supplies">Supplies</option>
          <option value="Travel">Travel</option>
          <option value="Miscellaneous">Miscellaneous</option>
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

    <!-- Expenses Table -->
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
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Recipient</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Category</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Amount</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Status</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Actions</th>
              </tr>
            </thead>
            <tbody>
              {#each filteredExpenses as expense, index}
                <tr 
                  class="border-b border-gray-800/50 hover:bg-gray-800/30 transition-all duration-200"
                  style="animation-delay: {index * 0.1}s;"
                >
                  <td class="px-6 py-4 text-gray-400">{expense.date}</td>
                  <td class="px-6 py-4 font-medium text-gray-100">{expense.recipient}</td>
                  <td class="px-6 py-4 text-gray-400">{expense.category}</td>
                  <td class="px-6 py-4 font-semibold text-green-400">Rs. {expense.amount}</td>
                  <td class="px-6 py-4">
                    <span class="px-3 py-1 rounded-full text-xs font-semibold border {getStatusColor(expense.status)}">
                      {expense.status}
                    </span>
                  </td>
                  <td class="px-6 py-4">
                    <div class="flex items-center space-x-2">
                      <button 
                        on:click={() => {
                          selectedExpense = expense;
                          showExpenseDetailsModal = true;
                        }}
                        class="p-2 hover:bg-gray-800/50 rounded-lg transition-colors"
                        title="View Details"
                      >
                        <Eye class="w-4 h-4 text-blue-400" />
                      </button>
                      <button 
                        on:click={() => deleteExpense(expense.id)}
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
          {#if filteredExpenses.length === 0}
            <div class="text-center py-12 text-gray-400">
              <Activity class="w-12 h-12 mx-auto mb-4 opacity-50" />
              <p>No expenses found</p>
            </div>
          {/if}
        </div>
      {/if}
    </div>

    <!-- New Expense Modal -->
    {#if showNewExpenseModal}
      <div class="fixed inset-0 bg-black/60 backdrop-blur-sm flex items-center justify-center z-50 p-4" on:click|self={() => showNewExpenseModal = false}>
        <div class="bg-gray-900/80 backdrop-blur-md rounded-2xl border border-gray-800/50 p-8 w-full max-w-md shadow-2xl">
          <div class="flex justify-between items-center mb-6">
            <h3 class="text-2xl font-bold text-gray-100">New Expense Entry</h3>
            <button 
              on:click={() => showNewExpenseModal = false}
              class="p-2 hover:bg-gray-800/50 rounded-lg transition-colors"
            >
              <X class="w-6 h-6 text-gray-400" />
            </button>
          </div>
          <div class="space-y-4">
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Category</label>
              <select 
                bind:value={form.category}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 focus:outline-none focus:ring-2 focus:ring-blue-600"
              >
                <option value="Operational">Operational</option>
                <option value="Utilities">Utilities</option>
                <option value="Supplies">Supplies</option>
                <option value="Travel">Travel</option>
                <option value="Miscellaneous">Miscellaneous</option>
              </select>
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Recipient</label>
              <input 
                type="text" 
                bind:value={form.recipient}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-600"
                placeholder="Recipient name"
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
              <label class="block text-sm font-medium text-gray-400 mb-2">Budget (Rs.)</label>
              <input 
                type="number" 
                bind:value={form.budget}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-600"
                placeholder="10000.00"
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
                placeholder="Expense description"
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
                on:click={() => showNewExpenseModal = false}
                class="flex-1 bg-gray-800 hover:bg-gray-700 border border-gray-800/50 px-4 py-2 rounded-lg font-medium transition-all duration-200 text-gray-100"
              >
                Cancel
              </button>
              <button 
                on:click={createExpense}
                disabled={isSubmitting}
                class="flex-1 bg-gradient-to-r from-blue-600 to-purple-600 hover:from-blue-700 hover:to-purple-700 disabled:opacity-50 disabled:cursor-not-allowed px-4 py-2 rounded-lg font-medium transition-all duration-200"
              >
                {isSubmitting ? 'Saving...' : 'Save Expense'}
              </button>
            </div>
          </div>
        </div>
      </div>
    {/if}

    <!-- Expense Details Modal -->
    {#if showExpenseDetailsModal && selectedExpense}
      <div class="fixed inset-0 bg-black/60 backdrop-blur-sm flex items-center justify-center z-50 p-4" on:click|self={() => showExpenseDetailsModal = false}>
        <div class="bg-gray-900/80 backdrop-blur-md rounded-2xl border border-gray-800/50 p-8 w-full max-w-md shadow-2xl">
          <div class="flex justify-between items-center mb-6">
            <h3 class="text-2xl font-bold text-gray-100">Expense Details</h3>
            <button 
              on:click={() => showExpenseDetailsModal = false}
              class="p-2 hover:bg-gray-800/50 rounded-lg transition-colors"
            >
              <X class="w-6 h-6 text-gray-400" />
            </button>
          </div>
          <div class="space-y-4">
            <div class="flex justify-between items-center py-3 border-b border-gray-800/50">
              <span class="text-gray-400">Recipient:</span>
              <span class="font-medium text-gray-100">{selectedExpense.recipient}</span>
            </div>
            <div class="flex justify-between items-center py-3 border-b border-gray-800/50">
              <span class="text-gray-400">Category:</span>
              <span class="font-medium text-gray-100">{selectedExpense.category}</span>
            </div>
            <div class="flex justify-between items-center py-3 border-b border-gray-800/50">
              <span class="text-gray-400">Amount:</span>
              <span class="font-semibold text-green-400">Rs. {selectedExpense.amount}</span>
            </div>
            <div class="flex justify-between items-center py-3 border-b border-gray-800/50">
              <span class="text-gray-400">Date:</span>
              <span class="font-medium text-gray-100">{selectedExpense.date}</span>
            </div>
            <div class="flex justify-between items-center py-3 border-b border-gray-800/50">
              <span class="text-gray-400">Status:</span>
              <span class="px-3 py-1 rounded-full text-xs font-semibold border {getStatusColor(selectedExpense.status)}">
                {selectedExpense.status}
              </span>
            </div>
            <div class="py-3">
              <span class="text-gray-400 block mb-2">Description:</span>
              <p class="text-gray-100 bg-gray-800/50 p-3 rounded-lg">{selectedExpense.description || 'No description provided'}</p>
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