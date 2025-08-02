<script>
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';
  import { Plus, Download, User, ShoppingCart, Search, Filter, Eye, Trash2, X, Check, AlertCircle } from 'lucide-svelte';

  // API Configuration - Update this when connecting to a real backend
  const BASE_URL = 'http://localhost:3000/api'; // Replace with your backend URL, e.g., 'https://your-backend.com/api'

  // Optional: Add authentication headers if required
  const getAuthHeaders = () => ({
    // 'Authorization': `Bearer ${localStorage.getItem('token') || 'YOUR_TOKEN_HERE'}`, // Uncomment and replace with actual token if needed
    'Content-Type': 'application/json'
  });

  // Stores for state management
  const users = writable([]);
  const sales = writable([]);
  const notifications = writable([]);
  const isLoading = writable(true);

  // UI state
  let currentTime = new Date().toLocaleTimeString();
  let searchTerm = '';
  let showNewUserModal = false;
  let showNewSaleModal = false;
  let showUserDetailsModal = false;
  let selectedUser = null;
  let isSubmitting = false;

  // Form states
  let userForm = {
    name: '',
    role: 'Seller',
    contact: ''
  };
  let saleForm = {
    userId: '',
    product: 'Product A',
    amount: '',
    date: new Date().toISOString().split('T')[0],
    description: ''
  };

  /**
   * Fetches users
   * Backend Endpoint: GET /api/users
   * Expected Response (JSON):
   *   - Status: 200 OK
   *   - Body: Array of user objects
   *   - Example:
   *     [
   *       {
   *         "id": "uuid-string",
   *         "name": "John Doe",
   *         "role": "Seller",
   *         "contact": "john@example.com"
   *       },
   *       ...
   *     ]
   * Error Handling:
   *   - 401 Unauthorized: Missing or invalid authentication token
   *   - 500 Internal Server Error: Server-side issue
   * Backend Notes:
   *   - Return an empty array [] if no users are found
   *   - Generate unique id (e.g., UUID) for each user
   *   - Implement rate limiting to prevent abuse
   *   - Validate authentication token if required
   */
  async function fetchUsers() {
    $isLoading = true;
    try {
      // Static data for now
      await new Promise(r => setTimeout(r, 800));
      const names = ['John Doe', 'Jane Smith', 'Alice Johnson', 'Bob Brown', 'Emma Davis'];
      const roles = ['Seller', 'Manager'];
      const mockUsers = [];
      Array.from({ length: 5 }).forEach(() => {
        mockUsers.push({
          id: crypto.randomUUID(),
          name: names[Math.floor(Math.random() * names.length)],
          role: roles[Math.floor(Math.random() * roles.length)],
          contact: `${names[Math.floor(Math.random() * names.length)].toLowerCase().replace(' ', '')}@example.com`
        });
      });

      // For backend integration, uncomment the following and comment out static data:
      /*
      const response = await fetch(`${BASE_URL}/users`, {
        method: 'GET',
        headers: getAuthHeaders()
      });
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      const mockUsers = await response.json();
      */

      if (!Array.isArray(mockUsers)) {
        throw new Error('Invalid user data format');
      }

      $users = mockUsers;
    } catch (error) {
      showNotification(`Failed to fetch users: ${error.message}`, 'error');
    } finally {
      $isLoading = false;
    }
  }

  /**
   * Fetches sales associated with users
   * Backend Endpoint: GET /api/sales
   * Query Parameters:
   *   - search (string): Search term for product or user name
   * Expected Response (JSON):
   *   - Status: 200 OK
   *   - Body: Array of sale objects
   *   - Example:
   *     [
   *       {
   *         "id": "uuid-string",
   *         "userId": "uuid-string",
   *         "userName": "John Doe",
   *         "product": "Product A",
   *         "amount": "750.50",
   *         "description": "Sale of Product A",
   *         "date": "2025-07-28",
   *         "status": "Completed"
   *       },
   *       ...
   *     ]
   * Error Handling:
   *   - 400 Bad Request: Invalid search parameter
   *   - 401 Unauthorized: Missing or invalid authentication token
   *   - 500 Internal Server Error: Server-side issue
   * Backend Notes:
   *   - Filter sales based on the 'search' query parameter (product or user name)
   *   - Return an empty array [] if no sales are found
   *   - Ensure amount is a string with two decimal places for client-side display
   *   - Include userName for display purposes (join with users table)
   *   - Generate unique id (e.g., UUID) for each sale
   *   - Implement rate limiting to prevent abuse
   *   - Validate authentication token if required
   */
  async function fetchSales() {
    try {
      // Static data for now
      await new Promise(r => setTimeout(r, 800));
      const products = ['Product A', 'Service B', 'Consulting', 'Software License', 'Hardware'];
      const mockSales = [];
      $users.forEach(user => {
        const count = Math.floor(Math.random() * 3) + 1;
        Array.from({ length: count }).forEach(() => {
          mockSales.push({
            id: crypto.randomUUID(),
            userId: user.id,
            userName: user.name,
            product: products[Math.floor(Math.random() * products.length)],
            amount: (Math.random() * 15000 + 500).toFixed(2),
            description: `Sale of ${products[Math.floor(Math.random() * products.length)]}`,
            date: new Date(Date.now() - Math.floor(Math.random() * 30) * 86400000).toISOString().split('T')[0],
            status: Math.random() > 0.2 ? 'Completed' : Math.random() > 0.5 ? 'Pending' : 'Processing'
          });
        });
      });

      // For backend integration, uncomment the following and comment out static data:
      /*
      const response = await fetch(`${BASE_URL}/sales?search=${encodeURIComponent(searchTerm)}`, {
        method: 'GET',
        headers: getAuthHeaders()
      });
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      const mockSales = await response.json();
      */

      if (!Array.isArray(mockSales)) {
        throw new Error('Invalid sales data format');
      }

      $sales = mockSales;
    } catch (error) {
      showNotification(`Failed to fetch sales: ${error.message}`, 'error');
    }
  }

  /**
   * Creates a new user
   * Backend Endpoint: POST /api/users
   * Request Body (JSON):
   *   - name (string): User name
   *   - role (string): 'Seller' or 'Manager'
   *   - contact (string): Optional contact info (e.g., email)
   *   - Example:
   *     {
   *       "name": "John Doe",
   *       "role": "Seller",
   *       "contact": "john@example.com"
   *     }
   * Expected Response (JSON):
   *   - Status: 201 Created
   *   - Body: Created user object
   *   - Example:
   *     {
   *       "id": "uuid-string",
   *       "name": "John Doe",
   *       "role": "Seller",
   *       "contact": "john@example.com"
   *     }
   * Error Handling:
   *   - 400 Bad Request: Missing or invalid fields
   *   - 401 Unauthorized: Missing or invalid authentication token
   *   - 500 Internal Server Error: Server-side issue
   * Backend Notes:
   *   - Validate required fields (name, role)
   *   - Generate unique id (e.g., UUID)
   *   - Return the full user object
   *   - Implement rate limiting to prevent abuse
   *   - Validate authentication token if required
   */
  async function createUser() {
    // Client-side validation
    if (!userForm.name) {
      showNotification('User name is required', 'error');
      return;
    }
    if (!userForm.role) {
      showNotification('Role is required', 'error');
      return;
    }

    isSubmitting = true;
    try {
      // Static data for now
      await new Promise(r => setTimeout(r, 1500));
      const user = {
        ...userForm,
        id: crypto.randomUUID()
      };

      // For backend integration, uncomment the following and comment out static data:
      /*
      const response = await fetch(`${BASE_URL}/users`, {
        method: 'POST',
        headers: getAuthHeaders(),
        body: JSON.stringify({
          name: userForm.name,
          role: userForm.role,
          contact: userForm.contact
        })
      });
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      const user = await response.json();
      */

      $users = [user, ...$users];
      showNotification('User added successfully!', 'success');
      userForm = {
        name: '',
        role: 'Seller',
        contact: ''
      };
      showNewUserModal = false;
    } catch (error) {
      showNotification(`Failed to create user: ${error.message}`, 'error');
    } finally {
      isSubmitting = false;
    }
  }

  /**
   * Updates a user
   * Backend Endpoint: PUT /api/users/:id
   * URL Parameters:
   *   - id (string): User ID
   * Request Body (JSON):
   *   - name (string): Updated user name
   *   - role (string): Updated role
   *   - contact (string): Updated contact info
   *   - Example:
   *     {
   *       "name": "John Doe",
   *       "role": "Seller",
   *       "contact": "john@example.com"
   *     }
   * Expected Response (JSON):
   *   - Status: 200 OK
   *   - Body: Updated user object
   *   - Example:
   *     {
   *       "id": "uuid-string",
   *       "name": "John Doe",
   *       "role": "Seller",
   *       "contact": "john@example.com"
   *     }
   * Error Handling:
   *   - 400 Bad Request: Missing or invalid fields
   *   - 404 Not Found: User ID not found
   *   - 401 Unauthorized: Missing or invalid authentication token
   *   - 500 Internal Server Error: Server-side issue
   * Backend Notes:
   *   - Validate required fields (name, role)
   *   - Update only provided fields
   *   - Return the full updated user object
   *   - Implement rate limiting to prevent abuse
   *   - Validate authentication token if required
   */
  async function updateUser() {
    // Client-side validation
    if (!userForm.name) {
      showNotification('User name is required', 'error');
      return;
    }
    if (!userForm.role) {
      showNotification('Role is required', 'error');
      return;
    }

    isSubmitting = true;
    try {
      // Static data for now
      await new Promise(r => setTimeout(r, 1500));
      const updatedUser = {
        id: selectedUser.id,
        ...userForm
      };

      // For backend integration, uncomment the following and comment out static data:
      /*
      const response = await fetch(`${BASE_URL}/users/${selectedUser.id}`, {
        method: 'PUT',
        headers: getAuthHeaders(),
        body: JSON.stringify({
          name: userForm.name,
          role: userForm.role,
          contact: userForm.contact
        })
      });
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      const updatedUser = await response.json();
      */

      $users = $users.map(u => (u.id === updatedUser.id ? updatedUser : u));
      $sales = $sales.map(s => (s.userId === updatedUser.id ? { ...s, userName: updatedUser.name } : s));
      showNotification('User updated successfully!', 'success');
      showUserDetailsModal = false;
    } catch (error) {
      showNotification(`Failed to update user: ${error.message}`, 'error');
    } finally {
      isSubmitting = false;
    }
  }

  /**
   * Deletes a user by ID
   * Backend Endpoint: DELETE /api/users/:id
   * URL Parameters:
   *   - id (string): User ID
   * Expected Response (JSON):
   *   - Status: 200 OK
   *   - Body: { success: true }
   *   - Example:
   *     {
   *       "success": true
   *     }
   * Error Handling:
   *   - 404 Not Found: User ID not found
   *   - 401 Unauthorized: Missing or invalid authentication token
   *   - 500 Internal Server Error: Server-side issue
   * Backend Notes:
   *   - Verify the user ID exists before deletion
   *   - Ensure no associated sales exist or handle cascading deletion
   *   - Return { success: true } on successful deletion
   *   - Implement rate limiting to prevent abuse
   *   - Validate authentication token if required
   */
  async function deleteUser(id) {
    try {
      // Static data for now
      await new Promise(r => setTimeout(r, 500));

      // For backend integration, uncomment the following:
      /*
      const response = await fetch(`${BASE_URL}/users/${id}`, {
        method: 'DELETE',
        headers: getAuthHeaders()
      });
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      const result = await response.json();
      if (!result.success) {
        throw new Error('Failed to delete user');
      }
      */

      $users = $users.filter(u => u.id !== id);
      $sales = $sales.filter(s => s.userId !== id); // Remove associated sales
      showNotification('User deleted successfully!', 'success');
    } catch (error) {
      showNotification(`Failed to delete user: ${error.message}`, 'error');
    }
  }

  /**
   * Creates a new sale
   * Backend Endpoint: POST /api/sales
   * Request Body (JSON):
   *   - userId (string): ID of the user (seller)
   *   - product (string): Product sold
   *   - amount (string): Sale amount with two decimal places
   *   - description (string): Optional description
   *   - date (string): ISO date string (e.g., '2025-07-28')
   *   - Example:
   *     {
   *       "userId": "uuid-string",
   *       "product": "Product A",
   *       "amount": "750.50",
   *       "description": "Sale of Product A",
   *       "date": "2025-07-28"
   *     }
   * Expected Response (JSON):
   *   - Status: 201 Created
   *   - Body: Created sale object with userName
   *   - Example:
   *     {
   *       "id": "uuid-string",
   *       "userId": "uuid-string",
   *       "userName": "John Doe",
   *       "product": "Product A",
   *       "amount": "750.50",
   *       "description": "Sale of Product A",
   *       "date": "2025-07-28",
   *       "status": "Completed"
   *     }
   * Error Handling:
   *   - 400 Bad Request: Missing or invalid fields
   *   - 401 Unauthorized: Missing or invalid authentication token
   *   - 500 Internal Server Error: Server-side issue
   * Backend Notes:
   *   - Validate required fields (userId, product, amount, date)
   *   - Ensure amount is a string with two decimal places
   *   - Generate unique id (e.g., UUID)
   *   - Include userName by joining with users table
   *   - Implement rate limiting to prevent abuse
   *   - Validate authentication token if required
   */
  async function createSale() {
    // Client-side validation
    if (!saleForm.userId) {
      showNotification('User is required', 'error');
      return;
    }
    if (!saleForm.product) {
      showNotification('Product is required', 'error');
      return;
    }
    if (!saleForm.amount || parseFloat(saleForm.amount) <= 0) {
      showNotification('Amount must be greater than 0', 'error');
      return;
    }
    if (!saleForm.date) {
      showNotification('Date is required', 'error');
      return;
    }

    isSubmitting = true;
    try {
      // Static data for now
      await new Promise(r => setTimeout(r, 1500));
      const user = $users.find(u => u.id === saleForm.userId);
      const sale = {
        ...saleForm,
        id: crypto.randomUUID(),
        userName: user ? user.name : 'Unknown',
        status: 'Completed',
        amount: parseFloat(saleForm.amount).toFixed(2) // Ensure two decimal places
      };

      // For backend integration, uncomment the following and comment out static data:
      /*
      const response = await fetch(`${BASE_URL}/sales`, {
        method: 'POST',
        headers: getAuthHeaders(),
        body: JSON.stringify({
          userId: saleForm.userId,
          product: saleForm.product,
          amount: parseFloat(saleForm.amount).toFixed(2),
          description: saleForm.description,
          date: saleForm.date
        })
      });
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      const sale = await response.json();
      */

      $sales = [sale, ...$sales];
      showNotification('Sale recorded successfully!', 'success');
      saleForm = {
        userId: '',
        product: 'Product A',
        amount: '',
        date: new Date().toISOString().split('T')[0],
        description: ''
      };
      showNewSaleModal = false;
    } catch (error) {
      showNotification(`Failed to record sale: ${error.message}`, 'error');
    } finally {
      isSubmitting = false;
    }
  }

  /**
   * Exports sales as CSV
   * Client-side function, but can leverage GET /api/sales/csv for data
   * Backend Endpoint (optional): GET /api/sales/csv
   * Query Parameters:
   *   - search (string): Search term for product or user name
   * Expected Response (optional server-side CSV):
   *   - Status: 200 OK
   *   - Content-Type: text/csv
   *   - Body: CSV content with headers: Date,User,Product,Amount,Status,Description
   * Client-side Notes:
   *   - Generates CSV from filteredSales using client-side filtering
   *   - Escapes commas in fields to prevent CSV formatting issues
   * Backend Notes:
   *   - Optionally implement server-side CSV generation to reduce client-side processing
   *   - Ensure CSV headers match client-side expectations
   *   - Apply same filtering logic (product, userName) server-side if implemented
   *   - Implement rate limiting to prevent abuse
   */
  function exportCSV() {
    try {
      const headers = ['Date', 'User', 'Product', 'Amount', 'Status', 'Description'];
      const rows = [];
      filteredSales.forEach(s => {
        // Escape commas in fields to prevent CSV formatting issues
        const escapedRow = [
          s.date,
          `"${s.userName.replace(/"/g, '""')}"`,
          s.product,
          s.amount,
          s.status,
          `"${s.description.replace(/"/g, '""')}"`
        ];
        rows.push(escapedRow);
      });

      // For server-side CSV generation, uncomment the following and comment out client-side logic:
      /*
      const response = await fetch(`${BASE_URL}/sales/csv?search=${encodeURIComponent(searchTerm)}`, {
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
      link.setAttribute('download', `sales_${new Date().toISOString().split('T')[0]}.csv`);
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

  // Reactive filtered sales
  $: filteredSales = $sales.filter(sale => {
    const matchesSearch = sale.userName.toLowerCase().includes(searchTerm.toLowerCase()) ||
                         sale.product.toLowerCase().includes(searchTerm.toLowerCase());
    return matchesSearch;
  });

  // Initialize data and time updates
  onMount(async () => {
    await fetchUsers();
    await fetchSales();
    const interval = setInterval(() => {
      currentTime = new Date().toLocaleTimeString();
    }, 1000);
    return () => clearInterval(interval);
  });

  // Refetch sales when users change or search term updates
  $: if ($users || searchTerm) {
    fetchSales();
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
          Users Dashboard
        </h1>
        <p class="text-gray-400">Manage sellers and track their sales</p>
        <p class="text-sm text-gray-500 mt-1">Last updated: {currentTime}</p>
      </div>
      <div class="flex items-center space-x-4 mt-4 lg:mt-0">
        <button 
          on:click={() => {
            userForm = { name: '', role: 'Seller', contact: '' };
            showNewUserModal = true;
          }}
          class="bg-gradient-to-r from-blue-600 to-purple-600 hover:from-blue-700 hover:to-purple-700 px-6 py-2 rounded-lg font-medium transition-all duration-200 flex items-center space-x-2 shadow-lg"
        >
          <Plus class="w-5 h-5" />
          <span>New User</span>
        </button>
        <button 
          on:click={() => showNewSaleModal = true}
          class="bg-gradient-to-r from-green-600 to-teal-600 hover:from-green-700 hover:to-teal-700 px-6 py-2 rounded-lg font-medium transition-all duration-200 flex items-center space-x-2 shadow-lg"
        >
          <ShoppingCart class="w-5 h-5" />
          <span>Record Sale</span>
        </button>
      </div>
    </div>

    <!-- Filters and Actions -->
    <div class="flex flex-col lg:flex-row justify-between items-start lg:items-center gap-4 mb-6">
      <div class="relative">
        <Search class="absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400 w-5 h-5" />
        <input
          type="text"
          placeholder="Search users or products..."
          bind:value={searchTerm}
          class="bg-black/50 backdrop-blur-md border border-gray-800/50 rounded-lg pl-10 pr-4 py-2 text-gray-100 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-600 w-full sm:w-64"
        />
      </div>
      <button 
        on:click={exportCSV}
        class="bg-gray-800 hover:bg-gray-700 border border-gray-800/50 px-4 py-2 rounded-lg font-medium transition-all duration-200 flex items-center space-x-2"
      >
        <Download class="w-5 h-5" />
        <span>Export Sales CSV</span>
      </button>
    </div>

    <!-- Users Table -->
    <div class="bg-gray-900/50 backdrop-blur-md rounded-2xl border border-gray-800/50 shadow-xl overflow-hidden mb-8">
      {#if $isLoading}
        <div class="flex items-center justify-center p-12">
          <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600"></div>
        </div>
      {:else}
        <div class="overflow-x-auto">
          <table class="min-w-full text-sm">
            <thead class="bg-gray-900/30 border-b border-gray-800/50">
              <tr>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Name</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Role</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Contact</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Actions</th>
              </tr>
            </thead>
            <tbody>
              {#each $users as user, index}
                <tr 
                  class="border-b border-gray-800/50 hover:bg-gray-800/30 transition-all duration-200"
                  style="animation-delay: {index * 0.1}s;"
                >
                  <td class="px-6 py-4 font-medium text-gray-100">{user.name}</td>
                  <td class="px-6 py-4 text-gray-400">{user.role}</td>
                  <td class="px-6 py-4 text-gray-400">{user.contact || 'N/A'}</td>
                  <td class="px-6 py-4">
                    <div class="flex items-center space-x-2">
                      <button 
                        on:click={() => {
                          selectedUser = user;
                          userForm = { ...user };
                          showUserDetailsModal = true;
                        }}
                        class="p-2 hover:bg-gray-800/50 rounded-lg transition-colors"
                        title="Edit User"
                      >
                        <Eye class="w-4 h-4 text-blue-400" />
                      </button>
                      <button 
                        on:click={() => deleteUser(user.id)}
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
          {#if $users.length === 0}
            <div class="text-center py-12 text-gray-400">
              <User class="w-12 h-12 mx-auto mb-4 opacity-50" />
              <p>No users found</p>
            </div>
          {/if}
        </div>
      {/if}
    </div>

    <!-- Sales Table -->
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
                <th class="px-6 py-4 text-left font-semibold text-gray-400">User</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Product</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Amount</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Status</th>
              </tr>
            </thead>
            <tbody>
              {#each filteredSales as sale, index}
                <tr 
                  class="border-b border-gray-800/50 hover:bg-gray-800/30 transition-all duration-200"
                  style="animation-delay: {index * 0.1}s;"
                >
                  <td class="px-6 py-4 text-gray-400">{sale.date}</td>
                  <td class="px-6 py-4 font-medium text-gray-100">{sale.userName}</td>
                  <td class="px-6 py-4 text-gray-400">{sale.product}</td>
                  <td class="px-6 py-4 font-semibold text-green-400">Rs. {sale.amount}</td>
                  <td class="px-6 py-4">
                    <span class="px-3 py-1 rounded-full text-xs font-semibold border {getStatusColor(sale.status)}">
                      {sale.status}
                    </span>
                  </td>
                </tr>
              {/each}
            </tbody>
          </table>
          {#if filteredSales.length === 0}
            <div class="text-center py-12 text-gray-400">
              <ShoppingCart class="w-12 h-12 mx-auto mb-4 opacity-50" />
              <p>No sales found</p>
            </div>
          {/if}
        </div>
      {/if}
    </div>

    <!-- New User Modal -->
    {#if showNewUserModal}
      <div class="fixed inset-0 bg-black/60 backdrop-blur-sm flex items-center justify-center z-50 p-4" on:click|self={() => showNewUserModal = false}>
        <div class="bg-gray-900/80 backdrop-blur-md rounded-2xl border border-gray-800/50 p-8 w-full max-w-md shadow-2xl">
          <div class="flex justify-between items-center mb-6">
            <h3 class="text-2xl font-bold text-gray-100">New User</h3>
            <button 
              on:click={() => showNewUserModal = false}
              class="p-2 hover:bg-gray-800/50 rounded-lg transition-colors"
            >
              <X class="w-6 h-6 text-gray-400" />
            </button>
          </div>
          <div class="space-y-4">
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Name</label>
              <input 
                type="text" 
                bind:value={userForm.name}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-600"
                placeholder="User name"
                required
              />
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Role</label>
              <select 
                bind:value={userForm.role}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 focus:outline-none focus:ring-2 focus:ring-blue-600"
              >
                <option value="Seller">Seller</option>
                <option value="Manager">Manager</option>
              </select>
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Contact (Optional)</label>
              <input 
                type="text" 
                bind:value={userForm.contact}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-600"
                placeholder="Email or phone"
              />
            </div>
            <div class="flex gap-4 pt-4">
              <button 
                on:click={() => showNewUserModal = false}
                class="flex-1 bg-gray-800 hover:bg-gray-700 border border-gray-800/50 px-4 py-2 rounded-lg font-medium transition-all duration-200 text-gray-100"
              >
                Cancel
              </button>
              <button 
                on:click={createUser}
                disabled={isSubmitting}
                class="flex-1 bg-gradient-to-r from-blue-600 to-purple-600 hover:from-blue-700 hover:to-purple-700 disabled:opacity-50 disabled:cursor-not-allowed px-4 py-2 rounded-lg font-medium transition-all duration-200"
              >
                {isSubmitting ? 'Saving...' : 'Add User'}
              </button>
            </div>
          </div>
        </div>
      </div>
    {/if}

    <!-- Edit User Modal -->
    {#if showUserDetailsModal && selectedUser}
      <div class="fixed inset-0 bg-black/60 backdrop-blur-sm flex items-center justify-center z-50 p-4" on:click|self={() => showUserDetailsModal = false}>
        <div class="bg-gray-900/80 backdrop-blur-md rounded-2xl border border-gray-800/50 p-8 w-full max-w-md shadow-2xl">
          <div class="flex justify-between items-center mb-6">
            <h3 class="text-2xl font-bold text-gray-100">Edit User</h3>
            <button 
              on:click={() => showUserDetailsModal = false}
              class="p-2 hover:bg-gray-800/50 rounded-lg transition-colors"
            >
              <X class="w-6 h-6 text-gray-400" />
            </button>
          </div>
          <div class="space-y-4">
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Name</label>
              <input 
                type="text" 
                bind:value={userForm.name}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-600"
                placeholder="User name"
                required
              />
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Role</label>
              <select 
                bind:value={userForm.role}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 focus:outline-none focus:ring-2 focus:ring-blue-600"
              >
                <option value="Seller">Seller</option>
                <option value="Manager">Manager</option>
              </select>
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Contact (Optional)</label>
              <input 
                type="text" 
                bind:value={userForm.contact}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-600"
                placeholder="Email or phone"
              />
            </div>
            <div class="flex gap-4 pt-4">
              <button 
                on:click={() => showUserDetailsModal = false}
                class="flex-1 bg-gray-800 hover:bg-gray-700 border border-gray-800/50 px-4 py-2 rounded-lg font-medium transition-all duration-200 text-gray-100"
              >
                Cancel
              </button>
              <button 
                on:click={updateUser}
                disabled={isSubmitting}
                class="flex-1 bg-gradient-to-r from-blue-600 to-purple-600 hover:from-blue-700 hover:to-purple-700 disabled:opacity-50 disabled:cursor-not-allowed px-4 py-2 rounded-lg font-medium transition-all duration-200"
              >
                {isSubmitting ? 'Updating...' : 'Update User'}
              </button>
            </div>
          </div>
        </div>
      </div>
    {/if}

    <!-- New Sale Modal -->
    {#if showNewSaleModal}
      <div class="fixed inset-0 bg-black/60 backdrop-blur-sm flex items-center justify-center z-50 p-4" on:click|self={() => showNewSaleModal = false}>
        <div class="bg-gray-900/80 backdrop-blur-md rounded-2xl border border-gray-800/50 p-8 w-full max-w-md shadow-2xl">
          <div class="flex justify-between items-center mb-6">
            <h3 class="text-2xl font-bold text-gray-100">Record Sale</h3>
            <button 
              on:click={() => showNewSaleModal = false}
              class="p-2 hover:bg-gray-800/50 rounded-lg transition-colors"
            >
              <X class="w-6 h-6 text-gray-400" />
            </button>
          </div>
          <div class="space-y-4">
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">User</label>
              <select 
                bind:value={saleForm.userId}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 focus:outline-none focus:ring-2 focus:ring-blue-600"
              >
                <option value="" disabled>Select a user</option>
                {#each $users as user}
                  <option value={user.id}>{user.name}</option>
                {/each}
              </select>
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Product</label>
              <select 
                bind:value={saleForm.product}
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
                bind:value={saleForm.amount}
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
                bind:value={saleForm.description}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-600 h-20 resize-none"
                placeholder="Sale description"
              ></textarea>
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Date</label>
              <input 
                type="date" 
                bind:value={saleForm.date}
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
                on:click={createSale}
                disabled={isSubmitting}
                class="flex-1 bg-gradient-to-r from-green-600 to-teal-600 hover:from-green-700 hover:to-teal-700 disabled:opacity-50 disabled:cursor-not-allowed px-4 py-2 rounded-lg font-medium transition-all duration-200"
              >
                {isSubmitting ? 'Saving...' : 'Record Sale'}
              </button>
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