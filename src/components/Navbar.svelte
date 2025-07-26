<script>
    import { goto } from "$app/navigation";
    let searchQuery = '';

    function handleSearch() {
        if (searchQuery.trim()) {
            console.log('Searching for:', searchQuery);
            // Add your actual search logic here
        }
    }

    function navigateTo(path) {
        goto(path);
    }
</script>

<header class="bg-black border-b border-gray-800 sticky top-0 z-50 shadow-md">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="flex justify-between items-center h-16">
            <!-- Logo -->
            <button 
                on:click={() => navigateTo('/')}
                class="flex items-center space-x-3 group"
                aria-label="Go to home"
            >
                <div class="w-10 h-10 bg-gradient-to-br from-blue-600 to-indigo-700 rounded-xl flex items-center justify-center shadow-md transform group-hover:scale-105 transition-transform duration-200">
                    <span class="text-white font-bold text-lg select-none">L</span>
                </div>
                <div class="hidden sm:block">
                    <h1 class="text-xl font-bold bg-gradient-to-r from-white to-gray-400 bg-clip-text text-transparent select-none">
                        YourBrand
                    </h1>
                    <p class="text-xs text-gray-500 -mt-1 select-none">Premium Solutions</p>
                </div>
            </button>

            <!-- Search Bar -->
            <div class="flex-1 max-w-lg mx-8">
                <div class="relative w-full">
                    <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none text-gray-500">
                        🔍
                    </div>
                    <input
                        type="text"
                        bind:value={searchQuery}
                        on:keydown={(e) => e.key === 'Enter' && handleSearch()}
                        placeholder="Search products, articles, or anything..."
                        class="block w-full pl-10 pr-4 py-2.5 bg-gray-900 text-white border border-gray-700 rounded-xl placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transition duration-200"
                        aria-label="Search"
                    />
                    <button
                        on:click={handleSearch}
                        class="absolute inset-y-0 right-0 pr-3 flex items-center text-gray-400 hover:text-blue-400 transition duration-200"
                        aria-label="Execute search"
                    >
                        <kbd class="px-2 py-1 text-xs font-semibold text-gray-300 bg-gray-800 border border-gray-600 rounded select-none">⌘K</kbd>
                    </button>
                </div>
            </div>
        </div>
    </div>

    <!-- Mobile Search Bar -->
    <div class="md:hidden px-4 pb-4">
        <div class="relative">
            <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none text-gray-500">
                🔍
            </div>
            <input
                type="text"
                bind:value={searchQuery}
                on:keydown={(e) => e.key === 'Enter' && handleSearch()}
                placeholder="Search..."
                class="block w-full pl-10 pr-4 py-2.5 bg-gray-900 text-white border border-gray-700 rounded-xl placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transition duration-200"
                aria-label="Search"
            />
        </div>
    </div>
</header>

<style>
    * {
        transition-property: color, background-color, border-color, box-shadow, transform;
        transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
    }
</style>
