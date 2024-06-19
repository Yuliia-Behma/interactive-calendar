<script setup lang="ts">
const pageTitle = useState("page-title");
onMounted(() => {
  pageTitle.value = "Calendar";
});
</script>

<template>
  <div class="main-wraper">
    <h2>Your interactive calendar</h2>
    <p>
      Lorem ipsum dolor sit amet consectetur adipisicing elit. Culpa sunt
      asperiores rerum vel voluptas tempora fugiat cum a, quia, mollitia sint
      libero veniam, rem blanditiis. Magnam repellendus sapiente aut numquam!
    </p>
    <MonthCalendar></MonthCalendar>
  </div>
</template>

<style>
.main-wraper {
  height: 100%;
  padding: 40px;
}
h2 {
  font-family: var(--font-alt);
  text-align: center;
  color: var(--success);
  font-weight: 700;
  font-size: 30px;
}
p {
  font-family: var(--font);
  color: var(--light-text);
  font-size: var(--font-size);
  text-align: center;
  margin: 20px 0;
}
</style>
