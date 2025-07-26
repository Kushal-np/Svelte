<script>
    import Header from '../components/Header.svelte';
  import '../index.css'; // global styles here (Tailwind imports etc)
</script>

<div class="min-h-screen bg-gray-900 text-white">
   <Header />
 
  <main class="p-6">
    <slot />
  </main>
</div>