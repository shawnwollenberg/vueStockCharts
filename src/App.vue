<template>
  <v-app>
    <v-app-bar
      app
      color="#fff"
      
    >
    <v-img
        alt="Vuetify Logo"
        class="shrink mr-2"
        contain
        src="@/assets/logo.png"
        transition="scale-transition"
      />
    <v-spacer></v-spacer>
    <span class="font-weight-bold hidden-sm-and-down pl-4" style="font-size:18px;">Stock/Crypto Growth</span>
    <v-btn icon @click="overlay=!overlay" title="Help">
      <v-icon small>mdi-help</v-icon>
    </v-btn>
    </v-app-bar>

    <v-main style="background-color:#f2f2f2">
      <router-view/>
    </v-main>
    <v-overlay opacity=".9" :value="overlay">
      <h1>What is This Application?</h1>
      This is an example of a webscraping API used to populate the data for Stock / Crypto price and growth comparisons.<br><strong>Technologies Used:</strong><br>
      <ul>
        <li><strong>Golang:</strong> Most APIs for getting stock prices are expensive.  I built this simple scraper to get stock prices from a well known financial reporting site so that I can get the data for free.  <br>The API was also built with Golang using Gorilla Mux.</li>
        <li><strong>VueJS / Vuetify:</strong> VueJS is currently my favorite JavaScript framework and Vuetify makes it pretty with Material Design.</li>
        <li><strong>PlotlyJS:</strong> I love working with PlotlyJS.  I've developed apps using many other charting technologies, but PlotlyJS is clean, customizable and provides everything I need</li> 
      </ul><br>
      <h1>How does it work?</h1>
      Enter one (or more) stock symbol(s) and press the "+" to see the past 100 closing prices.<br>
      In the "Chart View" drop down you can select "Growth", "Price" or "Anchor".
      <ul>
        <li><strong>Growth</strong> - This allows you to see the growth trends side by side</li>
        <li><strong>Price</strong> - You can view the actual dollar comparison for each stock</li>
        <li><strong>Anchor</strong> - If you click on the Anchor icon next to a selected stock symbol you will set that symbol as the "Anchor".  The chart will show you growth trends vs that "Anchor" symbol.</li>
      </ul>
      <br>** Note:  When entering crypto symbols I recommend adding "-USD" to the end (ex: BTC-USD or ETH-USD).  The program is designed to look for "-USD" if it can't find a symbol, but in the example of ETH there is an existing stock symbol.
<br><br>
      <v-btn @click="overlay=false" dark color=primary title="Close Help">Close Help</v-btn>
    </v-overlay>
  </v-app>
  
</template>

<script>
export default {
  name: 'App',
  data: () => ({
    overlay:false,
  }),
};
</script>
