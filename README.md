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
    <span class="stats">{{ cart.items.length }} items in cart, totalling {{ cartTotal | currency }}</span>
    <button class="btn btn-primary" @click="isShowingCart = true">View Cart</button>
</div>

Let's make it so so it either displays 'items' or 'item' depending on how many items are in the cart:

<div class="text-right pull-right cart-info">
    <span class="stats">{{ cart.items.length }} 
        <template v-if="cart.items.length == 1">item</template>
        <template v-else>items</template>
        in cart, totalling {{ cartTotal | currency }}
    </span>
    <button class="btn btn-primary" @click="isShowingCart = true">View Cart</button>
</div>

SWITCHING BETWEEN VIEWS

This is a simple app that only has a product page and the cart.

**The following is not best practice**

Add a data property that contains a boolean to decide which page we are on.
Set it to false as we don't want the cart displayed on startup:

data: {
    isShowingCart: false,

Use this data property to decide which elements to display.

Show the products if data property is not true then add this to the DOM:

<div v-if="!isShowingCart" id="products" class="row list-group">

Otherwise display the shopping cart:

<div v-else>
    <h1>Cart</h1>
    <p>The cart will be displayed here</p>
</div>

Now we need to be able to change the data property.
Use the button at the top of the page for 'view cart'.
Create a @click event that sets the isShowingCart to true when clicked.
For going back to the products page, do the same thing with the 'E-commerce' button.
Then use the prevent modifier to prevent the browser from navigating up to the '#':

<nav id="top-navigation" class="well well-sm flex flex-row align-center">
    <a href="#" @click.prevent="isShowingCart = false"><strong>E-commerce Inc.</strong></a>

    <div class="text-right pull-right cart-info">
        <span class="stats">{{ cart.items.length }} <template v-if="cart.items.length == 1">item</template><template v-else>items</template> in cart, totalling {{ cartTotal | currency }}</span>
        <button class="btn btn-primary" @click="isShowingCart = true">View Cart</button>
    </div>
</nav>

DISPLAYING THE CART

The cart items are displayed in a table.
Use an if statement to display the table if there are any items in the cart:

<table v-if="cart.items.length > 0" class="table table-striped">

Or a note if there is nothing in it:

<p v-else>Your cart is currently empty.</p>

Then in the table loop through the product name, quantity and price:

<tr v-for="item in cart.items">
    <td>{{ item.product.name }}</td>

    <td>
        {{ item.quantity }}
    </td>

    <td>{{ item.quantity * item.product.price | currency }}</td>
</tr>

Then show the subtotal:

<tr>
    <td class="text-right" colspan="2">
        <strong>Subtotal</strong>
    </td>

    <td>{{ cartTotal | currency }}</td>
</tr>

For the taxes use a computed property because the tax amount depends on the product amount.
Create a 'taxAmount' computed property:

taxAmount: function() {
    return ((this.cartTotal * 10) / 100);
}

Then apply this to the template:

<tr>
    <td class="text-right" colspan="2">
        <strong>Taxes</strong>
    </td>

    <td>{{ taxAmount | currency }}</td>
</tr>

Then add the grand total:

<tr>
    <td class="text-right" colspan="2">
        <strong>Grand total</strong>
    </td>

    <td>{{ cartTotal + taxAmount | currency }}</td>
</tr>

HANDLING ADDING PRODUCTS ALREADY IN CART

At the moment we can't change what is already added into the cart.

First check to see if that product is already in the cart when it is added.
Create a getCartItem method.
Pass it product.
Loop through the items in the cart.
Check if the product.id in the loop matched the current product.id.
Return this cart item.
Otherwise return null:

getCartItem: function(product) {
    for (var i = 0; i < this.cart.items.length; i++) {
        if (this.cart.items[i].product.id === product.id) {
            return this.cart.items[i];
        }
    }

    return null;
}

Then we can make use of this in the addProductToCart method.
Create a variable for it called cartItem.
Check if it does not equal null.
If it does add quantity.
If it doesn't then add the product as before:

addProductToCart: function(product) {
    var cartItem = this.getCartItem(product);

    if (cartItem != null) {
        cartItem.quantity++;
    } else {
        this.cart.items.push({
            product: product,
            quantity: 1
        });
    }

    product.inStock--;
}

Now if the same product is added to the cart more than once, it will display that amount as it's quantity instead of listing every instance.

INCREASING AND DECREASING PRODUCT QUANTITIES

Add an increase and decrease quantity button to cart items.

Create a method to increase the quantity of a cart item.
Decrease the in stock quantity by 1:

increaseQuantity: function(cartItem) {
    cartItem.product.inStock--;
    cartItem.quantity++;
}

Create a button with the click handler for this method.
Disable this button if there is no more stock:

<td>
    {{ item.quantity }} &nbsp;
    <button class="btn btn-success" @click="increaseQuantity(item)" :disabled="item.product.inStock == 0">+</button>
</td>

Remove an item from the cart.
Find the index of the item in the cart.
Verify that the item was found.
Remove it from the array with splice by passing it the idex and the amount to remove:

removeItemFromCart: function(cartItem) {
    var index = this.cart.items.indexOf(cartItem);

    if (index !== -1) {
        this.cart.items.splice(index, 1);
    }
}

Decrease the quantity of an item in the cart.
Increase the stock amount.
If the quantity is 0 then remove it from the cart:

decreaseQuantity: function(cartItem) {
    cartItem.quantity--;
    cartItem.product.inStock++;

    if (cartItem.quantity == 0) {
        this.removeItemFromCart(cartItem);
    }
}

Add the button for this:

<td>
    {{ item.quantity }} &nbsp;
    <button class="btn btn-success" @click="increaseQuantity(item)" :disabled="item.product.inStock == 0">+</button>
    <button class="btn btn-danger" @click="decreaseQuantity(item)">-</button>
</td>