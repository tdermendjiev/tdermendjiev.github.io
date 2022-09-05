# Loading views conditionally in your Appwraps mobile application 

There are a few reasons why brand owners may want a different design for the mobile app than their website. One reason is that the mobile app may be used differently than the website. For example, the app may be used more for quick tasks, such as checking the weather or looking up a phone number, while the website may be used for more complex tasks, such as researching a product or reading an article. Another reason is that the mobile app may be accessed by a different audience than the website. For example, the app may be used primarily by people who are already familiar with the brand, while the website may be used by people who are new to the brand. Finally, the mobile app may be subject to different constraints than the website. For example, the app may need to be designed for a smaller screen size or slower internet connection.

[Appwraps](https://appwraps.com) allows developers to show UI components conditionally in mobile apps. This can be extremely useful for showing different versions of the app to different users, or for hiding certain features from users who don't need them. 

You can either check `window.isOpenInAppwraps` variable or call the global `isOpenInAppwraps()` function and render component depending on its value. 

## Conditional React routes

Let's say you want do display a different homepage for your mobile app users. With Appwraps and React you can do something similar to:

```
import React from 'react';
import { BrowserRouter as Router, Routes, Route, Link } from 'react-router-dom';
import Navbar from './components/Navbar';
import Home from './components/Home';
import HomeMobile from './components/HomeMobile';
import Login from './components/Login';
import Register from './components/Register';
import './App.css';
import Contact from './components/Contact';
import Cart from './components/Cart';
import Faq from './components/Faq';
import Cart from './components/Cart';
import Product from './components/Product';

export default function App() {
  return (
    <>
      <Router>
        <Navbar />

        <Routes>
          //here we use the window.isOpenFromAppwraps variable
          <Route
            path="/"
            exact
            element={window.isOpenFromAppwraps ? <HomeMobile /> : <Home />}
          />
          <Route path="/contact" element={<Contact />} />
          <Route path="/faq" element={<Faq />} />
          <Route path="/cart" element={<Cart />} />
          <Route path="/login" element={<Login />} />
          <Route path="/register" element={<Register />} />
          <Route path="/product/:id" element={<Product />} />
        </Routes>
      </Router>

      <p> by @ismael </p>
    </>
  );
}

```

## Conditional Text

You may also need to set a different title in the navbar, or any text on the page. The `window.isOpenFromAppwraps` is used again here:

```
let title = 'Web Boutique';
  if (window.isOpenFromAppwraps) {
    title = 'Mobile Boutique';
  }
```

![](https://raw.githubusercontent.com/tdermendjiev/tdermendjiev.github.io/master/assets/img/Screenshot%202022-09-05%20at%2022.18.54.png)

![](https://raw.githubusercontent.com/tdermendjiev/tdermendjiev.github.io/master/assets/img/Screenshot%202022-09-05%20at%2022.23.00.png)

## Angular routes

In Typescript we recommend using `isOpenFromAppwraps` global function. To prevent the static checker from complaining use `declare function isOpenFromAppwraps(): boolean;`. Then  it's pretty straightforward again:

```
declare function isOpenFromAppwraps(): boolean;

let productsList =
  typeof isOpenFromAppwraps == 'undefined'
    ? ProductListComponent
    : MobileProductListComponent;

@NgModule({
  imports: [
    BrowserModule,
    HttpClientModule,
    ReactiveFormsModule,
    RouterModule.forRoot([
      { path: '', component: productsList },
      { path: 'products/:productId', component: ProductDetailsComponent },
      { path: 'cart', component: CartComponent },
      { path: 'shipping', component: ShippingComponent },
    ]),
  ],
  declarations: [
    AppComponent,
    TopBarComponent,
    ProductListComponent,
    ProductAlertsComponent,
    ProductDetailsComponent,
    CartComponent,
    ShippingComponent,
  ],
  bootstrap: [AppComponent],
  providers: [CartService],
})
export class AppModule {}
```

You see that with these two variables that Appwraps engine declares adds to your javascript context you can do any type of customization and hide or show UI depending on whether it's open from an Appwraps wrapper.


