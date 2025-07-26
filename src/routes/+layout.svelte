<script>
	import "../index.css";
	import Header from '../components/Header.svelte';
	import SidebarLink from '../components/SidebarLink.svelte';
	import { onMount } from 'svelte';

	let sidebarCollapsed = false;

	function toggleSidebar() {
		sidebarCollapsed = !sidebarCollapsed;
	}

	onMount(() => {
		const handleResize = () => {
			if (window.innerWidth < 1024) {
				sidebarCollapsed = true;
			}
		};

		window.addEventListener('resize', handleResize);
		handleResize();
		return () => window.removeEventListener('resize', handleResize);
	});
</script>

<!-- Layout container -->
<div class="flex h-screen bg-[#0f0f0f] text-white font-inter overflow-hidden">

	<!-- Sidebar -->
	<aside class={`flex flex-col border-r border-gray-800 shadow-md transition-all duration-300 
		${sidebarCollapsed ? 'w-16 items-center' : 'w-64'}`}>

		<!-- Sidebar Header -->
		<div class="p-4 border-b border-gray-800">
			{#if !sidebarCollapsed}
				<div class="flex items-center gap-3">
					<div class="w-8 h-8 bg-blue-600 rounded-lg flex items-center justify-center">
						<svg class="w-5 h-5 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
							<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z" />
						</svg>
					</div>
					<div>
						<h2 class="font-semibold text-sm">Business Hub</h2>
						<p class="text-xs text-gray-400">Management Suite</p>
					</div>
				</div>
			{:else}
				<div class="w-8 h-8 bg-blue-600 rounded-lg flex items-center justify-center mx-auto">
					<svg class="w-5 h-5 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
						<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z" />
					</svg>
				</div>
			{/if}
		</div>

		<!-- Sidebar Navigation -->
		<nav class="flex-1 p-3 overflow-y-auto">

			{#if !sidebarCollapsed}
				<h3 class="text-xs uppercase font-medium text-gray-400 mb-3 px-2">Operations</h3>
			{/if}
			<div class="space-y-1">
				<SidebarLink label="Dashboard" path="/dashboard" icon="M3 3h18M3 12h18M3 21h18" {sidebarCollapsed} />
				<SidebarLink label="Sales Entry" path="/sales-entry" icon="M12 4v16m8-8H4" {sidebarCollapsed} />
				<SidebarLink label="Purchase Entry" path="/purchase-entry" icon="M5 13l4 4L19 7" {sidebarCollapsed} />
				<SidebarLink label="Savings Entry" path="/savings-entry" icon="M12 8c-4.418 0-8 1.79-8 4s3.582 4 8 4 8-1.79 8-4-3.582-4-8-4z" {sidebarCollapsed} />
				<SidebarLink label="Cash Drawer" path="/drawer" icon="M17 9V7a4 4 0 00-8 0v2H5v12h14V9h-2z" {sidebarCollapsed} />
				<SidebarLink label="Expense Entry" path="/expense-entry" icon="M9 14l6-6M15 14H9V8" {sidebarCollapsed} />
				<SidebarLink label="Order Management" path="/order-list" icon="M3 7h18M3 12h18M3 17h18" {sidebarCollapsed} />
			</div>

			{#if !sidebarCollapsed}
				<h3 class="mt-6 mb-3 text-xs uppercase font-medium text-gray-400 px-2">Catalog & Users</h3>
			{/if}
			<div class="space-y-1">
				<SidebarLink label="Users" path="/users" icon="M5.121 17.804A4 4 0 017 16h10a4 4 0 011.879.804M12 12a4 4 0 100-8 4 4 0 000 8z" {sidebarCollapsed} />
				<SidebarLink label="Wholesale" path="/wholesale" icon="M3 3h18v4H3zM3 9h18v4H3zM3 15h18v4H3z" {sidebarCollapsed} />
				<SidebarLink label="Products" path="/products" icon="M4 6h16M4 10h16M4 14h16M4 18h16" {sidebarCollapsed} />
				<SidebarLink label="School Books" path="/school-books" icon="M4 19.5A2.5 2.5 0 006.5 22h11a2.5 2.5 0 002.5-2.5V4H4v15.5z" {sidebarCollapsed} />
				<SidebarLink label="Published Books" path="/published-books" icon="M12 20h9M12 4h9M3 4h9v16H3z" {sidebarCollapsed} />
				<SidebarLink label="Delivery" path="/delivery" icon="M3 3h18v4H3zM3 9h18v4H3zM3 15h18v4H3z" {sidebarCollapsed} />
			</div>
		</nav>

		<!-- Sidebar Footer -->
		<div class="p-3 border-t border-gray-800">
			{#if !sidebarCollapsed}
				<div class="flex items-center gap-3 p-2 rounded-lg bg-gray-800">
					<div class="w-8 h-8 bg-gradient-to-br from-blue-500 to-purple-600 rounded-full flex items-center justify-center text-white text-sm font-semibold">
						JD
					</div>
					<div>
						<p class="text-sm font-medium truncate">John Doe</p>
						<p class="text-xs text-gray-400 truncate">Administrator</p>
					</div>
				</div>
			{:else}
				<div class="w-8 h-8 bg-gradient-to-br from-blue-500 to-purple-600 rounded-full flex items-center justify-center text-white text-sm font-semibold mx-auto">
					JD
				</div>
			{/if}
		</div>
	</aside>

	<!-- Right Section -->
	<div class="flex flex-col flex-1 min-w-0">
		<Header {toggleSidebar} {sidebarCollapsed} />
		<main class="flex-1 overflow-auto bg-[#121212] p-4">
			<slot />
		</main>
	</div>
</div>

<style>
	:global(body) {
		font-family: 'Inter', sans-serif;
		background-color: #0f0f0f;
		color: white;
	}
</style>
