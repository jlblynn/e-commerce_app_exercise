# Vue.js E-commerce Example Application

This repository contains the code for the first example application from the [Vue.js: From Beginner to Professional course](https://l.codingexplained.com/course/vuejs?src=github).

Note that this application is **not intended for showing best practices**, as it is the very first application in a large Vue.js course.

DISPLAYING PRODUCTS:

In the index.html we want iterated over the products using the v-for directive.

Add:
v-for="product in products"
To the child div of the div with the id of 'products'.

Then replace the [PLACEHOLDERS] inside of that div with the product object and its relevant property.
Eg: [PRODUCT_NAME] to {{ product.name }}

Now to display the price as a currency.
Create a currency filter in app.js.

filters: {
  currency: function(value) {
      var formatter = new Intl.NumberFormat('en-US', {
          style: 'currency',
          currency: 'USD',
          minimumFractionDigits: 0
      });
      
      return formatter.format(value);
  }
}

Then add this currency filter to product.price like so:

{{ product.price | currency }}

Highlight with colours depending on how much stock of that product is left.
Bind the classes of few and none depending on the quantity of stock left:

<div class="number-in-stock" :class="{ few: product.inStock < 10 && product.inStock > 0, none: product.inStock == 0 }">
    {{ product.inStock }} in stock
</div>

Disable the 'add to cart' button if the product is out of stock.
Find the button and use the v-bind directive to bind to the disabled property if there is no stock:

<button class="btn btn-success" :disabled="product.inStock == 0">Add to cart</button>

ADDING PRODUCTS TO THE CART

Create a cart object under the data property.
Then an empty items array inside of that.

Create a method that is responsible for adding contents to the cart:

addProductToCart: function(product) {
    this.cart.items.push({
        product: product,
        quantity: 1
    });

    product.inStock--;
}

Now attatch this method to a click event button:

<button class="btn btn-success" @click="addProductToCart(product)" :disabled="product.inStock == 0">Add to cart</button>

DISPLAYING CART INFO IN THE MENU

We want to display how many items are in the cart and the total cost of the cart.
Use a computed pbject for this.
Create a cartTotal property within this.
Set the total to 0.
Loop through the products in the cart and find the sum total.
Then add the tax:

computed: {
    cartTotal: function() {
        var total = 0;

        this.cart.items.forEach(function(item) {
            total += item.quantity * item.product.price;
        });

        return total;
    },
    taxAmount: function() {
        return ((this.cartTotal * 10) / 100);
    }
}

Display this information:

<div class="text-right pull-right cart-info">
    <span class="stats">{{ cart.items.length }} <template v-if="cart.items.length == 1">item</template><template v-else>items</template> in cart, totalling {{ cartTotal | currency }}</span>
    <button class="btn btn-primary" @click="isShowingCart = true">View Cart</button>
</div>