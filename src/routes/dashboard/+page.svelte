<script>
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';
  
  // Stores for different data states
  const statsStore = writable([]);
  const activityStore = writable([]);
  const loadingStore = writable(false);
  const errorStore = writable(null);
  
  // UI State
  let mounted = false;
  let hoveredCard = null;
  let isGeneratingReport = false;
  let currentTime = new Date().toLocaleTimeString();
  let selectedPeriod = 'week';
  let showTransactionModal = false;
  let showReportsModal = false;
  let showDetailsModal = false;
  
  // Data state
  let stats = [];
  let todayActivity = [];
  let businessMetrics = {
    customerGrowth: '+25%',
    retentionRate: '92%',
    satisfaction: '4.8'
  };
  
  // API simulation functions
  const api = {
    async fetchStats(period = 'week') {
      loadingStore.set(true);
      errorStore.set(null);
      
      // Simulate API delay
      await new Promise(resolve => setTimeout(resolve, 1000));
      
      try {
        // Simulate different data based on period
        const periodMultiplier = {
          'today': 0.1,
          'week': 1,
          'month': 4.3
        };
        
        const multiplier = periodMultiplier[period] || 1;
        
        const baseStats = [
          {
            label: "Total Sales",
            baseValue: 150000,
            change: "+12%",
            icon: "M12 8c-1.657 0-3 .895-3 2s1.343 2 3 2 3 .895 3 2-1.343 2-3 2m0-8c1.11 0 2.08.402 2.599 1M12 8V7m0 1v8m0 0v1m0-1c-1.11 0-2.08-.402-2.599-1",
            trend: "up",
            color: "blue"
          },
          {
            label: "Orders",
            baseValue: 350,
            change: "+8%",
            icon: "M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-3 7h3m-3 4h3m-6-4h.01M9 16h.01",
            trend: "up",
            color: "emerald"
          },
          {
            label: "Revenue",
            baseValue: 225000,
            change: "+15%",
            icon: "M17 9V7a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2m2 4h10a2 2 0 002-2v-6a2 2 0 00-2-2H9a2 2 0 00-2 2v6a2 2 0 002 2zm7-5a2 2 0 11-4 0 2 2 0 014 0z",
            trend: "up",
            color: "purple"
          },
          {
            label: "Expenses",
            baseValue: 90000,
            change: "-5%",
            icon: "M9 7h6m0 10v-3m-3 3h.01M9 17h.01M9 14h.01M12 14h.01M15 11h.01M12 11h.01M9 11h.01M7 21h10a2 2 0 002-2V5a2 2 0 00-2-2H7a2 2 0 00-2 2v14a2 2 0 002 2z",
            trend: "down",
            color: "red"
          },
        ];
        
        const processedStats = baseStats.map(stat => ({
          ...stat,
          value: `Rs. ${Math.round(stat.baseValue * multiplier).toLocaleString()}`
        }));
        
        statsStore.set(processedStats);
        return processedStats;
      } catch (error) {
        errorStore.set('Failed to fetch statistics');
        throw error;
      } finally {
        loadingStore.set(false);
      }
    },
    
    async fetchTodayActivity() {
      await new Promise(resolve => setTimeout(resolve, 500));
      
      return [
        { label: 'Sales Today', value: 'Rs. 12,500', icon: 'M12 8c-1.657 0-3 .895-3 2s1.343 2 3 2 3 .895 3 2-1.343 2-3 2', color: 'blue' },
        { label: 'New Orders', value: '24', icon: 'M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2', color: 'emerald' },
        { label: 'Active Users', value: '1,284', icon: 'M12 4.354a4 4 0 110 5.292M15 21H3v-1a6 6 0 0112 0v1zm0 0h6v-1a6 6 0 00-9-5.197', color: 'purple' },
        { label: 'Conversions', value: '18.2%', icon: 'M13 7h8m0 0v8m0-8l-8 8-4-4-6 6', color: 'emerald' }
      ];
    },
    
    async generateReport(type = 'csv') {
      await new Promise(resolve => setTimeout(resolve, 2000));
      
      const headers = ["Label", "Value", "Change", "Trend", "Period"];
      const rows = stats.map(s => [s.label, s.value, s.change, s.trend, selectedPeriod]);
      
      if (type === 'csv') {
        const csvContent = "data:text/csv;charset=utf-8,"
          + headers.join(",") + "\n"
          + rows.map(e => e.join(",")).join("\n");
        
        const encodedUri = encodeURI(csvContent);
        const link = document.createElement("a");
        link.setAttribute("href", encodedUri);
        link.setAttribute("download", `dashboard_report_${selectedPeriod}_${new Date().toISOString().split('T')[0]}.csv`);
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
      }
      
      return { success: true, type, recordCount: rows.length };
    },
    
    async createTransaction(transactionData) {
      await new Promise(resolve => setTimeout(resolve, 1500));
      
      // Simulate API response
      return {
        id: Math.random().toString(36).substr(2, 9),
        ...transactionData,
        status: 'success',
        timestamp: new Date().toISOString()
      };
    }
  };
  
  // Event handlers
  async function handlePeriodChange(period) {
    selectedPeriod = period;
    stats = await api.fetchStats(period);
  }
  
  async function handleDownloadReport() {
    isGeneratingReport = true;
    try {
      await api.generateReport('csv');
      // Show success notification
      showNotification('Report downloaded successfully!', 'success');
    } catch (error) {
      showNotification('Failed to generate report', 'error');
    } finally {
      isGeneratingReport = false;
    }
  }
  
  function handleNewTransaction() {
    showTransactionModal = true;
  }
  
  function handleViewReports() {
    showReportsModal = true;
  }
  
  function handleViewDetails() {
    showDetailsModal = true;
  }
  
  async function submitTransaction(formData) {
    try {
      loadingStore.set(true);
      const result = await api.createTransaction(formData);
      showNotification('Transaction created successfully!', 'success');
      showTransactionModal = false;
      
      // Refresh data
      stats = await api.fetchStats(selectedPeriod);
      todayActivity = await api.fetchTodayActivity();
    } catch (error) {
      showNotification('Failed to create transaction', 'error');
    } finally {
      loadingStore.set(false);
    }
  }
  
  // Notification system
  let notifications = [];
  
  function showNotification(message, type = 'info') {
    const id = Math.random().toString(36).substr(2, 9);
    notifications = [...notifications, { id, message, type }];
    
    setTimeout(() => {
      notifications = notifications.filter(n => n.id !== id);
    }, 5000);
  }
  
  // Initialize data
  onMount(async () => {
    mounted = true;
    
    // Update time every second
    const timeInterval = setInterval(() => {
      currentTime = new Date().toLocaleTimeString();
    }, 1000);
    
    // Load initial data
    try {
      [stats, todayActivity] = await Promise.all([
        api.fetchStats(selectedPeriod),
        api.fetchTodayActivity()
      ]);
    } catch (error) {
      showNotification('Failed to load dashboard data', 'error');
    }
    
    return () => clearInterval(timeInterval);
  });

  function getColorClasses(color, isNegative = false) {
    if (isNegative) {
      return {
        bg: 'bg-gradient-to-br from-red-50 to-rose-100',
        darkBg: 'dark:from-red-950/50 dark:to-rose-900/30',
        border: 'border-red-200 dark:border-red-800/50',
        text: 'text-red-600 dark:text-red-400',
        iconBg: 'bg-red-500/10 dark:bg-red-500/20',
        shadow: 'shadow-red-500/20'
      };
    }

    const colors = {
      blue: {
        bg: 'bg-gradient-to-br from-blue-50 to-indigo-100',
        darkBg: 'dark:from-blue-950/50 dark:to-indigo-900/30',
        border: 'border-blue-200 dark:border-blue-800/50',
        text: 'text-blue-600 dark:text-blue-400',
        iconBg: 'bg-blue-500/10 dark:bg-blue-500/20',
        shadow: 'shadow-blue-500/20'
      },
      emerald: {
        bg: 'bg-gradient-to-br from-emerald-50 to-green-100',
        darkBg: 'dark:from-emerald-950/50 dark:to-green-900/30',
        border: 'border-emerald-200 dark:border-emerald-800/50',
        text: 'text-emerald-600 dark:text-emerald-400',
        iconBg: 'bg-emerald-500/10 dark:bg-emerald-500/20',
        shadow: 'shadow-emerald-500/20'
      },
      purple: {
        bg: 'bg-gradient-to-br from-purple-50 to-violet-100',
        darkBg: 'dark:from-purple-950/50 dark:to-violet-900/30',
        border: 'border-purple-200 dark:border-purple-800/50',
        text: 'text-purple-600 dark:text-purple-400',
        iconBg: 'bg-purple-500/10 dark:bg-purple-500/20',
        shadow: 'shadow-purple-500/20'
      },
      red: {
        bg: 'bg-gradient-to-br from-red-50 to-rose-100',
        darkBg: 'dark:from-red-950/50 dark:to-rose-900/30',
        border: 'border-red-200 dark:border-red-800/50',
        text: 'text-red-600 dark:text-red-400',
        iconBg: 'bg-red-500/10 dark:bg-red-500/20',
        shadow: 'shadow-red-500/20'
      }
    };

    return colors[color] || colors.blue;
  }
