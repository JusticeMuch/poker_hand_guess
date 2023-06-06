<script >
import { defineComponent } from 'vue';
import vSelect from 'vue-select'

const SUITES = {
  'H': 'Hearts',
  'S': 'Spades',
  'C': 'Clubs',
  'D': 'Diamonds'
};

const NUMBERS = {
  1: "Ace",
  2: "Two",
  3: "Three",
  4: "Four",
  5: "Five",
  6: "Six",
  7: "Seven",
  8: "Eight",
  9: "Nine",
  10: "Ten",
  11: "Jack",
  12: "Queen",
  13: "King"
};

export default defineComponent({
  components : {
    vSelect
  },
  props: {

  },
  data() {
    return {
      cards: [],
      showSubmit : false,
      result : null
    }
  },
  watch : {
    cards(){
      if (this.cards.length == 5){
        this.showSubmit = true;
        this.result = null;
      }else{
         this.showSubmit = false;
      }
    },
    result(){
      
    }
  },
  computed: {
    options() {
      let options = [];
      Object.entries(NUMBERS).forEach(([nkey, nval]) => {
        Object.entries(SUITES).forEach(([skey, sval]) => {
          options.push(
            {
              name: `${nval} ${sval}`,
              number : nkey,
              suite : skey
            }
          )
        });
      });

      return options;
    }
  },
  methods : {
    guessPokerHand(){
      let countObj = {};

      //conditions to check for
      let pair1 = null;
      let pair2 = null;
      let flush = null;
      let straight = null;

      //cycle through cards to get count of number and suite
      this.cards.forEach(row => {
        if (!(row.suite in countObj)){
          countObj[row.suite] = 0;
        }

        if (!(row.number in countObj)){
          countObj[row.number] = 0;
        }

        countObj[row.number]++;
        countObj[row.suite]++;

        //check for 1st and 2nd pair
        if (countObj[row.number] > 1){

          if (!pair1 || pair1.number == row.number){
            pair1 = {
              count : countObj[row.number],
              number : row.number
            }
          }

          if (pair1 && pair1.number != row.number){
            pair2 = {
              count : countObj[row.number],
              number : row.number
            }
          }
        }

        //check for flush
        if (countObj[row.suite] == 5){
          flush = true;
        }
      })

      //guess pairs;
      if (pair1 && !pair2){
        switch (pair1.count){
          case 2:
            this.result = "One Pair";
            return

          case 3:
            this.result = "Three of a Kind";
            return

          case 4:
            this.result = "Four of a Kind";
            return;
        }
      }

      //if two pairs
      if (pair1 && pair2){
        if (pair1.count + pair2.count == 5){
          this.result = "Full House";
        }else{
          this.result = "Two Pair";
        }
        return;
      }

      //done checking for all pairs
      //check for straight
      let objKeys = Object.keys(countObj);
      let numberKeys = objKeys.filter(k => !isNaN(k)).map(n => parseInt(n)).sort((a,b) => a - b);
      
      //change ace value in straight if second value in asc order is 10
      if (numberKeys[0] == 1 && numberKeys[1] == 10){
        numberKeys[0] = 14;
        numberKeys = numberKeys.sort((a,b) => a -b)
      }

      //check for straight condition 
      straight = numberKeys.length == 5 && numberKeys[4] - numberKeys[0] == 4;

      if (flush){
        this.result = straight ? "Straight Flush" : "Flush";
        return;
      }

      if (straight){
        this.result =  "Straight";
        return;
      }

      this.result = "High Card";
    },
  },
  mounted() {
    
  }
})
</script>

<template>
  <div class="container">
    <h3> Please select five cards </h3>
    <v-select label="name" :options="options" v-model="cards" multiple class="card-select"></v-select>
  </div>
  <button @click="guessPokerHand" v-if="showSubmit">Guess Poker Hand</button>
  <h4 v-if="result && showSubmit">{{ result }}</h4>
</template>

<style scoped>
h1 {
  font-weight: 500;
  font-size: 2.6rem;
  top: -10px;
}

h3 {
  font-size: 1.2rem;
}
.card-select{
    width: 600px;
  }
.container h1,

@media (min-width: 1024px) {

  .container h1,
  .container h3 {
    text-align: left;
  }

  
}
</style>
