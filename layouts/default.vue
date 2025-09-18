<template>
  <div class="slidev-layout h-full">
    <div class="my-auto">
      <slot />
    </div>
    <div class="bg-rpred-dark w-2 h-24 position-absolute top-0 left-0"/>
    <div class="position-absolute right-0 bottom-0 p-4
      font-semibold">redpencil<span class="text-rpred">.</span>io</div>
  </div>
</template>
