<script lang="ts">
  import toast, { Toaster } from "svelte-french-toast";
  import Cookies from "js-cookie";
  import { onMount } from "svelte";

  let formType = "login";
  let token;
  onMount(async () => {
    token = await Cookies.get("token");
    if (token) {
      window.location.href = "/";
    }
  });
  function showForm(type: "login" | "register") {
    formType = type;
  }

  const formLogin: { email: string; password: string } = {
    email: "",
    password: "",
  };

  const formRegister: { email: string; password: string; name: string } = {
    email: "",
    password: "",
    name: "",
  };

  const handleLogin = async () => {
    try {
      const response = await fetch("http://localhost:3000/auth/login", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(formLogin),
      });
      const data = await response.json();
      if (response.ok) {
        toast.success("ورود موفقیت آمیز بود");
        Cookies.set("token", data.token);
        console.log(data);
        setTimeout(() => {
          window.location.href = "/";
        }, 1000);
      } else {
        toast.error(data.errorMessage || "اطلاعات وارد شده صحیح نیست");
      }
    } catch (error) {
      toast.error("مشکلی پیش آمد");
      console.error(error);
    }
  };

  const handleRegister = async () => {
    if (formRegister.password.length < 8) {
      toast.error("رمز عبور باید حداقل 8 کاراکتر باشد");
      return;
    }

    if (!/^[a-zA-Z0-9]+$/.test(formRegister.password)) {
      toast.error("رمز عبور باید انگلیسی باشد و شامل حروف و اعداد باشد");
      return;
    }

    try {
      const response = await fetch("http://localhost:3000/auth/register", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(formRegister),
      });
      const data = await response.json();
      if (response.ok) {
        toast.success("ثبت نام موفقیت آمیز بود");
        Cookies.set("token", data.token);
        console.log(data);
        setTimeout(() => {
          window.location.href = "/";
        }, 500);
      } else {
        toast.error(data.errorMessage || "کاربر با این ایمیل از قبل وجود دارد");
      }
    } catch (error) {
      toast.error("مشکلی پیش آمد");
      console.error(error);
    }
  };
</script>

<svelte:head>
  <title>Rezerv - {formType}</title>
</svelte:head>

<Toaster />
<div class="min-h-screen flex items-center justify-center bg-gray-100">
  <div class="bg-white p-8 rounded-lg shadow-lg w-full max-w-md">
    <div class="toggle-container">
      <button
        type="button"
        class="toggle-button {formType === 'login' ? 'active' : ''}"
        on:click={() => showForm("login")}
      >
        ورود
      </button>
      <button
        type="button"
        class="toggle-button {formType === 'register' ? 'active' : ''}"
        on:click={() => showForm("register")}
      >
        ثبت نام
      </button>
    </div>
    {#if formType === "login"}
      <div>
        <h2 class="text-2xl font-bold mb-6 text-center">ورود</h2>
        <div class="mb-4">
          <label
            class="block text-gray-700 mb-2"
            for="login-email"
            style="direction: rtl;">ایمیل</label
          >
          <input
            class="w-full px-3 py-2 border rounded-lg focus:outline-none focus:border-blue-500"
            type="email"
            id="login-email"
            required
            bind:value={formLogin.email}
          />
        </div>
        <div class="mb-4">
          <label
            class="block text-gray-700 mb-2"
            for="login-password"
            style="direction: rtl;">رمز عبور</label
          >
          <input
            class="w-full px-3 py-2 border rounded-lg focus:outline-none focus:border-blue-500"
            type="password"
            id="login-password"
            required
            bind:value={formLogin.password}
          />
        </div>
        <button
          class="w-full bg-blue-500 text-white py-2 rounded-lg hover:bg-blue-600 transition duration-200"
          on:click={handleLogin}
          type="button">ورود</button
        >
      </div>
    {:else}
      <div>
        <h2 class="text-2xl font-bold mb-6 text-center">ثبت نام</h2>
        <div class="mb-4">
          <label
            class="block text-gray-700 mb-2"
            for="register-name"
            style="direction: rtl;">نام</label
          >
          <input
            class="w-full px-3 py-2 border rounded-lg focus:outline-none focus:border-blue-500"
            type="text"
            id="register-name"
            required
            bind:value={formRegister.name}
          />
        </div>
        <div class="mb-4">
          <label
            class="block text-gray-700 mb-2"
            for="register-email"
            style="direction: rtl;">ایمیل</label
          >
          <input
            class="w-full px-3 py-2 border rounded-lg focus:outline-none focus:border-blue-500"
            type="email"
            id="register-email"
            required
            bind:value={formRegister.email}
          />
        </div>
        <div class="mb-4">
          <label
            class="block text-gray-700 mb-2"
            for="register-password"
            style="direction: rtl;">رمز عبور</label
          >
          <input
            class="w-full px-3 py-2 border rounded-lg focus:outline-none focus:border-blue-500"
            type="password"
            id="register-password"
            required
            bind:value={formRegister.password}
          />
        </div>
        <button
          class="w-full bg-blue-500 text-white py-2 rounded-lg hover:bg-blue-600 transition duration-200"
          on:click={handleRegister}
          type="button">ثبت نام</button
        >
      </div>
    {/if}
  </div>
</div>

<style>
  .toggle-container {
    display: flex;
    justify-content: center;
    margin-bottom: 2rem;
  }
  .toggle-button {
    margin: 0 1rem;
    padding: 0.5rem 1rem;
    border-radius: 0.375rem;
    background-color: #f3f4f6;
    cursor: pointer;
    border: none;
  }
  .toggle-button.active {
    background-color: #3b82f6;
    color: white;
  }
</style>
