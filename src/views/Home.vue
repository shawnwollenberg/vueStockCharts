<template>
  <div class="home" >
    <v-row dense>
      <v-layout xs5 justify-center>
          <v-flex>
            <v-select
              title="Chart View"
              hint="Chart View"
              persistent-hint
              v-model="selChartView"
              :items="chartView"
              style="width:180px;"
              class="pl-2 pt-2"
              dense
              @change="buildPlotly()"
            ></v-select>
            <v-text-field
              v-model="stockSymbolAdd"
              label="Enter Symbol"
              hint="Stock Symbol"
              persistent-hint
              append-outer-icon="mdi-plus"
              single-line
              dense
              style="width:200px;"
              class="pl-2 pt-2"
              @click:append-outer="addStock()"
            >
            </v-text-field>
            <v-snackbar
              v-model="snackbarShow"
              top
              left
              color="error"
            >
              {{snackbarTxt}}
              <template v-slot:action="{ attrs }">
                <v-btn
                  dark
                  text
                  v-bind="attrs"
                  @click="snackbarShow = false"
                >
                  Close
                </v-btn>
              </template>

            </v-snackbar>            
            <ul>
              <li v-for="(x,indx) in stockData" :key="indx">
                {{ x.Symbol}} 
                <v-btn icon color="red" @click="removeData(x.indx)" title="Remove">
                  <v-icon small>mdi-minus</v-icon>
                </v-btn>
                <v-btn v-if="showAnchor" icon :color="symAnchor==x.Symbol?'blue':'gray'" @click="symAnchor==x.Symbol?symAnchor='':symAnchor=x.Symbol; buildPlotly();">
                  <v-icon small>mdi-anchor</v-icon>
                </v-btn>              
              </li>
            </ul>
          </v-flex>
      </v-layout>
      <v-layout xs7 justify-center class="pt-2">
        <div id="plotlyDiv" v-bind:style="cstyleAll"></div>
      </v-layout>
    </v-row>

  </div>
</template>

<script>
import _ from 'lodash'
import axios from 'axios';
export default {
    data () {
      return {
        overlay:false,
        selChartView:'Growth',
        chartView:['Growth','Price'],
        stockSymbolAdd:'',
        stockData:[],
        showAnchor:false,
        symAnchor:'',
        snackbarTxt:'',
        snackbarShow:false,
        cstyleAll:'background-color:white;position: relative; height:90vh; width:85vw;display:block;',
      }
    },  
    methods: {
      updateViews: function(){
        if(this.stockData.length>1){
          this.chartView=['Growth','Price','Anchor'];
          this.showAnchor=true;
        }else{
          this.showAnchor=false;
          if(this.selChartView=='Anchor'){
            this.selChartView='Growth'
          }
          this.chartView=['Growth','Price'];
        }
      },
      addStock: function(){
        var self=this;
        axios.post('http://thotspill.com/stock/'+this.stockSymbolAdd)
        .then(function (response) {
          if(Object.keys(response.data)[0]!="ERROR"){
            self.stockData.push(response.data);
            self.buildPlotly();
            self.updateViews();
          }else{
            self.displayError("Having an issue finding " + self.stockSymbolAdd)
          }
          self.stockSymbolAdd='';
        })
        .catch(function (error) {
          console.log(error);
        });
      },
      displayError: function(errStr){
        this.snackbarTxt=errStr;
        this.snackbarShow=true;
        setTimeout(function(){ this.snackbarShow=false;}, 5000);
      },
      removeData: function(i){
        this.stockData.splice(i,1);
        this.buildPlotly()
      },
      buildPlotly: function(){
        var dispData = [];
        var chartTitle=this.selChartView + ' Trends'
        let yCompGrowth=[];
        if(this.selChartView=="Anchor" && this.symAnchor){
          chartTitle="Growth Compared to " + this.symAnchor;
          for(let i=0;i<this.stockData.length;i++){  
            if(this.stockData[i].Symbol===this.symAnchor){
              let yDataFullVals=Object.values(this.stockData[i].DtStockPrice);
              let d=Object.values(this.stockData[i].DtStockPrice)[0];
              for(let j=0;j<yDataFullVals.length;j++){
                yCompGrowth.push(yDataFullVals[j]/d)
              }
            }
          }
        }
        for(let i=0;i<this.stockData.length;i++){
          let xData=Object.keys(this.stockData[i].DtStockPrice);
          let yDataFullVals=Object.values(this.stockData[i].DtStockPrice);
          let yTxt=Object.values(this.stockData[i].DtStockPrice);
          if(this.selChartView=="Price"){
            dispData.push({
              type:"scatter",
              mode:"lines",
              name:this.stockData[i].Symbol,
              //text:this.stockData[i].CompanyName,
              x:xData,
              y:yDataFullVals,
              txt:yTxt,
              hovertemplate: "<b>" + this.stockData[i].Symbol + " - %{x}</b> <br>Price: %{y:$.2f}<extra></extra>"
            })
          }else{
            let d=Object.values(this.stockData[i].DtStockPrice)[0];
            let yDataGrowth=[];
            let hvrtemp="<b>" + this.stockData[i].Symbol + " - %{x}</b><br>Growth: %{y:.0%} <br>Price: %{text:$.2f} <extra></extra>";
            if(this.selChartView=="Anchor" && this.symAnchor){
              hvrtemp="<b>" + this.stockData[i].Symbol + " - %{x}</b><br>Growth vs " + this.symAnchor + ": %{y:.0%} <br>Price: %{text:$.2f} <extra></extra>";
            }
            for(let j=0;j<yDataFullVals.length;j++){
              if(this.selChartView=="Anchor" && this.symAnchor){
                yDataGrowth.push((yDataFullVals[j]/d)-yCompGrowth[j])
              }else{
                yDataGrowth.push(yDataFullVals[j]/d)
              }
            }
            dispData.push({
              type:"scatter",
              mode:"lines",
              name:this.stockData[i].Symbol,
              //text:this.stockData[i].CompanyName,
              x:xData,
              y:yDataGrowth,
              text:yTxt,
              hovertemplate: hvrtemp
            })
          }
        }
        
        var layout = { 
          hovermode: 'closest',
          legend: {
            orientation: 'h',
            yanchor:'top',
            xanchor:'center',
            x:0.5,
            y: 1.05,
            yref: 'paper',
            font: {
              family: 'Arial, sans-serif',
              size: 16,
              color: 'grey',
            }
          },
          title: chartTitle
        };              
        var myDIV=document.getElementById('plotlyDiv');
        Plotly.newPlot(myDIV, dispData,layout, {responsive: true, displayModeBar: true, toImageButtonOptions: {filename:'Shawn\'s Stock Comparison',width: null, height: null}});
      }
   }
}
</script>
