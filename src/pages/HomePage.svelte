<script lang="ts">
  import Cookies from "js-cookie";
  import { onMount } from "svelte";
  import toast from "svelte-french-toast";
  import Header from "../lib/Header.svelte";
  import Flatpickr from "svelte-flatpickr";
  import "flatpickr/dist/flatpickr.css";
  import { Persian } from "flatpickr/dist/l10n/fa";

  let token: string | undefined;
  let rooms: { id: number; number: string; type: string }[] = [];
  let selectedDate: Date | undefined;
  let availableRooms: { id: number; number: string; type: string }[] = [];
  let showBookingForm = false;
  let showAvailableRooms = false;

  onMount(() => {
    token = Cookies.get("token");
  });

  const fetchRooms = async () => {
    try {
      const response = await fetch("http://localhost:3000/room/all");
      const data = await response.json();
      if (response.ok) {
        rooms = data.rooms;
      } else {
        handleError(data.errorMessage);
      }
    } catch (error) {
      handleError("مشکلی پیش آمد");
    }
  };

  const handleError = (message: string) => {
    toast.error(message);
    console.error(message);
  };

  const handleReserve = () => {
    showBookingForm = true;
    fetchRooms();
  };

  const checkAvailableRooms = async () => {
    if (!selectedDate) {
      toast.error("لطفا تاریخ را انتخاب کنید");
      return;
    }

    try {
      const headers = new Headers({
        "Content-Type": "application/json",
      });

      if (token) {
        headers.append("Authorization", token);
      }
      // Create a copy of the selectedDate to avoid mutating the original date
      const bookingDate = new Date(selectedDate);
      bookingDate.setDate(bookingDate.getDate() + 1);
      const date = bookingDate.toISOString().split("T")[0];
      const response = await fetch(
        "http://localhost:3000/room/available-rooms",
        {
          method: "POST",
          headers,
          body: JSON.stringify({
            date,
          }),
        }
      );

      const data = await response.json();
      if (response.ok) {
        availableRooms = data.rooms;
        showAvailableRooms = data.rooms.length > 0;
        if (!showAvailableRooms) toast.error("اتاقی موجود نیست");
      } else {
        handleError(data.errorMessage);
      }
    } catch (error) {
      handleError("مشکلی پیش آمد");
    }
  };

  const bookRoom = async (roomNumber: string) => {
    if (!selectedDate) return;

    // Create a copy of the selectedDate to avoid mutating the original date
    const bookingDate = new Date(selectedDate);
    bookingDate.setDate(bookingDate.getDate() + 1);
    const date = bookingDate.toISOString().split("T")[0];

    try {
      const headers = new Headers({
        "Content-Type": "application/json",
      });

      if (token) {
        headers.append("Authorization", token);
      }

      const response = await fetch("http://localhost:3000/booking/add", {
        method: "POST",
        headers,
        body: JSON.stringify({ roomNumber, date }),
      });

      const data = await response.json();
      if (response.ok) {
        toast.success("ثبت اتاق موفقیت آمیز بود");
        resetBookingForm();
      } else {
        handleError(data.errorMessage);
      }
    } catch (error) {
      handleError("مشکلی پیش آمد");
    }
  };

  const resetBookingForm = () => {
    showBookingForm = false;
    showAvailableRooms = false;
    selectedDate = undefined;
  };
</script>

<Header />
<svelte:head>
  <title>Rezerv - Home</title>
</svelte:head>
<div class="min-h-screen flex items-center justify-center bg-gray-100 py-12">
  {#if token}
    <div class="bg-white p-8 rounded-lg shadow-lg w-full max-w-md text-center">
      <h2 class="text-3xl font-bold mb-6 text-gray-800">! خوش‌آمدید</h2>
      <button
        class="bg-blue-500 text-white py-3 px-6 rounded-lg shadow-md hover:bg-blue-600 transition duration-200"
        on:click={handleReserve}
      >
        رزرو اتاق
      </button>

      {#if showBookingForm}
        <div class="mt-6">
          <Flatpickr
            options={{
              enableTime: true,
              dateFormat: "Y-m-d",
              minDate: "today",
              locale: Persian,
            }}
            bind:value={selectedDate}
            class="bg-gray-100 p-3 rounded-lg w-full text-center"
            placeholder="تاریخ را انتخاب کنید"
          />
          <button
            class="bg-green-500 text-white py-3 px-6 rounded-lg shadow-md hover:bg-green-600 transition duration-200 mt-6"
            on:click={checkAvailableRooms}
          >
            بررسی اتاق‌های موجود
          </button>
        </div>
      {/if}

      {#if showAvailableRooms}
        <div class="mt-6 space-y-4">
          {#each availableRooms as room}
            <div
              class="bg-white p-6 rounded-lg shadow-md border border-gray-200"
            >
              <p class="text-xl font-semibold text-gray-700">
                شماره اتاق: {room.number}
              </p>
              <p class="text-lg text-gray-600">نوع: {room.type}</p>
              <button
                class="bg-blue-500 text-white py-2 px-4 rounded-lg shadow-md hover:bg-blue-600 transition duration-200 mt-4"
                on:click={() => bookRoom(room.number)}
              >
                رزرو این اتاق
              </button>
            </div>
          {/each}
        </div>
      {/if}
    </div>
  {:else}
    <div class="bg-white p-8 rounded-lg shadow-lg w-full max-w-md text-center">
      <h2 class="text-3xl font-bold mb-6 text-gray-800">خطا</h2>
      <p class="mb-6 text-gray-600">
        برای رزرو اتاق ابتدا باید احراز هویت کنید.
      </p>
      <button
        class="bg-green-500 text-white py-3 px-6 rounded-lg shadow-md hover:bg-green-600 transition duration-200"
        on:click={() => (window.location.href = "/auth")}
      >
        احراز هویت
      </button>
    </div>
  {/if}
</div>
