<template>
  <div role="alert">

    <div class="CSuccess-border p-2">
      
      <div class="flex CSuccess-flex py-1">
        <img class="mr-2" src="@/assets/success_icon.png">
        <p class="text-center">{{msg1}}</p>
      </div>
      
      <p class="text-center CSuccess-msg mt-2 mb-1">{{ msg2 }}</p>
    </div>

  </div>
</template>

<script>
export default {
  name: "CSuccess",
  props: {
    msg1: String,
    msg2: String,
  },
};
</script>
<style scoped lang="scss">
.CSuccess-border {
  border-width: 3px;
  border-radius: 8px;
  border-style: dashed;
  border-color: var(--RichGreen__RichGreen);
  background-color: var(--Snow__DarkToneInk);
  .CSuccess-flex {
    align-items: center;
    justify-content: center;
    & p {
      font-weight: bold;
      font-size: 24px;
      color: var(--RichGreen__RichGreen);
    }
    & img {
      height: 28px;
    }
  }
  .CSuccess-msg {
    color: var(--RichGreen__RichGreen);
  }
}
</style>