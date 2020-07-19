<template>
  <div>
    <nav class="navbar navbar-light bg-light">
      <a class="navbar-brand">
        <img src="@/icons/exam.svg"
        width="30" height="30" class="d-inline-block align-top" alt="">
        Test generator
      </a>
    </nav>
    <h1 style='title border'>{{title}}</h1>
    <div v-if="!evaluated">
      <div v-if='!form_submitted'>
        <h3> {{error}} </h3>
      <form>
        <div class='form-group col-md-4 center mx-auto'>
          <label for='inputState'>Amount of answers</label>
          <select id='inputState' class='form-control' v-model='number_questions'>
            <option v-for='(i, index) in 41' :key='index'>{{i + 9}}</option>
          </select>
        </div>
        <div class='form-group col-md-4 center mx-auto'>
          <label for='inputState'>Choose a category</label>
          <select name='trivia_category' class='form-control' v-model="category">
            <option value='any'>Any Category</option>
            <option value='9'>General Knowledge</option>
            <option value='10'>Entertainment: Books</option>
            <option value='11'>Entertainment: Film</option>
            <option value='12'>Entertainment: Music</option>
            <option value='13'>Entertainment: Musicals &amp; Theatres</option>
            <option value='14'>Entertainment: Television</option>
            <option value='15'>Entertainment: Video Games</option>
            <option value='16'>Entertainment: Board Games</option>
            <option value='17'>Science &amp; Nature</option>
            <option value='18'>Science: Computers</option>
            <option value='19'>Science: Mathematics</option>
            <option value='20'>Mythology</option>
            <option value='21'>Sports</option>
            <option value='22'>Geography</option>
            <option value='23'>History</option>
            <option value='24'>Politics</option>
            <option value='25'>Art</option>
            <option value='26'>Celebrities</option>
            <option value='27'>Animals</option>
            <option value='28'>Vehicles</option>
            <option value='29'>Entertainment: Comics</option>
            <option value='30'>Science: Gadgets</option>
            <option value='31'>Entertainment: Japanese Anime &amp; Manga</option>
            <option value='32'>Entertainment: Cartoon &amp; Animations</option>
          </select>
        </div>
        <div class='form-group col-md-4 center mx-auto'>
          <label for='inputState'>Set the difficulty</label>
          <select name='trivia_difficulty' class='form-control' v-model='difficulty'>
            <option value='any'>Any difficulty</option>
            <option value='easy'>Easy</option>
            <option value='medium'>Medium</option>
            <option value='hard'>Hard</option>
          </select>
        </div>
        <div class='form-group col-md-4 center mx-auto'>
          <label for='inputState'>Select the type of answers</label>
          <select name='trivia_type' class='form-control' v-model='type'>
            &gt;
            <option value='any'>Any type</option>
            <option value='multiple'>Multiple selection</option>
            <option value='boolean'>True/False</option>
          </select>
        </div>
        <div class='form-group col-md-1 center mx-auto'>
          <input type='button' class='btn btn-info' value='Send' @click='getAPIRes' />
        </div>
      </form>
      <img src='https://i.imgur.com/KvZU0oE.gif' v-if='loading' />
      <footer style="
    left: 0;
    bottom: 0;
    width: 100%;
    text-align: center;">
      <p> <pre> Author: Sergio Garrido Domínguez Source code on Github (link <a href="https://github.com/iSergio97/vuejs"> here</a>) Mail to: <a href="mailto:sergardom@alum.us.es"> sergardom@alum.us.es</a> </pre></p>&nbsp;&nbsp;
    </footer>
    </div>
    <div v-else>
      <div v-for='(res, i) in results' :key='i' class='border border-info'>
        <div class='question'>
          <h4>{{res.question}}</h4>
          <div v-if="res.difficulty === 'hard'">
            <p class='border border-danger rounded' style='background-color: red;'>Hard</p>
          </div>
          <div v-else-if="res.difficulty === 'medium'">
            <p class='border border-warning rounded' style='background-color: yellow;'>Medium</p>
          </div>
          <div v-else>
            <p class='border border-success rounded' style='background-color: #90EE90;'>Easy</p>
          </div>
        </div>
        <div class='answers' v-for='(a) in res.answers' :key='a'>
          <input
            type='radio'
            :value='a'
            v-model='res.selected_answer'
            :name='res.actual_answer'
          />
          {{a}}
        </div>
        <br />
      </div>
      <div class='form-group col-md-3 center mx-auto'>
        <button @click="getResults" class='btn btn-info'>
          Get results! (Cross your fingers 🤞)
        </button>
      </div>
    </div>
    </div>
    <div v-else>
      <div v-if="score < 5">
        <img class="img-final sad" src="https://i.imgur.com/hynzOMV.png" >
      </div>
      <div v-else>
        <img class="img-final happy" src="https://i.imgur.com/lNpc8qG.png" >
      </div>
      <br>
      <br>
      <br>
      <h4> {{final_solution}}</h4>
      <h4> You can try again clicking <a href="/">here</a>! </h4>
      <h5> Final score: {{score}} </h5>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      form_submitted: false,
      number_questions: 10,
      category: 'any',
      difficulty: 'any',
      type: 'any',
      results: [],
      loading: false,
      evaluated: false,
      score: 0,
      title: 'Check your knowlegde here with a tons of themes to try!',
      final_solution: '',
      error: '',
    };
  },
  methods: {
    async getAPIRes() {
      /*eslint-disable*/
      this.loading = true;
      const BASE_URL = 'https://opentdb.com/api.php?';
      var concat = BASE_URL.concat('amount=' + this.number_questions);
      if (this.category !== 'any') {
        concat = concat.concat(`&category=${this.category}`);
      }
      if (this.difficulty !== 'any') {
        concat = concat.concat(`&difficulty=${this.difficulty}`);
      }
      if (this.type !== 'any') {
        concat = concat.concat(`&type=${this.type}`);
      }

      this.error = "";
      await fetch(concat)
        .then(res => res.json())
        .then(res => {
          this.results = res.results;
        });
      this.results.forEach((res, i) => {
        const array = [];
        res.incorrect_answers.forEach(str => {
          array.push(str);
        });
        array.push(res.correct_answer);
        array.sort(() => Math.random() - 0.5);
        res.answers = array;
        res.selected_answer = '';
        res.actual_answer = i;
        const arrayAnswers = [];
        res.answers.forEach(str => {
          let replaced = "";
          if(str.includes("&quot;") || str.includes("&#039;")) {
            if(str.includes("&quot;")) {
              let replaced = str.replace(/&quot;/g, "\"");
              arrayAnswers.push(replaced);
            } else {
              let replaced = str.replace(/&#039;/g, "\'");
              arrayAnswers.push(replaced);
            }
          } else {
            arrayAnswers.push(str);
          } 
        });
        res.answers = arrayAnswers;
        if(res.question.includes("&quot;")) {
          res.question = res.question.replace(/&quot;/g, "\"");
        } 
        if(res.question.includes("&#039;")) {
          res.question = res.question.replace(/&#039;/g, "\'");
        }
      });

      if(this.results.length === 0) {
        this.loading = false;
        this.error = "There are no questions for this combination, please try again with another one";
      } else {
        this.form_submitted = true;
        this.loading = false;
      }

      this.title = 'Test your knowlegde!';
    },
    getResults() {
      this.title = 'Here are your results!';
      this.evaluated = true;
      this.results.forEach(res => {
        if(res.selected_answer === res.correct_answer) {
          this.score += 1;
        };
      });
      this.score = (Math.round((this.score/this.number_questions) * 100)/100) * 10;
      if(this.score >= 5) {
        this.final_solution = 'Congratulations! You have passed this test!';
      } else {
        this.final_solution = 'Dont worry about this test!'
      }
    }
  }
};
</script>

<style>
.question, .answers {
  border: 'thick solid #0000FF';
  text-align: center;
}

h1, h3, h4, h5{
  text-align: center;
}

img {
  width: 10%;
  display: block;
  margin-left: auto;
  margin-right: auto;
}

p {
  display: inline-block;
}

.form-group col-md-1 submit {
  text-align: center;
}

btn {
  margin-left:50%;
  margin-right:50%;
  position: relative;
  text-align: center;
}

.img-final {
  width: 15%;
}

h3 {
color: red;
}
</style>
