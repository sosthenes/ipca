<template>
  <CFooter :fixed="false">
    <div>
      <a href="https://coreui.io" target="_blank">@Dev</a>
      <span class="ml-1">&copy; {{new Date().getFullYear()}} .</span>
    </div>
    <div class="ml-auto">
      <span class="mr-1" target="_blank">Powered by</span>
      Sosthenes!
    </div>
  </CFooter>
</template>

<script>
export default {
  name: 'TheFooter'
}
</script>
