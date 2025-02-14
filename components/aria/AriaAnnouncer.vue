<script setup lang="ts">
import type { AriaAnnounceType, AriaLive } from '~/composables/aria'
import type { LocaleObject } from '#i18n'

const router = useRouter()
const { t, locale, locales } = useI18n()
const { ariaAnnouncer, announce } = useAriaAnnouncer()

const localeMap = (locales.value as LocaleObject[]).reduce((acc, l) => {
  acc[l.code!] = l.name!
  return acc
}, {} as Record<string, string>)

let ariaLive = $ref<AriaLive>('polite')
let ariaMessage = $ref<string>('')

function onMessage(event: AriaAnnounceType, message?: string) {
  if (event === 'announce')
    ariaMessage = message!
  else if (event === 'mute')
    ariaLive = 'off'
  else
    ariaLive = 'polite'
}

watch(locale, (l, ol) => {
  if (ol) {
    announce(t('a11y.locale_changing', [localeMap[ol] ?? ol]))
    setTimeout(() => {
      announce(t('a11y.locale_changed', [localeMap[l] ?? l]))
    }, 1000)
  }
}, { immediate: true })

onMounted(() => {
  ariaAnnouncer.on(onMessage)
  router.beforeEach(() => {
    announce(t('a11y.loading_page'))
  })
  router.afterEach((to, from) => {
    from && setTimeout(() => {
      requestAnimationFrame(() => {
        const title = document.title.trim().split('|')
        announce(t('a11y.route_loaded', [title[0]]))
      })
    }, 512)
  })
})
</script>

<template>
  <p sr-only role="status" :aria-live="ariaLive">
    {{ ariaMessage }}
  </p>
</template>
