<template>
  <main>
    <div>
      <h1>Task Manager</h1>
      <div >
        <sign-in v-show="!signedIn"
          :submitForm="submitForm"
          :logIn="logIn"/>

        <task-list v-show="signedIn"
          :loggedInUserEmail="loggedInUserEmail"
          :newTask="newTask"/>
      </div>
    </div>
  </main>
</template>

<script>
import TaskList from './components/task-list.vue'
import SignIn from './components/sign-in.vue'

export default{
  name: 'task-manger',
  components:{
    'task-list': TaskList,
    'sign-in': SignIn
  },

  data(){
    return{
      signedIn: false,
      loggedInUserEmail : ''
    }
  },

  methods:{
    async newTask(value){
      
      const response = await fetch("http://localhost:3000/tasks", { 
            method: 'POST', 
            headers: { 
                'Content-type': 'application/json'
            }, 
            body: JSON.stringify(value) 
        });
    },

    async submitForm(value){

      const response = await fetch("http://localhost:3000/users", { 
            method: 'POST', 
            headers: { 
                'Content-type': 'application/json'
            }, 
            body: JSON.stringify(value) 
        });

    },


    logIn(value){
      this.loggedInUserEmail=value
      if(this.loggedInUserEmail != ''){
        this.signedIn = true;
      }
    }
  }
}


</script>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
