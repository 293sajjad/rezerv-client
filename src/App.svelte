<script lang="ts">
  import { Router, Route } from "svelte-routing";
  import AuthPage from "./pages/AuthPage.svelte";
  import HomePage from "./pages/HomePage.svelte";
  const message = "Hello, world!";
  export let url = "";
</script>

<Router {url}>
  <div class="bg-gray-100">
    <Route path="/" component={HomePage} />
    <Route path="/auth" component={AuthPage} />
  </div>
</Router>
