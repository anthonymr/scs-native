<template>
  <Page class="dark">
    <GridLayout rows="auto, auto, *" columns="auto, *">
      <DropDown
        :items="products"
        row="0"
        colSpan="2"
        hint="Producto"
        class="borded"
      />
    </GridLayout>
  </Page>
</template>

<script>
export default {
  data() {
    return {
      products: ['test', 'test2'],
    };
  },
};
</script>

<style>
.dark {
  background-color: rgb(18, 18, 18);
}

.borded {
  border-width: 1px;
  border-color: gray;
  border-style: solid;
  border-radius: 3px;
  line-height: 200px;
  font-size: 20px;
  margin: 20px;
  padding: 10px;
  color: white;
}
</style>
