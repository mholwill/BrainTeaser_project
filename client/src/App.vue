<template>
  <div id="app">
    <h1>.BrainTeaser.</h1>

  <div class="menu">
    <button class="nav-button" @click="handleStartFormClick($event)">New Game</button>
    <button class="nav-button" @click="handleTopScoresClick($event)">Top Scores</button>
    <button class="nav-button" @click="handleMuteSoundsClick">{{(muted)?'Enable':'Mute'}} Sounds</button>
  </div>

    <component v-bind:is='component' :categories='categories' :questions="questions[index]" :answers="answers[index]" :questionsAsked="questions" :total="total" :topScores="topScores" :muted="muted"/>
  </div>
</template>

<script>

import StartForm from '@/components/StartForm.vue';
import {eventBus} from '@/main.js';
import { shuffle } from 'lodash';
import Questions from '@/components/Questions.vue';
import EndScore from '@/components/EndScore.vue'
import EndScoreForm from '@/components/EndScoreForm.vue'
import TopScores from '@/components/TopScores.vue'
import firebase from "./firebaseInit";
const db = firebase.firestore();



export default {
  name: 'App',
  data(){
    return {
      categories: {},
      selectedCategory: null,
      questions: [],
      answers: [],
      index: 0,
      total: 0,
      topScores: [],
      component: StartForm,
      muted: true
    }
  },
  mounted(){

    fetch('https://opentdb.com/api_category.php')
    .then(res => res.json())
    .then(categories => this.categories = categories.trivia_categories)


    eventBus.$on('category-selected', (payload) => {
      this.selectedCategory = payload;

    fetch(`https://opentdb.com/api.php?amount=10&category=${this.selectedCategory.id}&difficulty=${this.selectedCategory.difficulty}&type=multiple`)
      .then(res => res.json())
      .then((questions) => {
        this.questions = questions.results;
        this.formattedQuestions(questions.results);
      })
      if (this.component === StartForm) {
        this.component = Questions;
      } else {
        this.component = StartForm;
      }
      })
      eventBus.$on('answer-selected', (payload) => {
         this.index += 1;
         this.total += payload;
         if( this.index === 10){
           this.component = EndScore;
         } else {
           this.component = Questions;
         }
      }),
      eventBus.$on('saveyourscore', (payload) => {
        if(this.component === EndScore) {
          this.component = EndScoreForm
        }else {
          this.component = EndScore
        }
      })
      eventBus.$on('score-added', (payload) => {
        this.createTopScore(payload)
        if(this.component === EndScoreForm) {
          this.component = TopScores
          this.getTopScores()
        } else {
          this.component = StartForm
        }
      })},
  components: {
    StartForm,
    Questions,
    EndScore,
    EndScoreForm,
    TopScores
  },
  methods: {
    formattedQuestions: function (questions) {
      questions.forEach((question) => {
        let newArray = [...question.incorrect_answers, question.correct_answer]

        newArray = shuffle(newArray)

        this.answers = [...this.answers, newArray]
      })
    },
    handleStartFormClick: function(){
      this.index = 0;
      this.total = 0;
      this.answers = [];
      this.component = StartForm
    },
    handleTopScoresClick: function(){
      this.getTopScores()
      this.component = TopScores
      
    },
    handleMuteSoundsClick: function(){
      this.muted = !this.muted
    },
    getTopScores() {
      this.topScores = []
      db.collection("players")
        .get()
        .then((querySnapshot) => {
          querySnapshot.forEach((doc) => {
            this.topScores.push({
              id: doc.id,
              name: doc.data().name,
              score: doc.data().score,
              difficulty: doc.data().difficulty,
              category: doc.data().category
            });

          });
        })
        .catch((error) => {
          console.log("Error getting documents: ", error);
        });
    },
    createTopScore(payload) {
      if (payload.name != "") {
        db.collection("players")      
          .add({ name: payload.name, score: payload.score, difficulty: payload.difficulty, category: payload.category })
          .then(() => {
            console.log("Document successfully written!");
            this.getTopScores();
          })
          .catch((error) => {
            console.error("Error writing document: ", error);
          });
      }
      }
  }
}
</script>

<style>

  @import url('https://fonts.googleapis.com/css?family=Quicksand:600&display=swap');

  @import url('https://fonts.googleapis.com/css?family=Bowlby+One+SC&display=swap');

  #app {
    font-family: 'Quicksand', sans-serif;
    color: black;
    margin-top: 60px;
  }

  .menu{
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;
    text-align: center;
    align-items: center;
    margin: 0 25% 30px 25%;
  }

  .menu li {
    text-decoration: none;
    list-style: none;
    padding: 0 30px;
    margin-bottom: 20px;
  }

  h1 {
    font-family: 'Bowlby One SC', cursive;
    font-size: 5em;
    line-height: 30px;
    text-align: center;
    color: #FFBA08;
    -webkit-text-stroke: 2px black;
  }

  .nav-button {
    background-color: #FFBA08;
    font-family: 'Quicksand', sans-serif;
    font-size: 1em;
    border: 2px #1C3144 solid;
    padding: 10px;
    border-radius: 8px;
    width: auto;
    margin-left: auto;
    margin-right: auto;
    text-align: center;
    outline: none;
    cursor: pointer;
  }

  .nav-button:hover {
    box-shadow: 0 8px 16px 0 #A2AEBB, 0 6px 20px 0 #A2AEBB
  }

  .active-TS {
    border: 2px red solid;
  }
  .active-SF {
    border: 2px red solid;
  }
</style>
