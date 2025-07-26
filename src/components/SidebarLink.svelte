<script>
	import { goto } from '$app/navigation';
	import { page } from '$app/stores';

	export let label;
	export let path;
	export let icon;
	export let sidebarCollapsed = false;
	export let badge = null;

	$: isActive = $page.url.pathname === path;

	function handleClick() {
		goto(path);
	}
</script>

<button
	on:click={handleClick}
	class={`relative w-full flex items-center gap-3 px-3 py-2.5 rounded-lg text-sm font-medium transition-all duration-200 group
		${isActive 
			? 'bg-blue-700 text-white shadow-sm' 
			: 'text-gray-400 hover:text-white hover:bg-gray-800'}
		${sidebarCollapsed ? 'justify-center' : 'justify-start'}`}
	title={sidebarCollapsed ? label : ''}
	aria-label={label}
>
	<!-- Icon -->
	<div class={`flex-shrink-0 ${sidebarCollapsed ? '' : 'w-5 h-5'}`}>
		<svg 
			class="w-5 h-5" 
			fill="none" 
			stroke="currentColor" 
			viewBox="0 0 24 24"
			stroke-width="1.5"
		>
			<path 
				stroke-linecap="round" 
				stroke-linejoin="round" 
				d={icon} 
			/>
		</svg>
	</div>

	<!-- Label -->
	{#if !sidebarCollapsed}
		<span class="flex-1 text-left truncate">{label}</span>

		<!-- Badge -->
		{#if badge}
			<span class={`inline-flex items-center justify-center px-2 py-0.5 text-xs font-medium rounded-full
				${isActive 
					? 'bg-white/20 text-white' 
					: 'bg-gray-700 text-gray-300'}`}>
				{badge}
			</span>
		{/if}

		<!-- Active Indicator -->
		{#if isActive}
			<div class="absolute left-0 top-1/2 -translate-y-1/2 w-1 h-6 bg-white rounded-r-full"></div>
		{/if}
	{:else}
		<!-- Dot indicator if active in collapsed mode -->
		{#if isActive}
			<div class="absolute left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2 w-2 h-2 bg-white rounded-full"></div>
		{/if}

		<!-- Tooltip for collapsed sidebar -->
		<div class="absolute left-full ml-2 px-3 py-2 bg-gray-900 text-white text-sm rounded-lg shadow-lg border border-gray-700
			whitespace-nowrap z-50 opacity-0 invisible group-hover:opacity-100 group-hover:visible transition-all duration-200 pointer-events-none">
			{label}
			{#if badge}
				<span class="ml-2 inline-flex items-center justify-center px-2 py-0.5 text-xs font-medium bg-gray-700 text-gray-300 rounded-full">
					{badge}
				</span>
			{/if}
		</div>
	{/if}
</button>
