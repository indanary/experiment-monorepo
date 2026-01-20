<template>
  <p class="text-caption">Today (from shared utils): {{ today }}</p>

  <q-btn color="primary" :label="primaryButton.label" />
</template>

<script setup lang="ts">
import { formatDate } from '@experiment-monorepo/shared';
import { Button } from '@experiment-monorepo/ui';

const today = formatDate(new Date());
const primaryButton = Button('Submit');

console.log('Web App');
console.log('Button from shared UI:', primaryButton);
</script>
