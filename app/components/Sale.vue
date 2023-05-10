<template>
  <Page class="dark">
    <FlexboxLayout flexDirection="column" class="container container3">
      <DropDown
        :items="products"
        key="name"
        row="0"
        colSpan="2"
        hint="Producto"
        class="borded"
        @selectedIndexChanged="productChange"
        ref="productList"
      />
      <FlexboxLayout>
        <TextField
          v-model="qty"
          hint="Qty"
          class="borded"
          keyboardType="number"
          @returnPress="qtyChanged"
          @blur="qtyChanged"
          :isEnabled="enableFields"
        />
        <TextField
          v-model="usd"
          hint="USD"
          class="borded"
          keyboardType="number"
          @returnPress="usdChanged"
          @blur="usdChanged"
          :isEnabled="enableFields"
        />
        <TextField
          v-model="bs"
          hint="BS"
          class="borded"
          keyboardType="number"
          @returnPress="bsChanged"
          @blur="bsChanged"
          :isEnabled="enableFields"
        />
        <Button
          text="+"
          class="borded btn"
          @tap="addToCart"
          :isEnabled="enableFields"
        />
      </FlexboxLayout>

      <stack-layout class="hr"></stack-layout>

      <FlexboxLayout v-if="!cart.length">
        <Label
          text="No hay artículos en el carrito"
          class="list-item list-placeholder"
        />
      </FlexboxLayout>

      <FlexboxLayout flexDirection="column" class="container2">
        <ListView for="(item, index) in cart">
          <v-template>
            <FlexboxLayout>
              <Label :text="item.name" class="list-item list-name" />
              <Label :text="item.qty" class="list-item list-qty" />
              <Label :text="item.usd" class="list-item list-usd" />
              <Label :text="item.bs" class="list-item list-bs" />
              <Button
                text="-"
                class="borded btn btn-small"
                @tap="removeFromCart(index)"
              />
            </FlexboxLayout>
          </v-template>
        </ListView>
      </FlexboxLayout>

      <stack-layout class="hr"></stack-layout>

      <FlexboxLayout class="container2">
        <TextField
          v-model="bsRate"
          hint="Tasa"
          class="borded list-name"
          keyboardType="number"
        />

        <Label text="Total:" class="list-item list-qty" />
        <Label :text="totals.usd" class="list-item list-usd" />
        <Label :text="totals.bs" class="list-item list-bs" />
        <Button text="-" class="borded btn btn-small" @tap="cart = []" />
      </FlexboxLayout>

      <FlexboxLayout class="container3">
        <Button text="Copiar" @tap="copy" class="borded btn btn-large" />
      </FlexboxLayout>
    </FlexboxLayout>
  </Page>
</template>

<script>
var clipboard = require('nativescript-clipboard');

export default {
  data() {
    return {
      bsRate: '',
      price50: 0,
      price100: 0,
      price1000: 0,

      qty: '',
      usd: '',
      bs: '',
      products: [],

      cart: [],
      selectedProductName: '',
      selectedProduct: null,
    };
  },

  created() {
    this.products = this.$root.products.map((item) => item.name);
  },

  methods: {
    copy() {
      let text = '';

      this.cart.forEach((item) => {
        text += `_${item.qty}${item.unit}_ ${item.name} *${item.usd}*$\n`;
      });

      const totalUSD = this.totals.usd;
      const totalBS = this.totals.bs;
      text += `\n*Total:* ${totalUSD}$ = ${totalBS}Bs`;

      clipboard.setText(text).then(function () {
        console.log('OK, copied to the clipboard');
      });
    },
    productChange() {
      this.resetFields();

      let index = this.$refs.productList.nativeView.selectedIndex;
      this.selectedProduct = this.$root.products[index];

      console.log(this.selectedProduct);

      if (this.selectedProduct.unit === 'u') {
        this.qty = 1;
        this.usd = this.selectedProduct.price1000;
        this.bs = parseFloat(this.usd) * this.rate;
        return;
      }

      if (this.selectedProduct.unit === 's') {
        this.qty = 1;
        return;
      }

      this.price50 = parseFloat(this.selectedProduct.price50);
      this.price100 = parseFloat(this.selectedProduct.price100);
      this.price1000 = parseFloat(this.selectedProduct.price1000);
    },
    removeFromCart(index) {
      this.cart.splice(index, 1);
    },
    addToCart() {
      if (
        !this.bs ||
        !this.usd ||
        !this.qty ||
        !this.selectedProduct ||
        !this.bsRate
      )
        return;

      this.cart.push({
        name: this.selectedProduct.name,
        bs: this.bs,
        usd: this.usd,
        qty: this.qty,
        unit: this.selectedProduct.unit,
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
        this.usd = (qty * this.pricePerGram50).toFixed(1);
      } else if (qty <= 999) {
        this.usd = (qty * this.pricePerGram100).toFixed(1);
      } else {
        this.usd = (qty * this.pricePerGram1000).toFixed(1);
      }

      this.bs = (parseFloat(this.usd) * this.rate).toFixed(1);
    },
    usdChanged() {
      if (this.selectedProduct.unit === 's') {
        this.bs = (parseFloat(this.usd) * this.rate).toFixed(1);
        return;
      }

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

      this.bs = (usd * this.rate).toFixed(1);
    },
    bsChanged() {
      const bs = parseFloat(this.bs);

      if (!this.bs) {
        this.resetFields();
        return;
      }

      this.usd = (bs / this.rate).toFixed(1);

      this.usdChanged();
    },

    resetFields() {
      this.bs = '';
      this.usd = '';
      this.qty = '';
    },
  },

  computed: {
    enableFields() {
      return this.bsRate && this.selectedProduct ? true : false;
    },
    rate() {
      if (!this.bsRate) return 0;

      return parseFloat(this.bsRate);
    },
    pricePerGram50() {
      return this.price50 / 50;
    },
    pricePerGram100() {
      return this.price100 / 100;
    },
    pricePerGram1000() {
      return this.price1000 / 1000;
    },

    totals() {
      if (!this.cart.length) return { usd: 0, bs: 0 };

      return this.cart.reduce(
        (acc, item) => {
          acc.usd = (acc.usd || 0) + parseFloat(item.usd);
          acc.bs = (acc.bs || 0) + parseFloat(item.bs);

          return acc;
        },
        { bs: 0, usd: 0 }
      );
    },
  },
};
</script>

<style>
.hr {
  width: 100%;
  height: 2px;
  background-color: white;
  margin: 20px 0;
}

.dark {
  background-color: rgb(18, 18, 18);
}

.container {
  height: 100%;
}

.container2 {
  color: white;
}

.container3 {
  width: 100%;
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
  color: black;
  background-color: white;
  placeholder-color: black;
  width: 28%;
}

.btn {
  width: 16%;
  background-color: white;
  color: black;
}

.btn.btn-small {
  width: 8%;
  line-height: 120px !important;
}

.btn.btn-large {
  width: 100%;
}

.list-item {
  line-height: 120px;
  font-size: 14px;
  padding: 10px;
}
.list-name {
  width: 41%;
}

.list-placeholder {
  color: gray;
}

.list-qty {
  width: 17%;
}
.list-bs {
  width: 17%;
}
.list-usd {
  width: 17%;
}
</style>