</script>

<svelte:head>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800;900&display=swap" rel="stylesheet">
  <script src="https://cdn.tailwindcss.com"></script>
  <script>
    tailwind.config = {
      darkMode: 'class',
      theme: {
        extend: {
          fontFamily: {
            'inter': ['Inter', 'sans-serif'],
          },
          animation: {
            'fade-in-up': 'fadeInUp 0.6s ease-out forwards',
            'slide-in': 'slideIn 0.5s ease-out forwards',
            'pulse-slow': 'pulse 3s cubic-bezier(0.4, 0, 0.6, 1) infinite',
            'float': 'float 6s ease-in-out infinite',
          }
        }
      }
    }
  </script>
</svelte:head>

<div class="min-h-screen bg-gray-50 dark:bg-gray-950 text-gray-900 dark:text-gray-100 font-inter transition-colors duration-300">
  <!-- Notifications -->
  <div class="fixed top-4 right-4 z-50 space-y-2">
    {#each notifications as notification (notification.id)}
      <div class={`px-4 py-3 rounded-lg shadow-lg border flex items-center space-x-3 min-w-[300px] animate-slide-in
                  ${notification.type === 'success' ? 'bg-green-50 dark:bg-green-950/50 border-green-200 dark:border-green-800 text-green-800 dark:text-green-200' : 
                    notification.type === 'error' ? 'bg-red-50 dark:bg-red-950/50 border-red-200 dark:border-red-800 text-red-800 dark:text-red-200' : 
                    'bg-blue-50 dark:bg-blue-950/50 border-blue-200 dark:border-blue-800 text-blue-800 dark:text-blue-200'}`}>
        <svg class="w-5 h-5 flex-shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          {#if notification.type === 'success'}
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/>
          {:else if notification.type === 'error'}
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/>
          {:else}
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/>
          {/if}
        </svg>
        <span class="text-sm font-medium">{notification.message}</span>
      </div>
    {/each}
  </div>

  <!-- Navigation Bar -->
  <nav class="sticky top-0 z-40 bg-white/80 dark:bg-gray-900/80 backdrop-blur-xl border-b border-gray-200/50 dark:border-gray-800/50">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
      <div class="flex items-center justify-between h-16">
        <div class="flex items-center space-x-4">
          <div class="w-10 h-10 bg-gradient-to-r from-blue-600 to-purple-600 rounded-xl flex items-center justify-center shadow-lg">
            <svg class="w-6 h-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M13 10V3L4 14h7v7l9-11h-7z"/>
            </svg>
          </div>
          <div>
            <h1 class="text-xl font-bold bg-gradient-to-r from-gray-900 to-gray-600 dark:from-white dark:to-gray-300 bg-clip-text text-transparent">Dashboard</h1>
            <p class="text-xs text-gray-500 dark:text-gray-400">Analytics Overview</p>
          </div>
        </div>
        
        <div class="flex items-center space-x-4">
          <div class="hidden sm:flex items-center space-x-2 px-3 py-1.5 bg-green-50 dark:bg-green-950/50 rounded-full border border-green-200 dark:border-green-800/50">
            <div class="w-2 h-2 bg-green-500 rounded-full animate-pulse"></div>
            <span class="text-xs font-medium text-green-700 dark:text-green-400">Live</span>
          </div>
          <div class="text-sm font-medium text-gray-600 dark:text-gray-400 hidden md:block">
            {currentTime}
          </div>
          <button 
            on:click={handleDownloadReport}
            class="w-9 h-9 bg-gray-100 dark:bg-gray-800 hover:bg-gray-200 dark:hover:bg-gray-700 rounded-xl flex items-center justify-center transition-colors"
            title="Quick Export">
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 10v6m0 0l-3-3m3 3l3-3m2 8H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"/>
            </svg>
          </button>
        </div>
      </div>
    </div>
  </nav>

  <main class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
    <!-- Header Section -->
    <div class="mb-12">
      <div class="flex flex-col lg:flex-row lg:items-center justify-between gap-6">
        <div>
          <h2 class="text-3xl lg:text-4xl font-bold text-gray-900 dark:text-white mb-2">
            Welcome back, <span class="bg-gradient-to-r from-blue-600 to-purple-600 bg-clip-text text-transparent">Sarah</span>
          </h2>
          <p class="text-lg text-gray-600 dark:text-gray-400">Here's what's happening with your business today.</p>
        </div>
        
        <div class="flex flex-wrap gap-3">
          <button 
            on:click={() => handlePeriodChange('today')}
            class={`px-4 py-2 rounded-xl text-sm font-medium transition-colors shadow-sm
                   ${selectedPeriod === 'today' ? 'bg-blue-600 text-white' : 'bg-white dark:bg-gray-800 hover:bg-gray-50 dark:hover:bg-gray-700 border border-gray-200 dark:border-gray-700 text-gray-900 dark:text-white'}`}>
            Today
          </button>
          <button 
            on:click={() => handlePeriodChange('week')}
            class={`px-4 py-2 rounded-xl text-sm font-medium transition-colors shadow-sm
                   ${selectedPeriod === 'week' ? 'bg-blue-600 text-white' : 'bg-white dark:bg-gray-800 hover:bg-gray-50 dark:hover:bg-gray-700 border border-gray-200 dark:border-gray-700 text-gray-900 dark:text-white'}`}>
            This Week
          </button>
          <button 
            on:click={() => handlePeriodChange('month')}
            class={`px-4 py-2 rounded-xl text-sm font-medium transition-colors shadow-sm
                   ${selectedPeriod === 'month' ? 'bg-blue-600 text-white' : 'bg-white dark:bg-gray-800 hover:bg-gray-50 dark:hover:bg-gray-700 border border-gray-200 dark:border-gray-700 text-gray-900 dark:text-white'}`}>
            This Month
          </button>
        </div>
      </div>
    </div>

    <!-- Loading State -->
    {#if $loadingStore}
      <div class="flex items-center justify-center py-12">
        <div class="flex items-center space-x-3">
          <div class="w-6 h-6 border-2 border-blue-600 border-t-transparent rounded-full animate-spin"></div>
          <span class="text-gray-600 dark:text-gray-400">Loading dashboard data...</span>
        </div>
      </div>
    {:else}
      <!-- Stats Grid -->
      <div class="grid grid-cols-1 sm:grid-cols-2 xl:grid-cols-4 gap-6 mb-12">
        {#each stats as stat, i}
          <div
            class={`group relative bg-white dark:bg-gray-900 rounded-2xl p-6 border border-gray-200/50 dark:border-gray-800/50
                    hover:border-gray-300 dark:hover:border-gray-700 hover:shadow-xl dark:hover:shadow-2xl
                    hover:shadow-gray-200/50 dark:hover:shadow-black/20 transition-all duration-500 cursor-pointer
                    hover:-translate-y-1 ${mounted ? 'animate-fade-in-up' : 'opacity-0'}`}
            style="animation-delay: {i * 100}ms;"
            on:mouseenter={() => hoveredCard = i}
            on:mouseleave={() => hoveredCard = null}
            on:click={() => showNotification(`Viewing detailed ${stat.label.toLowerCase()} analytics`, 'info')}
            role="button"
            tabindex="0"
          >
            <!-- Gradient Overlay on Hover -->
            <div class={`absolute inset-0 ${getColorClasses(stat.color, stat.change.includes('-')).bg} ${getColorClasses(stat.color, stat.change.includes('-')).darkBg} opacity-0 group-hover:opacity-100 transition-opacity duration-500 rounded-2xl`}></div>
            
            <div class="relative z-10">
              <div class="flex items-start justify-between mb-6">
                <div class={`w-12 h-12 ${getColorClasses(stat.color, stat.change.includes('-')).iconBg} rounded-xl flex items-center justify-center group-hover:scale-110 transition-transform duration-300`}>
                  <svg class={`w-6 h-6 ${getColorClasses(stat.color, stat.change.includes('-')).text}`} fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
                    <path d={stat.icon}/>
                  </svg>
                </div>
                
                <div class={`flex items-center space-x-1.5 px-2.5 py-1 rounded-full text-xs font-semibold
                            ${stat.change.includes('-') 
                              ? 'bg-red-50 dark:bg-red-950/30 text-red-600 dark:text-red-400' 
                              : 'bg-green-50 dark:bg-green-950/30 text-green-600 dark:text-green-400'}`}>
                  <svg class="w-3 h-3" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
                    {#if stat.change.includes('-')}
                      <path d="M13 17h8m0 0V9m0 8l-8-8-4 4-6-6"/>
                    {:else}
                      <path d="M13 7h8m0 0v8m0-8l-8 8-4-4-6 6"/>
                    {/if}
                  </svg>
                  {stat.change}
                </div>
              </div>
              
              <div class="space-y-2">
                <p class="text-sm font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wide">
                  {stat.label}
                </p>
                <p class="text-3xl font-bold text-gray-900 dark:text-white group-hover:text-gray-700 dark:group-hover:text-gray-200 transition-colors">
                  {stat.value}
                </p>
                <p class="text-sm text-gray-600 dark:text-gray-400">
                  vs last {selectedPeriod}
                </p>
              </div>
            </div>
            
            {#if hoveredCard === i}
              <div class="absolute inset-0 bg-gradient-to-br from-white/10 to-transparent dark:from-white/5 rounded-2xl"></div>
            {/if}
          </div>
        {/each}
      </div>

      <!-- Export Section -->
      <div class="mb-12">
        <div class="bg-gradient-to-r from-blue-600 to-purple-600 rounded-2xl p-8 text-white relative overflow-hidden">
          <div class="absolute inset-0 bg-black/10"></div>
          <div class="relative z-10 flex flex-col lg:flex-row items-center justify-between gap-6">
            <div class="flex items-center space-x-6">
              <div class="w-16 h-16 bg-white/20 rounded-2xl flex items-center justify-center backdrop-blur-sm">
                <svg class="w-8 h-8 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 17v-2m3 2v-4m3 4v-6m2 10H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"/>
                </svg>
              </div>
              <div>
                <h3 class="text-2xl font-bold mb-2">Export Analytics Report</h3>
                <p class="text-blue-100">Download comprehensive business insights for {selectedPeriod}</p>
              </div>
            </div>
            
            <button
              on:click={handleDownloadReport}
              disabled={isGeneratingReport}
              class="px-8 py-4 bg-white/20 hover:bg-white/30 disabled:bg-white/10 backdrop-blur-sm
                     text-white font-semibold rounded-xl border border-white/30 hover:border-white/50
                     transition-all duration-300 hover:scale-105 disabled:cursor-not-allowed 
                     disabled:scale-100 flex items-center space-x-3 min-w-[200px] justify-center"
            >
              {#if isGeneratingReport}
                <div class="w-5 h-5 border-2 border-white/30 border-t-white rounded-full animate-spin"></div>
                <span>Generating...</span>
              {:else}
                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 10v6m0 0l-3-3m3 3l3-3m2 8H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"/>
                </svg>
                <span>Download Report</span>
              {/if}
            </button>
          </div>
        </div>
      </div>

      <!-- Content Grid -->
      <div class="grid grid-cols-1 xl:grid-cols-3 gap-8">
        <!-- Business Overview -->
        <div class="xl:col-span-2 bg-white dark:bg-gray-900 rounded-2xl border border-gray-200/50 dark:border-gray-800/50 overflow-hidden hover:shadow-xl dark:hover:shadow-2xl hover:shadow-gray-200/50 dark:hover:shadow-black/20 transition-all duration-500">
          <div class="p-6 border-b border-gray-200/50 dark:border-gray-800/50">
            <div class="flex items-center justify-between">
              <div class="flex items-center space-x-4">
                <div class="w-3 h-3 bg-blue-500 rounded-full animate-pulse-slow"></div>
                <h3 class="text-xl font-bold text-gray-900 dark:text-white">Business Overview</h3>
              </div>
              <button 
                on:click={handleViewDetails}
                class="px-4 py-2 bg-blue-50 dark:bg-blue-950/50 hover:bg-blue-100 dark:hover:bg-blue-900/50 text-blue-600 dark:text-blue-400 rounded-xl text-sm font-medium transition-colors border border-blue-200 dark:border-blue-800/50">
                View Details
              </button>
            </div>
          </div>
          
          <div class="p-6">
            <p class="text-gray-600 dark:text-gray-400 leading-relaxed mb-8 text-lg">
              Monitor your business performance with real-time analytics and comprehensive insights. 
              Track key metrics, identify trends, and make data-driven decisions.
            </p>
            
            <div class="grid grid-cols-1 sm:grid-cols-3 gap-4 mb-8">
              <div class="bg-gray-50 dark:bg-gray-800 rounded-xl p-4 hover:bg-gray-100 dark:hover:bg-gray-700 transition-colors cursor-pointer"
                   on:click={() => showNotification('Customer growth details loaded', 'info')}>
                <p class="text-sm font-medium text-gray-500 dark:text-gray-400 mb-1">Customer Growth</p>
                <p class="text-2xl font-bold text-gray-900 dark:text-white">{businessMetrics.customerGrowth}</p>
              </div>
              <div class="bg-gray-50 dark:bg-gray-800 rounded-xl p-4 hover:bg-gray-100 dark:hover:bg-gray-700 transition-colors cursor-pointer"
                   on:click={() => showNotification('Retention rate analytics opened', 'info')}>
                <p class="text-sm font-medium text-gray-500 dark:text-gray-400 mb-1">Retention Rate</p>
                <p class="text-2xl font-bold text-gray-900 dark:text-white">{businessMetrics.retentionRate}</p>
              </div>
              <div class="bg-gray-50 dark:bg-gray-800 rounded-xl p-4 hover:bg-gray-100 dark:hover:bg-gray-700 transition-colors cursor-pointer"
                   on:click={() => showNotification('Customer satisfaction report viewing', 'info')}>
                <p class="text-sm font-medium text-gray-500 dark:text-gray-400 mb-1">Satisfaction</p>
                <p class="text-2xl font-bold text-gray-900 dark:text-white">{businessMetrics.satisfaction}</p>
              </div>
            </div>
            
            <div class="flex flex-col sm:flex-row gap-3">
              <button 
                on:click={handleNewTransaction}
                class="flex-1 px-6 py-3 bg-blue-600 hover:bg-blue-700 text-white font-semibold rounded-xl transition-colors flex items-center justify-center space-x-2 hover:scale-105 transform duration-200">
                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"/>
                </svg>
                <span>New Transaction</span>
              </button>
              <button 
                on:click={handleViewReports}
                class="flex-1 px-6 py-3 bg-gray-100 dark:bg-gray-800 hover:bg-gray-200 dark:hover:bg-gray-700 text-gray-900 dark:text-white font-semibold rounded-xl transition-colors flex items-center justify-center space-x-2 hover:scale-105 transform duration-200">
                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 17v-2m3 2v-4m3 4v-6m2 10H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"/>
                </svg>
                <span>View Reports</span>
              </button>
            </div>
          </div>
        </div>

        <!-- Activity Panel -->
        <div class="bg-white dark:bg-gray-900 rounded-2xl border border-gray-200/50 dark:border-gray-800/50 overflow-hidden hover:shadow-xl dark:hover:shadow-2xl hover:shadow-gray-200/50 dark:hover:shadow-black/20 transition-all duration-500">
          <div class="p-6 border-b border-gray-200/50 dark:border-gray-800/50">
            <div class="flex items-center space-x-4">
              <div class="w-3 h-3 bg-green-500 rounded-full animate-pulse-slow"></div>
              <h3 class="text-xl font-bold text-gray-900 dark:text-white">Today's Activity</h3>
            </div>
          </div>
          
          <div class="p-6 space-y-6">
            {#each todayActivity as item}
              <div class="flex items-center justify-between group hover:bg-gray-50 dark:hover:bg-gray-800 rounded-xl p-3 -mx-3 transition-colors cursor-pointer"
                   on:click={() => showNotification(`Viewing ${item.label.toLowerCase()} details`, 'info')}>
                <div class="flex items-center space-x-4">
                  <div class={`w-10 h-10 ${getColorClasses(item.color).iconBg} rounded-xl flex items-center justify-center group-hover:scale-105 transition-transform`}>
                    <svg class={`w-5 h-5 ${getColorClasses(item.color).text}`} fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
                      <path d={item.icon}/>
                    </svg>
                  </div>
                  <span class="font-medium text-gray-700 dark:text-gray-300">{item.label}</span>
                </div>
                <span class="text-xl font-bold text-gray-900 dark:text-white">{item.value}</span>
              </div>
            {/each}
            
            <div class="mt-8 pt-6 border-t border-gray-200/50 dark:border-gray-800/50">
              <div class="bg-gradient-to-r from-green-50 to-emerald-50 dark:from-green-950/50 dark:to-emerald-950/30 rounded-xl p-4 border border-green-200 dark:border-green-800/50 cursor-pointer hover:scale-105 transition-transform"
                   on:click={() => showNotification('Net profit breakdown opened', 'success')}>
                <div class="flex items-center justify-between">
                  <div class="flex items-center space-x-3">
                    <div class="w-2.5 h-2.5 bg-green-500 rounded-full animate-pulse"></div>
                    <span class="font-semibold text-green-800 dark:text-green-300">Net Profit Today</span>
                  </div>
                  <span class="text-2xl font-bold text-green-700 dark:text-green-400">Rs. 8,250</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    {/if}
  </main>

  <!-- Transaction Modal -->
  {#if showTransactionModal}
    <div class="fixed inset-0 bg-black/50 z-50 flex items-center justify-center p-4" on:click|self={() => showTransactionModal = false}>
      <div class="bg-white dark:bg-gray-900 rounded-2xl p-6 w-full max-w-md border border-gray-200 dark:border-gray-800">
        <div class="flex items-center justify-between mb-6">
          <h3 class="text-xl font-bold text-gray-900 dark:text-white">New Transaction</h3>
          <button on:click={() => showTransactionModal = false} class="w-8 h-8 bg-gray-100 dark:bg-gray-800 hover:bg-gray-200 dark:hover:bg-gray-700 rounded-lg flex items-center justify-center transition-colors">
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/>
            </svg>
          </button>
        </div>
        
        <form on:submit|preventDefault={(e) => {
          const formData = new FormData(e.target);
          const data = {
            type: formData.get('type'),
            amount: formData.get('amount'),
            description: formData.get('description'),
            category: formData.get('category')
          };
          submitTransaction(data);
        }}>
          <div class="space-y-4">
            <div>
              <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Transaction Type</label>
              <select name="type" required class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-lg bg-white dark:bg-gray-800 text-gray-900 dark:text-white focus:ring-2 focus:ring-blue-500 focus:border-transparent">
                <option value="income">Income</option>
                <option value="expense">Expense</option>
              </select>
            </div>
            
            <div>
              <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Amount (Rs.)</label>
              <input type="number" name="amount" required min="0" step="0.01" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-lg bg-white dark:bg-gray-800 text-gray-900 dark:text-white focus:ring-2 focus:ring-blue-500 focus:border-transparent">
            </div>
            
            <div>
              <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Category</label>
              <select name="category" required class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-lg bg-white dark:bg-gray-800 text-gray-900 dark:text-white focus:ring-2 focus:ring-blue-500 focus:border-transparent">
                <option value="sales">Sales</option>
                <option value="marketing">Marketing</option>
                <option value="operations">Operations</option>
                <option value="other">Other</option>
              </select>
            </div>
            
            <div>
              <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Description</label>
              <textarea name="description" rows="3" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-lg bg-white dark:bg-gray-800 text-gray-900 dark:text-white focus:ring-2 focus:ring-blue-500 focus:border-transparent resize-none"></textarea>
            </div>
          </div>
          
          <div class="flex space-x-3 mt-6">
            <button type="button" on:click={() => showTransactionModal = false} class="flex-1 px-4 py-2 bg-gray-100 dark:bg-gray-800 hover:bg-gray-200 dark:hover:bg-gray-700 text-gray-900 dark:text-white rounded-lg font-medium transition-colors">
              Cancel
            </button>
            <button type="submit" disabled={$loadingStore} class="flex-1 px-4 py-2 bg-blue-600 hover:bg-blue-700 disabled:bg-blue-400 text-white rounded-lg font-medium transition-colors flex items-center justify-center space-x-2">
              {#if $loadingStore}
                <div class="w-4 h-4 border-2 border-white/30 border-t-white rounded-full animate-spin"></div>
                <span>Creating...</span>
              {:else}
                <span>Create Transaction</span>
              {/if}
            </button>
          </div>
        </form>
      </div>
    </div>
  {/if}

  <!-- Reports Modal -->
  {#if showReportsModal}
    <div class="fixed inset-0 bg-black/50 z-50 flex items-center justify-center p-4" on:click|self={() => showReportsModal = false}>
      <div class="bg-white dark:bg-gray-900 rounded-2xl p-6 w-full max-w-2xl border border-gray-200 dark:border-gray-800 max-h-[80vh] overflow-y-auto">
        <div class="flex items-center justify-between mb-6">
          <h3 class="text-xl font-bold text-gray-900 dark:text-white">Analytics Reports</h3>
          <button on:click={() => showReportsModal = false} class="w-8 h-8 bg-gray-100 dark:bg-gray-800 hover:bg-gray-200 dark:hover:bg-gray-700 rounded-lg flex items-center justify-center transition-colors">
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/>
            </svg>
          </button>
        </div>
        
        <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
          {#each [
            { title: 'Sales Report', description: 'Detailed sales analytics and trends', icon: 'M12 8c-1.657 0-3 .895-3 2s1.343 2 3 2 3 .895 3 2-1.343 2-3 2', color: 'blue' },
            { title: 'Customer Report', description: 'Customer behavior and demographics', icon: 'M12 4.354a4 4 0 110 5.292M15 21H3v-1a6 6 0 0112 0v1zm0 0h6v-1a6 6 0 00-9-5.197', color: 'emerald' },
            { title: 'Financial Report', description: 'Income, expenses, and profit analysis', icon: 'M17 9V7a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2m2 4h10a2 2 0 002-2v-6a2 2 0 00-2-2H9a2 2 0 00-2 2v6a2 2 0 002 2zm7-5a2 2 0 11-4 0 2 2 0 014 0z', color: 'purple' },
            { title: 'Performance Report', description: 'KPIs and business performance metrics', icon: 'M9 17v-2m3 2v-4m3 4v-6m2 10H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z', color: 'red' }
          ] as report}
            <button class="p-4 bg-gray-50 dark:bg-gray-800 hover:bg-gray-100 dark:hover:bg-gray-700 rounded-xl border border-gray-200 dark:border-gray-700 transition-colors text-left group hover:scale-105 transform duration-200"
                    on:click={() => {
                      showNotification(`${report.title} is being generated...`, 'info');
                      showReportsModal = false;
                    }}>
              <div class="flex items-start space-x-3">
                <div class={`w-10 h-10 ${getColorClasses(report.color).iconBg} rounded-lg flex items-center justify-center group-hover:scale-110 transition-transform`}>
                  <svg class={`w-5 h-5 ${getColorClasses(report.color).text}`} fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
                    <path d={report.icon}/>
                  </svg>
                </div>
                <div class="flex-1">
                  <h4 class="font-semibold text-gray-900 dark:text-white mb-1">{report.title}</h4>
                  <p class="text-sm text-gray-600 dark:text-gray-400">{report.description}</p>
                </div>
              </div>
            </button>
          {/each}
        </div>
      </div>
    </div>
  {/if}

  <!-- Details Modal -->
  {#if showDetailsModal}
    <div class="fixed inset-0 bg-black/50 z-50 flex items-center justify-center p-4" on:click|self={() => showDetailsModal = false}>
      <div class="bg-white dark:bg-gray-900 rounded-2xl p-6 w-full max-w-4xl border border-gray-200 dark:border-gray-800 max-h-[80vh] overflow-y-auto">
        <div class="flex items-center justify-between mb-6">
          <h3 class="text-xl font-bold text-gray-900 dark:text-white">Business Analytics Details</h3>
          <button on:click={() => showDetailsModal = false} class="w-8 h-8 bg-gray-100 dark:bg-gray-800 hover:bg-gray-200 dark:hover:bg-gray-700 rounded-lg flex items-center justify-center transition-colors">
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/>
            </svg>
          </button>
        </div>
        
        <div class="space-y-6">
          <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
            {#each stats as stat}
              <div class="bg-gray-50 dark:bg-gray-800 rounded-xl p-4">
                <div class="flex items-center space-x-3 mb-3">
                  <div class={`w-8 h-8 ${getColorClasses(stat.color).iconBg} rounded-lg flex items-center justify-center`}>
                    <svg class={`w-4 h-4 ${getColorClasses(stat.color).text}`} fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
                      <path d={stat.icon}/>
                    </svg>
                  </div>
                  <h4 class="font-semibold text-gray-900 dark:text-white">{stat.label}</h4>
                </div>
                <p class="text-2xl font-bold text-gray-900 dark:text-white mb-1">{stat.value}</p>
                <p class="text-sm text-gray-600 dark:text-gray-400">Change: {stat.change}</p>
                <div class="mt-3 bg-gray-200 dark:bg-gray-700 rounded-full h-2">
                  <div class={`h-2 rounded-full ${stat.trend === 'up' ? 'bg-green-500' : 'bg-red-500'}`} 
                       style="width: {Math.abs(parseInt(stat.change))}%"></div>
                </div>
              </div>
            {/each}
          </div>
          
          <div class="bg-gradient-to-r from-blue-50 to-purple-50 dark:from-blue-950/20 dark:to-purple-950/20 rounded-xl p-6 border border-blue-200 dark:border-blue-800/30">
            <h4 class="text-lg font-semibold text-gray-900 dark:text-white mb-4">Key Insights for {selectedPeriod}</h4>
            <ul class="space-y-2 text-gray-700 dark:text-gray-300">
              <li class="flex items-center space-x-2">
                <div class="w-2 h-2 bg-green-500 rounded-full"></div>
                <span>Revenue is trending upward with strong customer acquisition</span>
              </li>
              <li class="flex items-center space-x-2">
                <div class="w-2 h-2 bg-blue-500 rounded-full"></div>
                <span>Order volume has increased by 8% compared to last {selectedPeriod}</span>
              </li>
              <li class="flex items-center space-x-2">
                <div class="w-2 h-2 bg-purple-500 rounded-full"></div>
                <span>Customer satisfaction remains high at 4.8/5.0</span>
              </li>
              <li class="flex items-center space-x-2">
                <div class="w-2 h-2 bg-orange-500 rounded-full"></div>
                <span>Expenses decreased by 5%, improving profit margins</span>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  {/if}
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

  @keyframes slideIn {
    from {
      opacity: 0;
      transform: translateX(20px);
    }
    to {
      opacity: 1;
      transform: translateX(0);
    }
  }

  @keyframes float {
    0%, 100% {
      transform: translateY(0px);
    }
    50% {
      transform: translateY(-10px);
    }
  }

  :global(html) {
    scroll-behavior: smooth;
  }

  :global(body) {
    font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', sans-serif;
  }

  /* Custom scrollbar */
  :global(::-webkit-scrollbar) {
    width: 8px;
  }

  :global(::-webkit-scrollbar-track) {
    background: transparent;
  }

  :global(::-webkit-scrollbar-thumb) {
    background: #cbd5e1;
    border-radius: 4px;
  }

  :global(.dark ::-webkit-scrollbar-thumb) {
    background: #475569;
  }

  :global(::-webkit-scrollbar-thumb:hover) {
    background: #94a3b8;
  }

  :global(.dark ::-webkit-scrollbar-thumb:hover) {
    background: #64748b;
  }
</style>