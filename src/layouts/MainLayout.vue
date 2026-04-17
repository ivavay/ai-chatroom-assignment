<template>
  <q-layout view="lHh Lpr lFf">


    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script setup>

const linksList = [
  {
    title: 'Home',
    caption: 'Home',
    icon: 'home',
    to: '/'
  },
  {
    title: 'Colors',
    caption: 'Color palette and utilities',
    icon: 'palette',
    to: '/colors'
  },
  {
    title: 'Typography',
    caption: 'Typography',
    icon: 'text_fields',
    to: '/typography'
  },
  {
    title: 'Docs',
    caption: 'quasar.dev',
    icon: 'school',
    target: '_blank',
    href: 'https://quasar.dev'
  },
  {
    title: 'Github',
    caption: 'github.com/quasarframework',
    icon: 'code',
    target: '_blank',
    href: 'https://github.com/quasarframework'
  },
]
</script>
