<script>
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';
  import { Plus, Download, Truck, Package, Search, Filter, Eye, Trash2, X, Check, AlertCircle } from 'lucide-svelte';

  // API Configuration - Update this when connecting to a real backend
  const BASE_URL = 'http://localhost:3000/api'; // Replace with your backend URL, e.g., 'https://your-backend.com/api'

  // Optional: Add authentication headers if required
  const getAuthHeaders = () => ({
    // 'Authorization': `Bearer ${localStorage.getItem('token') || 'YOUR_TOKEN_HERE'}`, // Uncomment and replace with actual token if needed
    'Content-Type': 'application/json'
  });

  // Stores for state management
  const distributors = writable([]);
  const purchases = writable([]);
  const notifications = writable([]);
  const metrics = writable({
    totalPurchaseValue: 0,
    avgPurchaseValue: 0,
    purchaseCount: 0,
    stockValue: 0
  });
  const isLoading = writable(true);

  // UI state
  let currentTime = new Date().toLocaleTimeString();
  let searchTerm = '';
  let showNewDistributorModal = false;
  let showNewPurchaseModal = false;
  let showDistributorDetailsModal = false;
  let selectedDistributor = null;
  let isSubmitting = false;

  // Form states
  let distributorForm = {
    name: '',
    contact: '',
    address: ''
  };
  let purchaseForm = {
    distributorId: '',
    product: '',
    quantity: '',
    unitPrice: '',
    date: new Date().toISOString().split('T')[0],
    description: ''
  };

  /**
   * Fetches distributors
   * Backend Endpoint: GET /api/distributors
   * Expected Response (JSON):
   *   - Status: 200 OK
   *   - Body: Array of distributor objects
   *   - Example:
   *     [
   *       {
   *         "id": "uuid-string",
   *         "name": "Global Suppliers",
   *         "contact": "info@globalsuppliers.com",
   *         "address": "123 Supplier St, City"
   *       },
   *       ...
   *     ]
   * Error Handling:
   *   - 401 Unauthorized: Missing or invalid authentication token
   *   - 500 Internal Server Error: Server-side issue
   * Backend Notes:
   *   - Return an empty array [] if no distributors are found
   *   - Generate unique id (e.g., UUID) for each distributor
   *   - Implement rate limiting to prevent abuse
   *   - Validate authentication token if required
   */
  async function fetchDistributors() {
    $isLoading = true;
    try {
      // Static data for now
      await new Promise(r => setTimeout(r, 800));
      const names = ['Global Suppliers', 'TradeMart', 'Bulk Distributors', 'Prime Imports', 'SupplyChain Co'];
      const mockDistributors = [];
      Array.from({ length: 5 }).forEach(() => {
        const name = names[Math.floor(Math.random() * names.length)];
        mockDistributors.push({
          id: crypto.randomUUID(),
          name,
          contact: `${name.toLowerCase().replace(' ', '')}@example.com`,
          address: `${Math.floor(Math.random() * 1000)} ${name} St, City`
        });
      });

      // For backend integration, uncomment the following and comment out static data:
      /*
      const response = await fetch(`${BASE_URL}/distributors`, {
        method: 'GET',
        headers: getAuthHeaders()
      });
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      const mockDistributors = await response.json();
      */

      if (!Array.isArray(mockDistributors)) {
        throw new Error('Invalid distributor data format');
      }

      $distributors = mockDistributors;
    } catch (error) {
      showNotification(`Failed to fetch distributors: ${error.message}`, 'error');
    } finally {
      $isLoading = false;
    }
  }

  /**
   * Fetches purchase records
   * Backend Endpoint: GET /api/purchases
   * Query Parameters:
   *   - search (string): Search term for product or distributor name
   * Expected Response (JSON):
   *   - Status: 200 OK
   *   - Body: Array of purchase objects
   *   - Example:
   *     [
   *       {
   *         "id": "uuid-string",
   *         "distributorId": "uuid-string",
   *         "distributorName": "Global Suppliers",
   *         "product": "Product A",
   *         "quantity": 100,
   *         "unitPrice": "10.50",
   *         "description": "Bulk purchase of Product A",
   *         "date": "2025-07-28",
   *         "status": "Received"
   *       },
   *       ...
   *     ]
   * Error Handling:
   *   - 400 Bad Request: Invalid search parameter
   *   - 401 Unauthorized: Missing or invalid authentication token
   *   - 500 Internal Server Error: Server-side issue
   * Backend Notes:
   *   - Filter purchases based on the 'search' query parameter (product or distributor name)
   *   - Return an empty array [] if no purchases are found
   *   - Ensure unitPrice is a string with two decimal places for client-side display
   *   - Include distributorName for display purposes (join with distributors table)
   *   - Generate unique id (e.g., UUID) for each purchase
   *   - Implement rate limiting to prevent abuse
   *   - Validate authentication token if required
   */
  async function fetchPurchases() {
    try {
      // Static data for now
      await new Promise(r => setTimeout(r, 800));
      const products = ['Product A', 'Product B', 'Component C', 'Material D', 'Item E'];
      const mockPurchases = [];
      $distributors.forEach(dist => {
        const count = Math.floor(Math.random() * 3) + 1;
        Array.from({ length: count }).forEach(() => {
          mockPurchases.push({
            id: crypto.randomUUID(),
            distributorId: dist.id,
            distributorName: dist.name,
            product: products[Math.floor(Math.random() * products.length)],
            quantity: Math.floor(Math.random() * 200) + 10,
            unitPrice: (Math.random() * 50 + 5).toFixed(2),
            description: `Purchase of ${products[Math.floor(Math.random() * products.length)]}`,
            date: new Date(Date.now() - Math.floor(Math.random() * 30) * 86400000).toISOString().split('T')[0],
            status: Math.random() > 0.2 ? 'Received' : Math.random() > 0.5 ? 'Pending' : 'In Transit'
          });
        });
      });

      // For backend integration, uncomment the following and comment out static data:
      /*
      const response = await fetch(`${BASE_URL}/purchases?search=${encodeURIComponent(searchTerm)}`, {
        method: 'GET',
        headers: getAuthHeaders()
      });
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      const mockPurchases = await response.json();
      */

      if (!Array.isArray(mockPurchases)) {
        throw new Error('Invalid purchase data format');
      }

      $purchases = mockPurchases;
      updateMetrics(mockPurchases);
    } catch (error) {
      showNotification(`Failed to fetch purchases: ${error.message}`, 'error');
    }
  }

  /**
   * Updates metrics based on purchase data
   * Client-side function, no backend endpoint required
   * Notes:
   *   - Calculates totalPurchaseValue, avgPurchaseValue, purchaseCount, and stockValue from purchases
   *   - Filters for received purchases only to ensure accurate financial metrics
   *   - Updates metrics store with formatted values
   */
  function updateMetrics(data) {
    const received = [];
    data.forEach(p => {
      if (p.status === 'Received') {
        received.push(p);
      }
    });
    let total = 0;
    let stock = 0;
    received.forEach(p => {
      const value = parseFloat(p.unitPrice) * parseInt(p.quantity);
      total += value;
      stock += value;
    });
    $metrics = {
      totalPurchaseValue: total.toFixed(2),
      avgPurchaseValue: received.length > 0 ? (total / received.length).toFixed(2) : '0.00',
      purchaseCount: data.length,
      stockValue: stock.toFixed(2)
    };
  }

  /**
   * Creates a new distributor
   * Backend Endpoint: POST /api/distributors
   * Request Body (JSON):
   *   - name (string): Distributor name
   *   - contact (string): Contact info (e.g., email or phone)
   *   - address (string): Distributor address
   *   - Example:
   *     {
   *       "name": "Global Suppliers",
   *       "contact": "info@globalsuppliers.com",
   *       "address": "123 Supplier St, City"
   *     }
   * Expected Response (JSON):
   *   - Status: 201 Created
   *   - Body: Created distributor object
   *   - Example:
   *     {
   *       "id": "uuid-string",
   *       "name": "Global Suppliers",
   *       "contact": "info@globalsuppliers.com",
   *       "address": "123 Supplier St, City"
   *     }
   * Error Handling:
   *   - 400 Bad Request: Missing or invalid fields
   *   - 401 Unauthorized: Missing or invalid authentication token
   *   - 500 Internal Server Error: Server-side issue
   * Backend Notes:
   *   - Validate required fields (name, contact, address)
   *   - Generate unique id (e.g., UUID)
   *   - Return the full distributor object
   *   - Implement rate limiting to prevent abuse
   *   - Validate authentication token if required
   */
  async function createDistributor() {
    // Client-side validation
    if (!distributorForm.name) {
      showNotification('Distributor name is required', 'error');
      return;
    }
    if (!distributorForm.contact) {
      showNotification('Contact info is required', 'error');
      return;
    }
    if (!distributorForm.address) {
      showNotification('Address is required', 'error');
      return;
    }

    isSubmitting = true;
    try {
      // Static data for now
      await new Promise(r => setTimeout(r, 1500));
      const distributor = {
        ...distributorForm,
        id: crypto.randomUUID()
      };

      // For backend integration, uncomment the following and comment out static data:
      /*
      const response = await fetch(`${BASE_URL}/distributors`, {
        method: 'POST',
        headers: getAuthHeaders(),
        body: JSON.stringify({
          name: distributorForm.name,
          contact: distributorForm.contact,
          address: distributorForm.address
        })
      });
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      const distributor = await response.json();
      */

      $distributors = [distributor, ...$distributors];
      showNotification('Distributor added successfully!', 'success');
      distributorForm = {
        name: '',
        contact: '',
        address: ''
      };
      showNewDistributorModal = false;
    } catch (error) {
      showNotification(`Failed to create distributor: ${error.message}`, 'error');
    } finally {
      isSubmitting = false;
    }
  }

  /**
   * Updates a distributor
   * Backend Endpoint: PUT /api/distributors/:id
   * URL Parameters:
   *   - id (string): Distributor ID
   * Request Body (JSON):
   *   - name (string): Updated distributor name
   *   - contact (string): Updated contact info
   *   - address (string): Updated address
   *   - Example:
   *     {
   *       "name": "Global Suppliers",
   *       "contact": "info@globalsuppliers.com",
   *       "address": "123 Supplier St, City"
   *     }
   * Expected Response (JSON):
   *   - Status: 200 OK
   *   - Body: Updated distributor object
   *   - Example:
   *     {
   *       "id": "uuid-string",
   *       "name": "Global Suppliers",
   *       "contact": "info@globalsuppliers.com",
   *       "address": "123 Supplier St, City"
   *     }
   * Error Handling:
   *   - 400 Bad Request: Missing or invalid fields
   *   - 404 Not Found: Distributor ID not found
   *   - 401 Unauthorized: Missing or invalid authentication token
   *   - 500 Internal Server Error: Server-side issue
   * Backend Notes:
   *   - Validate required fields (name, contact, address)
   *   - Update only provided fields
   *   - Return the full updated distributor object
   *   - Implement rate limiting to prevent abuse
   *   - Validate authentication token if required
   */
  async function updateDistributor() {
    // Client-side validation
    if (!distributorForm.name) {
      showNotification('Distributor name is required', 'error');
      return;
    }
    if (!distributorForm.contact) {
      showNotification('Contact info is required', 'error');
      return;
    }
    if (!distributorForm.address) {
      showNotification('Address is required', 'error');
      return;
    }

    isSubmitting = true;
    try {
      // Static data for now
      await new Promise(r => setTimeout(r, 1500));
      const updatedDistributor = {
        id: selectedDistributor.id,
        ...distributorForm
      };

      // For backend integration, uncomment the following and comment out static data:
      /*
      const response = await fetch(`${BASE_URL}/distributors/${selectedDistributor.id}`, {
        method: 'PUT',
        headers: getAuthHeaders(),
        body: JSON.stringify({
          name: distributorForm.name,
          contact: distributorForm.contact,
          address: distributorForm.address
        })
      });
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      const updatedDistributor = await response.json();
      */

      $distributors = $distributors.map(d => (d.id === updatedDistributor.id ? updatedDistributor : d));
      $purchases = $purchases.map(p => (p.distributorId === updatedDistributor.id ? { ...p, distributorName: updatedDistributor.name } : p));
      showNotification('Distributor updated successfully!', 'success');
      showDistributorDetailsModal = false;
    } catch (error) {
      showNotification(`Failed to update distributor: ${error.message}`, 'error');
    } finally {
      isSubmitting = false;
    }
  }

  /**
   * Deletes a distributor by ID
   * Backend Endpoint: DELETE /api/distributors/:id
   * URL Parameters:
   *   - id (string): Distributor ID
   * Expected Response (JSON):
   *   - Status: 200 OK
   *   - Body: { success: true }
   *   - Example:
   *     {
   *       "success": true
   *     }
   * Error Handling:
   *   - 404 Not Found: Distributor ID not found
   *   - 401 Unauthorized: Missing or invalid authentication token
   *   - 500 Internal Server Error: Server-side issue
   * Backend Notes:
   *   - Verify the distributor ID exists before deletion
   *   - Ensure no associated purchases exist or handle cascading deletion
   *   - Return { success: true } on successful deletion
   *   - Implement rate limiting to prevent abuse
   *   - Validate authentication token if required
   */
  async function deleteDistributor(id) {
    try {
      // Static data for now
      await new Promise(r => setTimeout(r, 500));

      // For backend integration, uncomment the following:
      /*
      const response = await fetch(`${BASE_URL}/distributors/${id}`, {
        method: 'DELETE',
        headers: getAuthHeaders()
      });
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      const result = await response.json();
      if (!result.success) {
        throw new Error('Failed to delete distributor');
      }
      */

      $distributors = $distributors.filter(d => d.id !== id);
      $purchases = $purchases.filter(p => p.distributorId !== id); // Remove associated purchases
      showNotification('Distributor deleted successfully!', 'success');
    } catch (error) {
      showNotification(`Failed to delete distributor: ${error.message}`, 'error');
    }
  }

  /**
   * Creates a new purchase
   * Backend Endpoint: POST /api/purchases
   * Request Body (JSON):
   *   - distributorId (string): ID of the distributor
   *   - product (string): Product name
   *   - quantity (number): Quantity purchased
   *   - unitPrice (string): Unit price with two decimal places
   *   - description (string): Optional description
   *   - date (string): ISO date string (e.g., '2025-07-28')
   *   - Example:
   *     {
   *       "distributorId": "uuid-string",
   *       "product": "Product A",
   *       "quantity": 100,
   *       "unitPrice": "10.50",
   *       "description": "Bulk purchase of Product A",
   *       "date": "2025-07-28"
   *     }
   * Expected Response (JSON):
   *   - Status: 201 Created
   *   - Body: Created purchase object with distributorName
   *   - Example:
   *     {
   *       "id": "uuid-string",
   *       "distributorId": "uuid-string",
   *       "distributorName": "Global Suppliers",
   *       "product": "Product A",
   *       "quantity": 100,
   *       "unitPrice": "10.50",
   *       "description": "Bulk purchase of Product A",
   *       "date": "2025-07-28",
   *       "status": "Received"
   *     }
   * Error Handling:
   *   - 400 Bad Request: Missing or invalid fields
   *   - 401 Unauthorized: Missing or invalid authentication token
   *   - 500 Internal Server Error: Server-side issue
   * Backend Notes:
   *   - Validate required fields (distributorId, product, quantity, unitPrice, date)
   *   - Ensure unitPrice is a string with two decimal places
   *   - Generate unique id (e.g., UUID)
   *   - Include distributorName by joining with distributors table
   *   - Implement rate limiting to prevent abuse
   *   - Validate authentication token if required
   */
  async function createPurchase() {
    // Client-side validation
    if (!purchaseForm.distributorId) {
      showNotification('Distributor is required', 'error');
      return;
    }
    if (!purchaseForm.product) {
      showNotification('Product is required', 'error');
      return;
    }
    if (!purchaseForm.quantity || parseInt(purchaseForm.quantity) <= 0) {
      showNotification('Quantity must be greater than 0', 'error');
      return;
    }
    if (!purchaseForm.unitPrice || parseFloat(purchaseForm.unitPrice) <= 0) {
      showNotification('Unit price must be greater than 0', 'error');
      return;
    }
    if (!purchaseForm.date) {
      showNotification('Date is required', 'error');
      return;
    }

    isSubmitting = true;
    try {
      // Static data for now
      await new Promise(r => setTimeout(r, 1500));
      const distributor = $distributors.find(d => d.id === purchaseForm.distributorId);
      const purchase = {
        ...purchaseForm,
        id: crypto.randomUUID(),
        distributorName: distributor ? distributor.name : 'Unknown',
        status: 'Received',
        unitPrice: parseFloat(purchaseForm.unitPrice).toFixed(2), // Ensure two decimal places
        quantity: parseInt(purchaseForm.quantity) // Ensure integer
      };

      // For backend integration, uncomment the following and comment out static data:
      /*
      const response = await fetch(`${BASE_URL}/purchases`, {
        method: 'POST',
        headers: getAuthHeaders(),
        body: JSON.stringify({
          distributorId: purchaseForm.distributorId,
          product: purchaseForm.product,
          quantity: parseInt(purchaseForm.quantity),
          unitPrice: parseFloat(purchaseForm.unitPrice).toFixed(2),
          description: purchaseForm.description,
          date: purchaseForm.date
        })
      });
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      const purchase = await response.json();
      */

      $purchases = [purchase, ...$purchases];
      updateMetrics($purchases);
      showNotification('Purchase recorded successfully!', 'success');
      purchaseForm = {
        distributorId: '',
        product: '',
        quantity: '',
        unitPrice: '',
        date: new Date().toISOString().split('T')[0],
        description: ''
      };
      showNewPurchaseModal = false;
    } catch (error) {
      showNotification(`Failed to record purchase: ${error.message}`, 'error');
    } finally {
      isSubmitting = false;
    }
  }

  /**
   * Exports purchases as CSV
   * Client-side function, but can leverage GET /api/purchases/csv for data
   * Backend Endpoint (optional): GET /api/purchases/csv
   * Query Parameters:
   *   - search (string): Search term for product or distributor name
   * Expected Response (optional server-side CSV):
   *   - Status: 200 OK
   *   - Content-Type: text/csv
   *   - Body: CSV content with headers: Date,Distributor,Product,Quantity,UnitPrice,Status,Description
   * Client-side Notes:
   *   - Generates CSV from filteredPurchases using client-side filtering
   *   - Escapes commas in fields to prevent CSV formatting issues
   * Backend Notes:
   *   - Optionally implement server-side CSV generation to reduce client-side processing
   *   - Ensure CSV headers match client-side expectations
   *   - Apply same filtering logic (product, distributorName) server-side if implemented
   *   - Implement rate limiting to prevent abuse
   */
  function exportCSV() {
    try {
      const headers = ['Date', 'Distributor', 'Product', 'Quantity', 'UnitPrice', 'Status', 'Description'];
      const rows = [];
      filteredPurchases.forEach(p => {
        // Escape commas in fields to prevent CSV formatting issues
        const escapedRow = [
          p.date,
          `"${p.distributorName.replace(/"/g, '""')}"`,
          p.product,
          p.quantity,
          p.unitPrice,
          p.status,
          `"${p.description.replace(/"/g, '""')}"`
        ];
        rows.push(escapedRow);
      });

      // For server-side CSV generation, uncomment the following and comment out client-side logic:
      /*
      const response = await fetch(`${BASE_URL}/purchases/csv?search=${encodeURIComponent(searchTerm)}`, {
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
      link.setAttribute('download', `purchases_${new Date().toISOString().split('T')[0]}.csv`);
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

  // Reactive filtered purchases
  $: filteredPurchases = $purchases.filter(purchase => {
    const matchesSearch = purchase.distributorName.toLowerCase().includes(searchTerm.toLowerCase()) ||
                         purchase.product.toLowerCase().includes(searchTerm.toLowerCase());
    return matchesSearch;
  });

  // Initialize data and time updates
  onMount(async () => {
    await fetchDistributors();
    await fetchPurchases();
    const interval = setInterval(() => {
      currentTime = new Date().toLocaleTimeString();
    }, 1000);
    return () => clearInterval(interval);
  });

  // Refetch purchases when distributors change or search term updates
  $: if ($distributors || searchTerm) {
    fetchPurchases();
  }

  function getStatusColor(status) {
    switch (status) {
      case 'Received': return 'bg-green-600/20 text-green-400 border-green-600/50';
      case 'Pending': return 'bg-yellow-600/20 text-yellow-300 border-yellow-600/50';
      case 'In Transit': return 'bg-blue-600/20 text-blue-400 border-blue-600/50';
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
          Wholesale Distributor Dashboard
        </h1>
        <p class="text-gray-400">Manage distributors and track purchase records</p>
        <p class="text-sm text-gray-500 mt-1">Last updated: {currentTime}</p>
      </div>
      <div class="flex items-center space-x-4 mt-4 lg:mt-0">
        <button 
          on:click={() => {
            distributorForm = { name: '', contact: '', address: '' };
            showNewDistributorModal = true;
          }}
          class="bg-gradient-to-r from-blue-600 to-purple-600 hover:from-blue-700 hover:to-purple-700 px-6 py-2 rounded-lg font-medium transition-all duration-200 flex items-center space-x-2 shadow-lg"
        >
          <Plus class="w-5 h-5" />
          <span>New Distributor</span>
        </button>
        <button 
          on:click={() => showNewPurchaseModal = true}
          class="bg-gradient-to-r from-green-600 to-teal-600 hover:from-green-700 hover:to-teal-700 px-6 py-2 rounded-lg font-medium transition-all duration-200 flex items-center space-x-2 shadow-lg"
        >
          <Truck class="w-5 h-5" />
          <span>Record Purchase</span>
        </button>
      </div>
    </div>

    <!-- Metrics Grid -->
    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
      <div class="bg-gray-900/80 backdrop-blur-md p-6 rounded-2xl shadow-xl border border-gray-800/50 hover:bg-gray-800/80 transition-all duration-300 animate-fade-in-up">
        <div class="flex items-center justify-between">
          <div>
            <p class="text-sm font-medium text-gray-400 mb-1">Total Purchase Value</p>
            <h2 class="text-3xl font-bold text-green-400">Rs. {$metrics.totalPurchaseValue}</h2>
          </div>
          <Package class="w-8 h-8 text-green-400" />
        </div>
      </div>
      <div class="bg-gray-900/80 backdrop-blur-md p-6 rounded-2xl shadow-xl border border-gray-800/50 hover:bg-gray-800/80 transition-all duration-300 animate-fade-in-up" style="animation-delay: 0.1s;">
        <div class="flex items-center justify-between">
          <div>
            <p class="text-sm font-medium text-gray-400 mb-1">Average Purchase Value</p>
            <h2 class="text-3xl font-bold text-blue-400">Rs. {$metrics.avgPurchaseValue}</h2>
          </div>
          <Truck class="w-8 h-8 text-blue-400" />
        </div>
      </div>
      <div class="bg-gray-900/80 backdrop-blur-md p-6 rounded-2xl shadow-xl border border-gray-800/50 hover:bg-gray-800/80 transition-all duration-300 animate-fade-in-up" style="animation-delay: 0.2s;">
        <div class="flex items-center justify-between">
          <div>
            <p class="text-sm font-medium text-gray-400 mb-1">Purchase Count</p>
            <h2 class="text-3xl font-bold text-purple-400">{$metrics.purchaseCount}</h2>
          </div>
          <Package class="w-8 h-8 text-purple-400" />
        </div>
      </div>
      <div class="bg-gray-900/80 backdrop-blur-md p-6 rounded-2xl shadow-xl border border-gray-800/50 hover:bg-gray-800/80 transition-all duration-300 animate-fade-in-up" style="animation-delay: 0.3s;">
        <div class="flex items-center justify-between">
          <div>
            <p class="text-sm font-medium text-gray-400 mb-1">Stock Value</p>
            <h2 class="text-3xl font-bold text-orange-400">Rs. {$metrics.stockValue}</h2>
          </div>
          <Truck class="w-8 h-8 text-orange-400" />
        </div>
      </div>
    </div>

    <!-- Filters and Actions -->
    <div class="flex flex-col lg:flex-row justify-between items-start lg:items-center gap-4 mb-6">
      <div class="relative">
        <Search class="absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400 w-5 h-5" />
        <input
          type="text"
          placeholder="Search distributors or products..."
          bind:value={searchTerm}
          class="bg-black/50 backdrop-blur-md border border-gray-800/50 rounded-lg pl-10 pr-4 py-2 text-gray-100 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-600 w-full sm:w-64"
        />
      </div>
      <button 
        on:click={exportCSV}
        class="bg-gray-800 hover:bg-gray-700 border border-gray-800/50 px-4 py-2 rounded-lg font-medium transition-all duration-200 flex items-center space-x-2"
      >
        <Download class="w-5 h-5" />
        <span>Export Purchases CSV</span>
      </button>
    </div>

    <!-- Distributors Table -->
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
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Contact</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Address</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Actions</th>
              </tr>
            </thead>
            <tbody>
              {#each $distributors as distributor, index}
                <tr 
                  class="border-b border-gray-800/50 hover:bg-gray-800/30 transition-all duration-200"
                  style="animation-delay: {index * 0.1}s;"
                >
                  <td class="px-6 py-4 font-medium text-gray-100">{distributor.name}</td>
                  <td class="px-6 py-4 text-gray-400">{distributor.contact}</td>
                  <td class="px-6 py-4 text-gray-400">{distributor.address}</td>
                  <td class="px-6 py-4">
                    <div class="flex items-center space-x-2">
                      <button 
                        on:click={() => {
                          selectedDistributor = distributor;
                          distributorForm = { ...distributor };
                          showDistributorDetailsModal = true;
                        }}
                        class="p-2 hover:bg-gray-800/50 rounded-lg transition-colors"
                        title="Edit Distributor"
                      >
                        <Eye class="w-4 h-4 text-blue-400" />
                      </button>
                      <button 
                        on:click={() => deleteDistributor(distributor.id)}
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
          {#if $distributors.length === 0}
            <div class="text-center py-12 text-gray-400">
              <Truck class="w-12 h-12 mx-auto mb-4 opacity-50" />
              <p>No distributors found</p>
            </div>
          {/if}
        </div>
      {/if}
    </div>

    <!-- Purchases Table -->
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
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Distributor</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Product</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Quantity</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Unit Price</th>
                <th class="px-6 py-4 text-left font-semibold text-gray-400">Status</th>
              </tr>
            </thead>
            <tbody>
              {#each filteredPurchases as purchase, index}
                <tr 
                  class="border-b border-gray-800/50 hover:bg-gray-800/30 transition-all duration-200"
                  style="animation-delay: {index * 0.1}s;"
                >
                  <td class="px-6 py-4 text-gray-400">{purchase.date}</td>
                  <td class="px-6 py-4 font-medium text-gray-100">{purchase.distributorName}</td>
                  <td class="px-6 py-4 text-gray-400">{purchase.product}</td>
                  <td class="px-6 py-4 text-gray-400">{purchase.quantity}</td>
                  <td class="px-6 py-4 font-semibold text-green-400">Rs. {purchase.unitPrice}</td>
                  <td class="px-6 py-4">
                    <span class="px-3 py-1 rounded-full text-xs font-semibold border {getStatusColor(purchase.status)}">
                      {purchase.status}
                    </span>
                  </td>
                </tr>
              {/each}
            </tbody>
          </table>
          {#if filteredPurchases.length === 0}
            <div class="text-center py-12 text-gray-400">
              <Package class="w-12 h-12 mx-auto mb-4 opacity-50" />
              <p>No purchases found</p>
            </div>
          {/if}
        </div>
      {/if}
    </div>

    <!-- New Distributor Modal -->
    {#if showNewDistributorModal}
      <div class="fixed inset-0 bg-black/60 backdrop-blur-sm flex items-center justify-center z-50 p-4" on:click|self={() => showNewDistributorModal = false}>
        <div class="bg-gray-900/80 backdrop-blur-md rounded-2xl border border-gray-800/50 p-8 w-full max-w-md shadow-2xl">
          <div class="flex justify-between items-center mb-6">
            <h3 class="text-2xl font-bold text-gray-100">New Distributor</h3>
            <button 
              on:click={() => showNewDistributorModal = false}
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
                bind:value={distributorForm.name}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-600"
                placeholder="Distributor name"
                required
              />
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Contact</label>
              <input 
                type="text" 
                bind:value={distributorForm.contact}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-600"
                placeholder="Email or phone"
                required
              />
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Address</label>
              <input 
                type="text" 
                bind:value={distributorForm.address}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-600"
                placeholder="Address"
                required
              />
            </div>
            <div class="flex gap-4 pt-4">
              <button 
                on:click={() => showNewDistributorModal = false}
                class="flex-1 bg-gray-800 hover:bg-gray-700 border border-gray-800/50 px-4 py-2 rounded-lg font-medium transition-all duration-200 text-gray-100"
              >
                Cancel
              </button>
              <button 
                on:click={createDistributor}
                disabled={isSubmitting}
                class="flex-1 bg-gradient-to-r from-blue-600 to-purple-600 hover:from-blue-700 hover:to-purple-700 disabled:opacity-50 disabled:cursor-not-allowed px-4 py-2 rounded-lg font-medium transition-all duration-200"
              >
                {isSubmitting ? 'Saving...' : 'Add Distributor'}
              </button>
            </div>
          </div>
        </div>
      </div>
    {/if}

    <!-- Edit Distributor Modal -->
    {#if showDistributorDetailsModal && selectedDistributor}
      <div class="fixed inset-0 bg-black/60 backdrop-blur-sm flex items-center justify-center z-50 p-4" on:click|self={() => showDistributorDetailsModal = false}>
        <div class="bg-gray-900/80 backdrop-blur-md rounded-2xl border border-gray-800/50 p-8 w-full max-w-md shadow-2xl">
          <div class="flex justify-between items-center mb-6">
            <h3 class="text-2xl font-bold text-gray-100">Edit Distributor</h3>
            <button 
              on:click={() => showDistributorDetailsModal = false}
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
                bind:value={distributorForm.name}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-600"
                placeholder="Distributor name"
                required
              />
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Contact</label>
              <input 
                type="text" 
                bind:value={distributorForm.contact}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-600"
                placeholder="Email or phone"
                required
              />
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Address</label>
              <input 
                type="text" 
                bind:value={distributorForm.address}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-600"
                placeholder="Address"
                required
              />
            </div>
            <div class="flex gap-4 pt-4">
              <button 
                on:click={() => showDistributorDetailsModal = false}
                class="flex-1 bg-gray-800 hover:bg-gray-700 border border-gray-800/50 px-4 py-2 rounded-lg font-medium transition-all duration-200 text-gray-100"
              >
                Cancel
              </button>
              <button 
                on:click={updateDistributor}
                disabled={isSubmitting}
                class="flex-1 bg-gradient-to-r from-blue-600 to-purple-600 hover:from-blue-700 hover:to-purple-700 disabled:opacity-50 disabled:cursor-not-allowed px-4 py-2 rounded-lg font-medium transition-all duration-200"
              >
                {isSubmitting ? 'Updating...' : 'Update Distributor'}
              </button>
            </div>
          </div>
        </div>
      </div>
    {/if}

    <!-- New Purchase Modal -->
    {#if showNewPurchaseModal}
      <div class="fixed inset-0 bg-black/60 backdrop-blur-sm flex items-center justify-center z-50 p-4" on:click|self={() => showNewPurchaseModal = false}>
        <div class="bg-gray-900/80 backdrop-blur-md rounded-2xl border border-gray-800/50 p-8 w-full max-w-md shadow-2xl">
          <div class="flex justify-between items-center mb-6">
            <h3 class="text-2xl font-bold text-gray-100">Record Purchase</h3>
            <button 
              on:click={() => showNewPurchaseModal = false}
              class="p-2 hover:bg-gray-800/50 rounded-lg transition-colors"
            >
              <X class="w-6 h-6 text-gray-400" />
            </button>
          </div>
          <div class="space-y-4">
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Distributor</label>
              <select 
                bind:value={purchaseForm.distributorId}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 focus:outline-none focus:ring-2 focus:ring-blue-600"
              >
                <option value="" disabled>Select a distributor</option>
                {#each $distributors as distributor}
                  <option value={distributor.id}>{distributor.name}</option>
                {/each}
              </select>
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Product</label>
              <input 
                type="text" 
                bind:value={purchaseForm.product}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-600"
                placeholder="Product name"
                required
              />
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Quantity</label>
              <input 
                type="number" 
                bind:value={purchaseForm.quantity}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-600"
                placeholder="Quantity"
                required
                min="1"
                step="1"
              />
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Unit Price (Rs.)</label>
              <input 
                type="number" 
                bind:value={purchaseForm.unitPrice}
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
                bind:value={purchaseForm.description}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-600 h-20 resize-none"
                placeholder="Purchase description"
              ></textarea>
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-2">Date</label>
              <input 
                type="date" 
                bind:value={purchaseForm.date}
                class="w-full bg-black/50 border border-gray-800/50 rounded-lg px-4 py-2 text-gray-100 focus:outline-none focus:ring-2 focus:ring-blue-600"
                required
              />
            </div>
            <div class="flex gap-4 pt-4">
              <button 
                on:click={() => showNewPurchaseModal = false}
                class="flex-1 bg-gray-800 hover:bg-gray-700 border border-gray-800/50 px-4 py-2 rounded-lg font-medium transition-all duration-200 text-gray-100"
              >
                Cancel
              </button>
              <button 
                on:click={createPurchase}
                disabled={isSubmitting}
                class="flex-1 bg-gradient-to-r from-green-600 to-teal-600 hover:from-green-700 hover:to-teal-700 disabled:opacity-50 disabled:cursor-not-allowed px-4 py-2 rounded-lg font-medium transition-all duration-200"
              >
                {isSubmitting ? 'Saving...' : 'Record Purchase'}
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