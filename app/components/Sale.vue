<template>
  <Page class="dark">
    <FlexboxLayout flexDirection="column" class="container">
      <DropDown
        :items="products"
        row="0"
        colSpan="2"
        hint="Producto"
        class="borded"
      />
      <FlexboxLayout>
        <TextField
          v-model="qty"
          hint="Qty"
          class="borded"
          keyboardType="number"
          @returnPress="qtyChanged"
        />
        <TextField
          v-model="usd"
          hint="USD"
          class="borded"
          keyboardType="number"
          @returnPress="usdChanged"
        />
        <TextField
          v-model="bs"
          hint="BS"
          class="borded"
          keyboardType="number"
          @returnPress="bsChanged"
        />
        <Button text="+" class="borded btn" @tap="addToCart" />
      </FlexboxLayout>
    </FlexboxLayout>
    <ListView for="item in cart">
      <v-template>
        <Label :text="item.name" />
        <Label :text="item.qty" />
        <Label :text="item.usd" />
        <Label :text="item.bs" />
      </v-template>
    </ListView>
  </Page>
</template>

<script>
export default {
  data() {
    return {
      bsRate: 25,
      price50: 0.08,
      price100: 1.5,
      price1000: 14,

      qty: '',
      usd: '',
      bs: '',
      products: ['test', 'test2'],

      cart: [],
    };
  },

  methods: {
    addToCart() {
      this.cart.push({
        name: 'Testing',
        bs: this.bs,
        usd: this.usd,
        qty: this.qty,
      });

      this.resetFields();
    },
    qtyChanged() {
      const qty = parseFloat(this.qty);

      if (!this.qty) {
        this.resetFields();
        return;
      }

      if (qty <= 99) {
        this.usd = qty * this.pricePerGram50;
      } else if (qty <= 999) {
        this.usd = qty * this.pricePerGram100;
      } else {
        this.usd = qty * this.pricePerGram1000;
      }

      this.bs = parseFloat(this.usd) * this.bsRate;
    },
    usdChanged() {
      const usd = parseFloat(this.usd);

      if (!this.usd) {
        this.resetFields();
        return;
      }

      if (usd < this.price100) {
        this.qty = usd / this.pricePerGram50;
      } else if (usd < this.price1000) {
        this.qty = usd / this.pricePerGram100;
      } else {
        this.qty = usd / this.pricePerGram1000;
      }

      this.bs = usd * this.bsRate;
    },
    bsChanged() {
      const bs = parseFloat(this.bs);

      if (!this.bs) {
        this.resetFields();
        return;
      }

      this.usd = bs / this.bsRate;

      this.usdChanged();
    },

    resetFields() {
      this.bs = '';
      this.usd = '';
      this.qty = '';
    },
  },

  computed: {
    pricePerGram50() {
      return this.price50 / 50;
    },
    pricePerGram100() {
      return this.price100 / 100;
    },
    pricePerGram1000() {
      return this.price1000 / 1000;
    },
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
  padding: 10px;
  margin: 5px;
  color: white;
  placeholder-color: white;
  width: 28%;
}

.btn {
  width: 16%;
  background-color: white;
  color: black;
}
</style>
