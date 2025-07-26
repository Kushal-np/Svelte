<!-- Header.svelte -->
<script>
  import { goto } from "$app/navigation";
  export let toggleSidebar;
  export let sidebarCollapsed;

  let searchQuery = '';
  let showUserMenu = false;
  let showNotifications = false;

  function handleSearch() {
    if (searchQuery.trim()) {
      console.log('Searching for:', searchQuery);
    }
  }

  function navigateTo(path) {
    goto(path);
  }

  function handleKeydown(event) {
    if ((event.metaKey || event.ctrlKey) && event.key === 'k') {
      event.preventDefault();
      document.querySelector('input[type="search"]')?.focus();
    }
  }

  function handleClickOutside() {
    showUserMenu = false;
    showNotifications = false;
  }
</script>

<svelte:window on:keydown={handleKeydown} on:click={handleClickOutside} />

<header class="h-14 bg-neutral-900 border-b border-neutral-700 flex items-center justify-between px-4 shadow-sm text-white z-10">
  <!-- Left: Sidebar Toggle + Breadcrumb -->
  <div class="flex items-center gap-4">
    <button
      on:click={toggleSidebar}
      class="p-2 hover:bg-neutral-800 rounded-lg transition-colors duration-200 text-neutral-400 hover:text-white"
      aria-label="Toggle sidebar"
    >
      <svg class="w-5 h-5" stroke="currentColor" fill="none" viewBox="0 0 24 24">
        <path stroke-width="2" stroke-linecap="round" stroke-linejoin="round" d="M4 6h16M4 12h16M4 18h16" />
      </svg>
    </button>

    <div class="hidden md:flex items-center gap-2 text-sm text-neutral-300">
      <button on:click={() => navigateTo('/dashboard')} class="hover:text-white transition">
        Dashboard
      </button>
      <svg class="w-4 h-4 text-neutral-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-width="2" stroke-linecap="round" stroke-linejoin="round" d="M9 5l7 7-7 7" />
      </svg>
      <span class="font-semibold text-white">Overview</span>
    </div>
  </div>

  <!-- Center: Search -->
  <div class="flex-1 max-w-md mx-4">
    <div class="relative">
      <div class="absolute inset-y-0 left-0 pl-3 flex items-center text-neutral-500">
        <svg class="w-4 h-4" stroke="currentColor" fill="none" viewBox="0 0 24 24">
          <path stroke-width="2" stroke-linecap="round" stroke-linejoin="round" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
        </svg>
      </div>
      <input
        type="search"
        bind:value={searchQuery}
        on:keydown={(e) => e.key === 'Enter' && handleSearch()}
        placeholder="Search everything..."
        class="w-full pl-10 pr-12 py-2 bg-neutral-800 border border-neutral-700 text-sm rounded-lg placeholder-neutral-500 text-white focus:outline-none focus:ring-2 focus:ring-blue-500"
      />
      <div class="absolute inset-y-0 right-0 pr-3 flex items-center">
        <kbd class="hidden sm:inline-flex items-center px-2 py-1 text-xs text-neutral-500 bg-neutral-700 rounded select-none">⌘K</kbd>
      </div>
    </div>
  </div>

  <!-- Right: Actions -->
  <div class="flex items-center gap-2">
    <!-- Quick Actions -->
    <div class="hidden lg:flex gap-1">
      <button class="p-2 hover:bg-neutral-800 rounded-lg transition text-neutral-400 hover:text-white" title="New Sale">
        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-width="2" stroke-linecap="round" stroke-linejoin="round" d="M12 6v6m0 0v6m0-6h6m-6 0H6" />
        </svg>
      </button>
      <button class="p-2 hover:bg-neutral-800 rounded-lg transition text-neutral-400 hover:text-white" title="Settings">
        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-width="2" stroke-linecap="round" stroke-linejoin="round"
            d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0...M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
        </svg>
      </button>
    </div>

    <!-- Notifications -->
    <div class="relative">
      <button on:click|stopPropagation={() => showNotifications = !showNotifications}
        class="relative p-2 hover:bg-neutral-800 rounded-lg transition text-neutral-400 hover:text-white">
        <svg class="w-5 h-5" stroke="currentColor" fill="none" viewBox="0 0 24 24">
          <path stroke-width="2" stroke-linecap="round" stroke-linejoin="round"
            d="M15 17h5l-1.405-1.405A2...m0-6H6" />
        </svg>
        <span class="absolute -top-1 -right-1 w-3 h-3 bg-red-500 rounded-full ring-2 ring-neutral-900"></span>
      </button>
      {#if showNotifications}
        <div class="absolute right-0 mt-2 w-80 bg-neutral-900 border border-neutral-700 rounded-lg shadow-lg z-50">
          <div class="px-4 py-2 border-b border-neutral-700 font-semibold text-white text-sm">Notifications</div>
          <div class="max-h-64 overflow-y-auto">
            <div class="px-4 py-3 hover:bg-neutral-800 cursor-pointer text-white">
              <div class="text-sm font-medium">New order received</div>
              <div class="text-xs text-neutral-400">Order #1234 - $150.00 • 2 mins ago</div>
            </div>
          </div>
          <div class="px-4 py-2 border-t border-neutral-700">
            <button class="text-xs text-blue-400 hover:underline">View all notifications</button>
          </div>
        </div>
      {/if}
    </div>

    <!-- User Menu -->
    <div class="relative">
      <button on:click|stopPropagation={() => showUserMenu = !showUserMenu}
        class="flex items-center gap-2 p-1 hover:bg-neutral-800 rounded-lg transition text-white">
        <div class="w-8 h-8 bg-gradient-to-br from-blue-500 to-purple-600 rounded-full flex items-center justify-center text-sm font-medium">
          JD
        </div>
        <svg class="w-4 h-4 text-neutral-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-width="2" stroke-linecap="round" stroke-linejoin="round" d="M19 9l-7 7-7-7" />
        </svg>
      </button>
      {#if showUserMenu}
        <div class="absolute right-0 mt-2 w-64 bg-neutral-900 border border-neutral-700 rounded-lg shadow-lg z-50">
          <div class="px-4 py-3 border-b border-neutral-700 flex items-center gap-3">
            <div class="w-10 h-10 bg-gradient-to-br from-blue-500 to-purple-600 rounded-full flex items-center justify-center text-white font-medium">JD</div>
            <div class="text-sm text-white">
              <p class="font-medium">John Doe</p>
              <p class="text-xs text-neutral-400">john@company.com</p>
            </div>
          </div>
          <div class="py-1 text-white text-sm">
            <button class="w-full px-4 py-2 text-left hover:bg-neutral-800 flex gap-3 items-center">Profile Settings</button>
            <button class="w-full px-4 py-2 text-left hover:bg-neutral-800 flex gap-3 items-center">Preferences</button>
            <button class="w-full px-4 py-2 text-left hover:bg-neutral-800 flex gap-3 items-center">Help & Support</button>
            <div class="border-t border-neutral-700 my-1"></div>
            <button class="w-full px-4 py-2 text-left text-red-500 hover:bg-neutral-800 flex gap-3 items-center">Sign Out</button>
          </div>
        </div>
      {/if}
    </div>
  </div>
</header>
