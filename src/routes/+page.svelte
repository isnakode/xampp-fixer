<script lang="ts">
  import { path } from "@tauri-apps/api";
  import { open } from "@tauri-apps/plugin-dialog";
  import {
    copyFile,
    exists,
    mkdir,
    readDir,
    remove,
  } from "@tauri-apps/plugin-fs";
  import { onMount } from "svelte";

  let choosenDir = $state("C:\\xampp");

  let isValidXammpPath = $state(false);

  onMount(() => {
    checkMysqlPath();
  });

  const checkMysqlPath = async () => {
    const mysqlPath = await path.join(choosenDir, "mysql");
    isValidXammpPath = await exists(mysqlPath);
  };

  const pickDir = async () => {
    const dir = await open({ directory: true });
    if (dir) {
      choosenDir = dir;
      checkMysqlPath();
    }
  };

  const copyDir = async (src: string, to: string) => {
    await mkdir(to);
    const entries = await readDir(src);
    for (let i = 0; i < entries.length; i++) {
      const entry = entries[i];
      if (entry.isFile) {
        await copyFile(
          await path.join(src, entry.name),
          await path.join(to, entry.name),
        );
      } else {
        await copyDir(
          await path.join(src, entry.name),
          await path.join(to, entry.name),
        );
      }
    }
  };

  const fixXampp = async () => {
    await copyDir(
      await path.join(choosenDir, "mysql", "data"),
      await path.join(choosenDir, "mysql", "data_old"),
    );
    await copyDir(
      await path.join(choosenDir, "mysql", "backup", "mysql"),
      await path.join(choosenDir, "mysql", "data", "mysql"),
    );
    await copyDir(
      await path.join(choosenDir, "mysql", "backup", "performance_schema"),
      await path.join(choosenDir, "mysql", "data", "performance_schema"),
    );
    await copyDir(
      await path.join(choosenDir, "mysql", "backup", "phpmyadmin"),
      await path.join(choosenDir, "mysql", "data", "phpmyadmin"),
    );
    await remove(await path.join(choosenDir, "mysql", "data_old"), {
      recursive: true,
    });
  };
</script>

<main class="p-4 flex flex-col gap-3">
  <div>
    <p class="font-semibold text-lg">Pilih Folder</p>
    <p class="text-sm text-gray-500 dark:text-gray-400">
      Pilih lokasi folder XAMPP yang ingin kamu perbaiki
    </p>
  </div>
  <div class="input input-md w-full rounded-md">{choosenDir}</div>

  <div class="flex gap-2">
    <button class="btn btn-primary grow rounded-md text-white" onclick={pickDir}
      ><svg
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
        stroke-width="1.5"
        stroke="currentColor"
        class="size-5"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          d="M2.25 12.75V12A2.25 2.25 0 0 1 4.5 9.75h15A2.25 2.25 0 0 1 21.75 12v.75m-8.69-6.44-2.12-2.12a1.5 1.5 0 0 0-1.061-.44H4.5A2.25 2.25 0 0 0 2.25 6v12a2.25 2.25 0 0 0 2.25 2.25h15A2.25 2.25 0 0 0 21.75 18V9a2.25 2.25 0 0 0-2.25-2.25h-5.379a1.5 1.5 0 0 1-1.06-.44Z"
        />
      </svg>
      Pilih folder
    </button>

    <button
      disabled={!isValidXammpPath}
      class="btn btn-outline btn-primary grow rounded-md"
      onclick={fixXampp}
    >
      <svg
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
        stroke-width="1.5"
        stroke="currentColor"
        class="size-5"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          d="M11.42 15.17 17.25 21A2.652 2.652 0 0 0 21 17.25l-5.877-5.877M11.42 15.17l2.496-3.03c.317-.384.74-.626 1.208-.766M11.42 15.17l-4.655 5.653a2.548 2.548 0 1 1-3.586-3.586l6.837-5.63m5.108-.233c.55-.164 1.163-.188 1.743-.14a4.5 4.5 0 0 0 4.486-6.336l-3.276 3.277a3.004 3.004 0 0 1-2.25-2.25l3.276-3.276a4.5 4.5 0 0 0-6.336 4.486c.091 1.076-.071 2.264-.904 2.95l-.102.085m-1.745 1.437L5.909 7.5H4.5L2.25 3.75l1.5-1.5L7.5 4.5v1.409l4.26 4.26m-1.745 1.437 1.745-1.437m6.615 8.206L15.75 15.75M4.867 19.125h.008v.008h-.008v-.008Z"
        />
      </svg>

      Perbaiki
    </button>
  </div>
  <div
    role="alert"
    class="alert text-xs alert-soft {isValidXammpPath
      ? 'alert-success border-success!'
      : 'alert-error border-error!'}"
  >
    <span>
      Folder XAMPP {isValidXammpPath
        ? "valid, tekan perbaiki untuk mulai perbaikan"
        : "tidak valid, Folder MySQL tidak ditemukan"}
    </span>
  </div>
</main>
