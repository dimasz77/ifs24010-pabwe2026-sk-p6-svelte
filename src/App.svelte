<script>
  import { onMount } from "svelte";
  import { currentPath, navigate, resolveRoute } from "./router.js";
  import { authProfile, isPreload, loadProfile } from "./features/auth/states/authStore.js";
  import AuthLayout from "./features/auth/layouts/AuthLayout.svelte";
  import LoginPage from "./features/auth/pages/LoginPage.svelte";
  import LinkTreeLayout from "./features/linktrees/layouts/LinkTreeLayout.svelte";

  const pageLoaders = {
    login: () => Promise.resolve({ default: LoginPage }),
    register: () => import("./features/auth/pages/RegisterPage.svelte"),
    home: () => import("./features/linktrees/pages/HomePage.svelte"),
    detail: () => import("./features/linktrees/pages/DetailPage.svelte"),
    users: () => import("./features/users/pages/UsersPage.svelte"),
    profile: () => import("./features/users/pages/ProfilePage.svelte"),
    notFound: () => import("./features/common/pages/NotFoundPage.svelte"),
  };

  const route = $derived(resolveRoute($currentPath));

  onMount(() => {
    loadProfile();
  });

  // Penjaga rute: tamu diarahkan ke login, pengguna login diarahkan ke beranda.
  $effect(() => {
    if ($isPreload) return;
    if (route.layout === "auth" && $authProfile) navigate("/", true);
    if (route.layout === "app" && !$authProfile) navigate("/auth/login", true);
  });
</script>

{#if $isPreload}
  <main class="grid min-h-screen place-items-center">
    <h1 class="font-display text-xl font-bold text-muted">Memuat...</h1>
  </main>
{:else if (route.layout === "auth" && $authProfile) || (route.layout === "app" && !$authProfile)}
  <main class="grid min-h-screen place-items-center">
    <h1 class="font-display text-xl font-bold text-muted">Memuat...</h1>
  </main>
{:else}
  {#await pageLoaders[route.name]()}
    <main class="grid min-h-screen place-items-center">
      <h1 class="font-display text-xl font-bold text-muted">Memuat...</h1>
    </main>
  {:then pageModule}
    {@const Page = pageModule.default}
    {#if route.layout === "auth" && !$authProfile}
      <AuthLayout>
        <Page />
      </AuthLayout>
    {:else if route.layout === "app" && $authProfile}
      <LinkTreeLayout>
        <Page {...route.params} />
      </LinkTreeLayout>
    {:else if route.layout === "plain"}
      <Page />
    {/if}
  {:catch error}
    <main class="grid min-h-screen place-items-center">
      <div class="text-center">
        <h1 class="font-display text-xl font-bold text-muted">Halaman gagal dimuat.</h1>
        <p class="mt-2 text-sm text-muted">{error instanceof Error ? error.message : String(error)}</p>
      </div>
    </main>
  {/await}
{/if}
