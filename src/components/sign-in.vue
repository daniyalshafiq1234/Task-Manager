<template>
    <div>
        <div v-if="options">
            <button @click="signIn">Sign In</button>
            <button @click="signUp">Sign Up</button>
        </div>
        <div v-else>
            <div v-if="!signin">
                <h2>Sign Up</h2>
                <div>
                    <h3>First Name:</h3>
                    <input type="text" v-model="first_name"/>
                    <h3>Last Name:</h3>
                    <input type="text" v-model="last_name"/>
                    <h3>Email:</h3>
                    <input type="text" v-model="email"/>
                    <button @click="formSubmission">Submit</button>
                </div>
            </div>
            <div v-else>
                <h2>Log in</h2>
                <div>
                    <h3>Email:</h3>
                    <input type="text" v-model="email">
                    <button @click="logIn(email)">Log In</button>
                </div>
            </div>
        </div>
    </div>
</template>

<script>

export default {
    name: 'SignIn',
    props:["submitForm", "logIn"],
    data(){
        return{
            first_name: '',
            last_name: '',
            email: '',
            signin: false,
            options: true
        }
    },

    watch:{
        signin:{
            handler(val){
                console.log(val)
            }
        }
    },

    methods:{
        formSubmission(){
            let ID = Math.random()
            this.submitForm({id: ID, first_name: this.first_name, last_name: this.last_name, email: this.email});
            this.signup = false
            this.options = true
        },

        signUp(){
            this.options = false
            this.signin = false
        },

        signIn(){
            this.options = false
            this.signin = true
        },
    }
}

</script>