<template>
  <div>
    <p>Parameter : {{ parameter }}</p>
  </div>
</template>

<script>
export default {
   layout : 'main-layout',
  // validate 는 data 보다 먼저 정의 되어 있어야 함
  validate(params) {
    return /^\w+&/.test(this.$route.params.paramNum)
  },
  data() {
    return {parameter : this.$route.params.paramNum}
  },

}
</script>
