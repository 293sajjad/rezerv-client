<script lang="ts">
  import Cookies from "js-cookie";
  import { onMount } from "svelte";
  import toast, { Toaster } from "svelte-french-toast";

  let token: string | undefined;

  onMount(() => {
    token = Cookies.get("token");
  });

  const handleLogout = () => {
    Cookies.remove("token");
    token = undefined;
    toast.success("خروج موفقیت آمیز بود");
    setTimeout(() => {
      window.location.href = "/auth";
    }, 200);
  };
</script>

<Toaster />
<header class="bg-blue-600 p-4 flex justify-between items-center">
  <h1 class="text-white text-xl">Rezerv</h1>
  <div>
    {#if token}
      <button
        class="bg-yellow-400 text-white py-2 px-4 rounded hover:bg-yellow-500 transition duration-200"
        on:click={handleLogout}
      >
        خروج
      </button>
    {:else}
      <button
        class="bg-green-500 text-white py-2 px-4 rounded hover:bg-green-600 transition duration-200"
        on:click={() => (window.location.href = "/auth")}
      >
        احراز هویت
      </button>
    {/if}
  </div>
</header>
