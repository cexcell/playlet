<script lang="ts">
  import PlayletLogoDark from "assets/logo-dark.svg.svelte";
  import PlayletLogoLight from "assets/logo-light.svg.svelte";
  import UserIcon from "assets/user.svg.svelte";
  import { PlayletApi } from "lib/Api/PlayletApi";
  import { ExternalControlProtocol } from "lib/Api/ExternalControlProtocol";
  import { appThemeStore, playletStateStore } from "lib/Stores";
  import ThemeSelect from "lib/ThemeToggle.svelte";

  let version;
  let loggedIn = false;
  let auth_url;
  let currentInstance;
  let loggedInInstance;
  let username;
  let appId = "693751";

  playletStateStore.subscribe((value) => {
    version = value?.app?.lib_version ?? "";
    if (value?.app?.id) {
      appId = value?.app?.id;
    }
    if (appId === "dev") {
      version += "-dev";
    }
    if (
      value?.app?.lib_url_type === "custom" &&
      value?.app?.lib_url ===
        "https://github.com/iBicha/playlet/releases/download/canary/playlet-lib.zip"
    ) {
      version += "-canary";
    }
    loggedIn = value?.invidious?.logged_in;
    auth_url = value?.invidious?.auth_url;
    currentInstance = value?.invidious?.current_instance;
    loggedInInstance = value?.invidious?.logged_in_instance;
    username = value?.invidious?.logged_in_username;
  });

  const login = () => {
    if (!auth_url) {
      alert("Error with login, please refresh the page.");
      return;
    }
    window.location = auth_url;
  };
  const logout = async () => {
    await PlayletApi.logout();
    PlayletApi.getState().then((value) => {
      playletStateStore.set(value);
    });
  };
</script>

<div class="navbar bg-base-100 sticky top-0 z-40">
  <div class="flex-1">
    <button on:click={() => ExternalControlProtocol.launchApp(appId)}>
      {#if $appThemeStore === "dark"}
        <PlayletLogoDark />
      {:else}
        <PlayletLogoLight />
      {/if}
    </button>
    <h4 class="label brightness-75">{version}</h4>
  </div>
  <div class="flex-none">
    <ThemeSelect />
    {#if loggedIn && username}
      <div class="badge badge-outline">
        <span>{username}</span>
      </div>
    {/if}
    <div class="dropdown dropdown-end">
      <div tabindex="-1" class="btn btn-ghost btn-circle avatar">
        <div class="w-8 rounded-full">
          <UserIcon />
        </div>
      </div>
      <ul
        tabindex="-1"
        class="menu menu-sm dropdown-content mt-3 p-2 shadow bg-base-100 rounded-box"
      >
        {#if loggedIn}
          <li>
            <div
              class="tooltip tooltip-left"
              data-tip={`Logout from ${loggedInInstance}`}
            >
              <button on:click={logout}>Logout</button>
            </div>
          </li>
        {:else}
          <li>
            <div
              class="tooltip tooltip-left"
              data-tip={`Login using ${currentInstance}`}
            >
              <button on:click={login}>Login to Invidious</button>
            </div>
          </li>
        {/if}
      </ul>
    </div>
  </div>
</div>
