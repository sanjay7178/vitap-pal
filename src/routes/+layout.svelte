<script lang="ts">
  import "../app.css";
  import { University, SquareMenu, Notebook, Menu, Cog } from "lucide-svelte";
  import { page } from "$app/state";
  import { goto } from "$app/navigation";
  import { setContext, untrack } from "svelte";
  import { Info } from "lucide-svelte";
  let { children } = $props();

  const currentPage = () => {
    let path = page.url.pathname;
    let n = path.split("/");
    return "/" + n[1];
  };

  function updatError() {
    $inspect(errors);
    setTimeout(() => {
      if (
        errors.code == "stop" &&
        errors.msg != "PC" &&
        errors.msg != "LI" &&
        errors.msg != "NC"
      ) {
        errors.code = undefined;
      }
    }, 2000);
  }

  let reload = $state({ status: false });
  let errors = $state({ code: undefined, msg: undefined });
  setContext("reload", reload);
  setContext("errors", errors);
  $effect(() => {
    errors.code;
    errors.msg;
    untrack(() => {
      updatError();
    });
  });

  //errors
  // undefined no errors
  // stop : stop all backend functions
  // loginfailed  : logined failed
  // (msg)
  //NE: no internet
  //VE : Vtop offline or  set code to stop
  //NC : useranme or password set code to stop
  //PC : password cahnged

  function tag(val) {
    let k = String(val).charAt(0).toUpperCase() + String(val).slice(1);
    let p = k.split("/");
    if (p.length > 2) {
      return p[0];
    } else {
      return k;
    }
  }
</script>

<div class="drawer">
  <input id="my-drawer" type="checkbox" class="drawer-toggle" />
  <div class="drawer-content bg-base-300 p-2">
    <!-- Navbar -->
    <div class="navbar bg-base-100 shadow-sm ml-2 mr-0 md:ml-4 md:mr-2 my-2 rounded-box pl-3 pr-2 md:pl-5 md:pr-4">
      <div class="navbar-start gap-2">
        <label for="my-drawer" class="btn btn-ghost drawer-button lg:hidden">
          <Menu />
        </label>
        <div class="btn btn-ghost text-xl">VitapPal</div>
      </div>
      <div class="navbar-center">
        <a href="/timetable" class="font-medium">{tag(currentPage().replace("/", ""))}</a>
      </div>
      <div class="navbar-end gap-2">
        <div class="tooltip tooltip-bottom">
          <button class="btn btn-ghost btn-sm btn-circle"><Info class="size-5" /></button>
          <div class="tooltip-content translate-x-[-16vh]">
            <div class="overflow-auto rounded-box">
              <table class="table">
                <thead>
                  <tr>
                    <th></th>
                    <th>Spin</th>
                    <th>No Spin</th>
                  </tr>
                </thead>
                <tbody>
                  <tr>
                    <th><Cog class="size-5 text-info" /></th>
                    <td>logging into vtop</td>
                    <td>-</td>
                  </tr>
                  <tr>
                    <th><Cog class="size-5 text-warning" /></th>
                    <td>Fetching data</td>
                    <td>No Internet</td>
                  </tr>
                  <tr>
                    <th><Cog class="size-5 text-success" /></th>
                    <td>-</td>
                    <td>Everything is upto date</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
        <button
          aria-label="settings"
          onclick={() => goto("/settings")}
          class="btn btn-ghost btn-circle"
        >
          <Cog
            class="{errors.code == undefined &&
            reload.status != true &&
            errors.msg == undefined
              ? 'text-success'
              : errors.code == 'stop' && errors.msg != 'NE'
                ? 'text-error'
                : 'text-warning'}  {reload.status == true
              ? 'animate-spin duration-1000 text-info '
              : ''} "
          />
        </button>
      </div>
    </div>

    <!-- Error messages -->
    <div class="absolute w-full translate-y-[-1vh] z-10">
      {#if errors.msg == "NE"}
        <div class="bg-base-100">
          <span class=" whitespace-nowrap overflow-auto text-xs"
            ><div class=" text-center">
              ⚠️ No Internet Connection. Data might be outdated
            </div></span
          >
        </div>
      {:else if errors.msg == "LI"}
        <div class="bg-base-100">
          <span class=" whitespace-nowrap overflow-auto text-xs"
            ><div class="text-center">
              <span class="loading loading-spinner text-info loading-xs"></span>
              Attempting to log in to VTOP
            </div></span
          >
        </div>
      {:else if reload.status == true}
        <div class=" bg-base-100">
          <span class=" whitespace-nowrap overflow-auto text-xs">
            <div class="text-center">
              <span class="loading loading-spinner text-warning loading-xs"></span>
              Updating data...
            </div></span
          >
        </div>
      {/if}
    </div>

    <!-- Page content -->
    <div class="p-1">
      {@render children()}
    </div>
  </div>
  
  <!-- Drawer side -->
  <div class="drawer-side z-20">
    <label for="my-drawer" aria-label="close sidebar" class="drawer-overlay"></label>
    <ul class="menu p-6 w-64 min-h-full bg-base-200 text-base-content gap-2">
      <li class="menu-title font-bold text-lg mb-4">VitapPal</li>
      <li><a href="/timetable" class="text-base">Timetable</a></li>
      <li><a href="/attendance" class="text-base">Attendance</a></li>
      <li><a href="/utils" class="text-base">Utils</a></li>
      <li><a href="/settings" class="text-base">Settings</a></li>
    </ul>
  </div>
</div>

<!-- Dock remains outside the drawer -->
<div class="dock">
  <button
    onclick={() => goto("/timetable")}
    class={currentPage() != "/timetable" ? "" : "dock-active"}
  >
    <University />
    <span class="dock-label">Timetable</span>
  </button>

  <button
    onclick={() => goto("/attendance")}
    class={currentPage() != "/attendance" ? "" : "dock-active"}
  >
    <Notebook />
    <span class="dock-label">Attendance</span>
  </button>

  <button
    onclick={() => goto("/utils")}
    class={currentPage() != "/utils" ? "" : "dock-active"}
  >
    <SquareMenu />
    <span class="dock-label">Utils</span>
  </button>
</div>
