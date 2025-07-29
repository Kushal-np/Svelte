<script>
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';
  import { Plus, Download, TrendingUp, DollarSign, Users, Activity, Search, Filter, Calendar, Eye, Trash2, X, Check, AlertCircle } from 'lucide-svelte';

  // API Configuration - Update this when connecting to a real backend
  const BASE_URL = 'http://localhost:3000/api'; // Replace with your backend URL, e.g., 'https://your-backend.com/api'

  // Optional: Add authentication headers if required
  const getAuthHeaders = () => ({
    // 'Authorization': `Bearer ${localStorage.getItem('token') || 'YOUR_TOKEN_HERE'}`, // Uncomment and replace with actual token if needed
    'Content-Type': 'application/json'
  });

  // Stores for state management
  const transactions = writable([]);
  const notifications = writable([]);
  const metrics = writable({
    totalSales: 0,
    avgSale: 0,
    count: 0,
    successRate: '100%',
  });
  const isLoading = writable(true);

  // UI state
  let currentPeriod = 'Today';
  let currentTime = new Date().toLocaleTimeString();
  let searchTerm = '';
  let statusFilter = 'All';
  let showNewSaleModal = false;
  let showTransactionModal = false;
  let selectedTransaction = null;
  let isSubmitting = false;

  // Form state
  let form = {
    type: 'Sale',
    amount: '',
    customer: '',
    product: 'Product A',
    description: '',
    date: new Date().toISOString().split('T')[0]
  };

  const periodMultiplier = {
    Today: 1,
    'This Week': 7,
    'This Month': 30
  };

  /**
   * Fetches transactions for a given period
   * Backend Endpoint: GET /api/transactions
   * Query Parameters:
   *   - period (string): 'Today', 'This Week', or 'This Month'
   * Expected Response (JSON):
   *   - Status: 200 OK
   *   - Body: Array of transaction objects
   *   - Example:
   *     [
   *       {
   *         "id": "uuid-string",
   *         "customer": "Acme Corp",
   *         "product": "Product A",
   *         "amount": "750.50",
   *         "description": "Transaction for Product A",
   *         "date": "2025-07-28",
   *         "status": "Completed",
   *         "type": "Sale"
   *       },
   *       ...
   *     ]
   * Error Handling:
   *   - 400 Bad Request: Invalid period parameter
   *   - 401 Unauthorized: Missing or invalid authentication token
   *   - 500 Internal Server Error: Server-side issue
   * Backend Notes:
   *   - Filter transactions based on the 'period' query parameter
   *   - Return an empty array [] if no transactions are found
   *   - Ensure amount is a string with two decimal places for client-side display
   *   - Generate unique id (e.g., UUID) for each transaction
   *   - Implement rate limiting to prevent abuse
   *   - Validate authentication token if required
   */
  async function fetchTransactions(period = 'Today') {
    $isLoading = true;
    try {
      // Static data for now
      await new Promise(r => setTimeout(r, 800));
      const count = Math.floor(Math.random() * 10) + 5;
      const multiplier = periodMultiplier[period];
      const products = ['Product A', 'Service B', 'Consulting', 'Software License', 'Hardware'];
      const customers = ['Acme Corp', 'TechStart Inc', 'Global Solutions', 'Innovation Labs', 'Future Systems'];

      const mock = [];
      Array.from({ length: count }).forEach(() => {
        mock.push({
          id: crypto.randomUUID(),
          customer: customers[Math.floor(Math.random() * customers.length)],
          product: products[Math.floor(Math.random() * products.length)],
          amount: (Math.random() * 15000 + 500).toFixed(2),
          description: `Transaction for ${products[Math.floor(Math.random() * products.length)]}`,
          date: new Date(Date.now() - Math.floor(Math.random() * multiplier) * 86400000).toISOString().split('T')[0],
          status: Math.random() > 0.2 ? 'Completed' : Math.random() > 0.5 ? 'Pending' : 'Processing',
          type: form.type
        });
      });

      // For backend integration, uncomment the following and comment out static data:
      /*
      const response = await fetch(`${BASE_URL}/transactions?period=${encodeURIComponent(period)}`, {
        method: 'GET',
        headers: getAuthHeaders()
      });
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      const mock = await response.json();
      */

      if (!Array.isArray(mock)) {
        throw new Error('Invalid transaction data format');
      }

      $transactions = mock;
      updateMetrics(mock);
    } catch (error) {
      showNotification(`Failed to fetch transactions: ${error.message}`, 'error');
    } finally {
      $isLoading = false;
    }
  }

  /**
   * Updates metrics based on transaction data
   * Client-side function, no backend endpoint required
   * Notes:
   *   - Calculates totalSales, avgSale, count, and successRate from transaction data
   *   - Filters for completed transactions only to ensure accurate financial metrics
   *   - Updates metrics store with formatted values
   */
  function updateMetrics(data) {
    const completed = [];
    data.forEach(t => {
      if (t.status === 'Completed') {
        completed.push(t);
      }
    });
    let total = 0;
    completed.forEach(t => {
      total += parseFloat(t.amount);
    });
    $metrics = {
      totalSales: total.toFixed(2),
      avgSale: completed.length > 0 ? (total / completed.length).toFixed(2) : '0.00',
      count: data.length,
      successRate: data.length > 0 ? `${Math.round((completed.length / data.length) * 100)}%` : '0%'
    };
  }

  /**
   * Creates a new transaction
   * Backend Endpoint: POST /api/transactions
   * Request Body (JSON):
   *   - type (string): 'Sale', 'Refund', 'Service'
   *   - amount (string): Transaction amount with two decimal places
   *   - customer (string): Customer of the transaction
   *   - product (string): 'Product A', 'Service B', 'Consulting', 'Software License', 'Hardware'
   *   - description (string): Optional description
   *   - date (string): ISO date string (e.g., '2025-07-28')
   *   - Example:
   *     {
   *       "type": "Sale",
   *       "amount": "750.50",
   *       "customer": "Acme Corp",
   *       "product": "Product A",
   *       "description": "Transaction for Product A",
   *       "date": "2025-07-28"
   *     }
   * Expected Response (JSON):
   *   - Status: 201 Created
   *   - Body: Created transaction object
   *   - Example:
   *     {
   *       "id": "uuid-string",
   *       "type": "Sale",
   *       "amount": "750.50",
   *       "customer": "Acme Corp",
   *       "product": "Product A",
   *       "description": "Transaction for Product A",
   *       "date": "2025-07-28",
   *       "status": "Completed"
   *     }
   * Error Handling:
   *   - 400 Bad Request: Missing or invalid fields
   *   - 401 Unauthorized: Missing or invalid authentication token
   *   - 500 Internal Server Error: Server-side issue
   * Backend Notes:
   *   - Validate required fields (type, amount, customer, product, date)
   *   - Ensure amount is a string with two decimal places
   *   - Generate unique id (e.g., UUID)
   *   - Return the full transaction object
   *   - Implement rate limiting to prevent abuse
   *   - Validate authentication token if required
   */
  async function createTransaction() {
    // Client-side validation
    if (!form.customer) {
      showNotification('Customer is required', 'error');
      return;
    }
    if (!form.amount || parseFloat(form.amount) <= 0) {
      showNotification('Amount must be greater than 0', 'error');
      return;
    }
    if (!form.date) {
      showNotification('Date is required', 'error');
      return;
    }

    isSubmitting = true;
    try {
      // Static data for now
      await new Promise(r => setTimeout(r, 1500));
      const transaction = {
        ...form,
        id: crypto.randomUUID(),
        status: 'Completed',
        amount: parseFloat(form.amount).toFixed(2) // Ensure two decimal places
      };

      // For backend integration, uncomment the following and comment out static data:
      /*
      const response = await fetch(`${BASE_URL}/transactions`, {
        method: 'POST',
        headers: getAuthHeaders(),
        body: JSON.stringify({
          type: form.type,
          amount: parseFloat(form.amount).toFixed(2),
          customer: form.customer,
          product: form.product,
          description: form.description,
          date: form.date
        })
      });
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      const transaction = await response.json();
      */

      $transactions = [transaction, ...$transactions];
      updateMetrics($transactions);
      showNotification('Transaction saved successfully!', 'success');
      form = {
        type: 'Sale',
        amount: '',
        customer: '',
        product: 'Product A',
        description: '',
        date: new Date().toISOString().split('T')[0]
      };
      showNewSaleModal = false;
    } catch (error) {
      showNotification(`Failed to create transaction: ${error.message}`, 'error');
    } finally {
      isSubmitting = false;
    }
  }

  /**
   * Deletes a transaction by ID
   * Backend Endpoint: DELETE /api/transactions/:id
   * URL Parameters:
   *   - id (string): Transaction ID
   * Expected Response (JSON):
   *   - Status: 200 OK
   *   - Body: { success: true }
   *   - Example:
   *     {
   *       "success": true
   *     }
   * Error Handling:
   *   - 404 Not Found: Transaction ID not found
   *   - 401 Unauthorized: Missing or invalid authentication token
   *   - 500 Internal Server Error: Server-side issue
   * Backend Notes:
   *   - Verify the transaction ID exists before deletion
   *   - Return { success: true } on successful deletion
   *   - Implement rate limiting to prevent abuse
   *   - Validate authentication token if required
   */
  async function deleteTransaction(id) {
    try {
      // Static data for now
      await new Promise(r => setTimeout(r, 500));

      // For backend integration, uncomment the following:
      /*
      const response = await fetch(`${BASE_URL}/transactions/${id}`, {
        method: 'DELETE',
        headers: getAuthHeaders()
      });
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      const result = await response.json();
      if (!result.success) {
        throw new Error('Failed to delete transaction');
      }
      */

      $transactions = $transactions.filter(t => t.id !== id);
      updateMetrics($transactions);
      showNotification('Transaction deleted successfully!', 'success');
    } catch (error) {
      showNotification(`Failed to delete transaction: ${error.message}`, 'error');
    }
  }

  /**
   * Exports transactions as CSV
   * Client-side function, but can leverage GET /api/transactions/csv for data
   * Backend Endpoint (optional): GET /api/transactions/csv
   * Query Parameters:
   *   - period (string): 'Today', 'This Week', or 'This Month'
   *   - status (string): 'All', 'Completed', 'Pending', 'Processing'
   *   - search (string): Search term for customer or product
   * Expected Response (optional server-side CSV):
   *   - Status: 200 OK
   *   - Content-Type: text/csv
   *   - Body: CSV content with headers: Date,Customer,Product,Amount,Status,Description
   * Client-side Notes:
   *   - Generates CSV from filteredTransactions using client-side filtering
   *   - Escapes commas in fields to prevent CSV formatting issues
   * Backend Notes:
   *   - Optionally implement server-side CSV generation to reduce client-side processing
   *   - Ensure CSV headers match client-side expectations
   *   - Apply same filtering logic (customer, product, status) server-side if implemented
   *   - Implement rate limiting to prevent abuse
   */
  function exportCSV() {
    try {
      const headers = ['Date', 'Customer', 'Product', 'Amount', 'Status', 'Description'];
      const rows = [];
      filteredTransactions.forEach(t => {
        // Escape commas in fields to prevent CSV formatting issues
        const escapedRow = [
          t.date,
          `"${t.customer.replace(/"/g, '""')}"`,
          t.product,
          t.amount,
          t.status,
          `"${t.description.replace(/"/g, '""')}"`
        ];
        rows.push(escapedRow);
      });

      // For server-side CSV generation, uncomment the following and comment out client-side logic:
      /*
      const response = await fetch(`${BASE_URL}/transactions/csv?period=${encodeURIComponent(currentPeriod)}&status=${encodeURIComponent(statusFilter)}&search=${encodeURIComponent(searchTerm)}`, {
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
      link.setAttribute('download', `transactions_${new Date().toISOString().split('T')[0]}.csv`);
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

  // Reactive filtered transactions
  $: filteredTransactions = $transactions.filter(transaction => {
    const matchesSearch = transaction.customer.toLowerCase().includes(searchTerm.toLowerCase()) ||
                         transaction.product.toLowerCase().includes(searchTerm.toLowerCase());
    const matchesStatus = statusFilter === 'All' || transaction.status === statusFilter;
    return matchesSearch && matchesStatus;
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
          Sales Dashboard
        </h1>
        <p class="text-gray-400">Real-time sales monitoring and analytics</p>
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
          on:click={() => showNewSaleModal = true}
          class="bg-gradient-to-r from-blue-600 to-purple-600 hover:from-blue-700 hover:to-purple-700 px-6 py-2 rounded-lg font-medium transition-all duration-200 flex items-center space-x-2 shadow-lg"
        >
          <Plus class="w-5 h-5" />
          <span>New Sale</span>
        </button>
      </div>
    </div>

    <!-- Metrics Grid -->
    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
      <div class="bg-gray-900/80 backdrop-blur-md p-6 rounded-2xl shadow-xl border border-gray-800/50 hover:bg-gray-800/80 transition-all duration-300 animate-fade-in-up">
        <div class="flex items-center justify-between">
          <div>
            <p class="text-sm font-medium text-gray-400 mb-1">Total Sales</p>
            <h2 class="text-3xl font-bold text-green-400">Rs. {$metrics.totalSales}</h2>
          </div>
          <DollarSign class="w-8 h-8 text-green-400" />
        </div>
      </div>
      <div class="bg-gray-900/80 backdrop-blur-md p-6 rounded-2xl shadow-xl border border-gray-800/50 hover:bg-gray-800/80 transition-all duration-300 animate-fade-in-up" style="animation-delay: 0.1s;">
        <div class="flex items-center justify-between">
          <div>
            <p class="text-sm font-medium text-gray-400 mb-1">Average Sale</p>
            <h2 class="text-3xl font-bold text-blue-400">Rs. {$metrics.avgSale}</h2>
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
            <p class="text-sm font-medium text-gray-400 mb-1">Success Rate</p>
            <h2 class="text-3xl font-bold text-orange-400">{$metrics.successRate}</h2>
          </div>
          <Users class="w-8 h-8 text-orange-400" />
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
            class="bg-black/50 backdrop-blur-md border border-gray-800/50 rounded-lg pl-10 pr-4 py-2 text-gray-100 placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-blue-600 w-full sm:w-64"
          />
        </div>
        <select 
          bind:value={statusFilter}
          class="bg-black/50 backdrop-blur-md border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 focus:outline-none focus:ring-2 focus:ring-blue-600"
        >
          <option value="All">All Status</option>
          <option value="Completed">Completed</option>
          <option value="Pending">Pending</option>
          <option value="Processing">Processing</option>
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
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Customer</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Product</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Amount</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Status</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Actions</th>
              </tr>
            </thead>
            <tbody>
              {#each filteredTransactions as txn, index}
                <tr 
                  class="border-b border-gray-800/50 hover:bg-gray-800/30 transition-all duration-200"
                  style="animation-delay: {index * 0.1}s;"
                >
                  <td class="px-6 py-4 text-gray-400">{txn.date}</td>
                  <td class="px-6 py-4 font-medium text-gray-100">{txn.customer}</td>
                  <td class="px-6 py-4 text-gray-400">{txn.product}</td>
                  <td class="px-6 py-4 font-semibold text-green-400">Rs. {txn.amount}</td>
                  <td class="px-6 py-4">
                    <span class="px-3 py-1 rounded-full text-xs font-semibold border {getStatusColor(txn.status)}">
                      {txn.status}
                    </span>
                  </td>
                  <td class="px-6 py-4">
                    <div class="flex items-center space-x-2">
                      <button 
                        on:click={() => {
                          selectedTransaction = txn;
                          showTransactionModal = true;
                        }}
                        class="p-2 hover:bg-gray-800/50 rounded-lg transition-colors"
                        title="View Details"
                      >
                        <Eye class="w-4 h-4 text-blue-400" />
                      </button>
                      <button 
                        on:click={() => deleteTransaction(txn.id)}
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

    <!-- New Sale Modal -->
    {#if showNewSaleModal}
      <div class="fixed inset-0 bg-black/60 backdrop-blur-sm flex items-center justify-center z-50 p-4" on:click|self={() => showNewSaleModal = false}>
        <div class="bg-gray-900/80 backdrop-blur-md rounded-2xl border border-gray-800/50 p-8 w-full max-w-md shadow-2xl">
          <div class="flex justify-between items-center mb-6">
            <h3 class="text-2xl font-bold text-gray-100">New Transaction</h3>
            <button 
              on:click={() => showNewSaleModal = false}
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
                <option value="Sale">Sale</option>
                <option value="Refund">Refund</option>
                <option value="Service">Service</option>
              </select>
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Customer</label>
              <input 
                type="text" 
                bind:value={form.customer}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-600"
                placeholder="Customer name"
                required
              />
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Product</label>
              <select 
                bind:value={form.product}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 focus:outline-none focus:ring-2 focus:ring-blue-600"
              >
                <option value="Product A">Product A</option>
                <option value="Service B">Service B</option>
                <option value="Consulting">Consulting</option>
                <option value="Software License">Software License</option>
                <option value="Hardware">Hardware</option>
              </select>
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
                on:click={() => showNewSaleModal = false}
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
    {#if showTransactionModal && selectedTransaction}
      <div class="fixed inset-0 bg-black/60 backdrop-blur-sm flex items-center justify-center z-50 p-4" on:click|self={() => showTransactionModal = false}>
        <div class="bg-gray-900/80 backdrop-blur-md rounded-2xl border border-gray-800/50 p-8 w-full max-w-md shadow-2xl">
          <div class="flex justify-between items-center mb-6">
            <h3 class="text-2xl font-bold text-gray-100">Transaction Details</h3>
            <button 
              on:click={() => showTransactionModal = false}
              class="p-2 hover:bg-gray-800/50 rounded-lg transition-colors"
            >
              <X class="w-6 h-6 text-gray-400" />
            </button>
          </div>
          <div class="space-y-4">
            <div class="flex justify-between items-center py-3 border-b border-gray-800/50">
              <span class="text-gray-400">Customer:</span>
              <span class="font-medium text-gray-100">{selectedTransaction.customer}</span>
            </div>
            <div class="flex justify-between items-center py-3 border-b border-gray-800/50">
              <span class="text-gray-400">Product:</span>
              <span class="font-medium text-gray-100">{selectedTransaction.product}</span>
            </div>
            <div class="flex justify-between items-center py-3 border-b border-gray-800/50">
              <span class="text-gray-400">Amount:</span>
              <span class="font-semibold text-green-400">Rs. {selectedTransaction.amount}</span>
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