<template>
    <div v-show="loggedInUserEmail != ''">
        <h3 v-show="!assigneeFilter">{{ loggedInUserName }}</h3>
        <div style="display: flex;justify-content: space-around;align-items: flex-end;">
            <button @click="addTask">Add Task</button>
            <div>
                <h5 style="margin-top: 2.178;
                        margin-top: 0px;
                        margin-bottom: 0px;
                    ">Filter list by assignee:</h5>
                <select v-model="assigneeFilter">
                    <option v-for="(user, key) in users" 
                            :value="user.id"
                            :key="key">
                        {{user.first_name + " " + user.last_name}}
                    </option>
                    <option :value=null
                            :key="key">
                            No filter
                    </option>
                </select>
            </div>
            <div v-show="!assigneeFilter">
                <h5 style="margin-top: 2.178;
                        margin-top: 0px;
                        margin-bottom: 0px;
                    ">Filter list by priority:</h5>
                <select v-model="priorityFilter">
                    <option value="low"
                            :key="key">
                            low
                    </option>
                    <option value="high"
                            :key="key">
                            high
                    </option>
                    <option value="medium"
                            :key="key">
                            medium
                    </option>
                    <option value=""
                            :key="key">
                            No filter
                    </option>
                </select>
            </div>   
        </div>
        <table >
            <tr>
                <td>Tasks</td>
                <td>Description</td>
                <td>Creation date</td>
            </tr>
            <tr v-for="(task, index) in loggedInUserTasks">
                <td>{{ task.title }}</td>
                <td>{{ task.description }}</td>
                <td>{{ task.created_at }}</td>
                <td><button @click="deleteTask(task.id)">Delete</button></td>
                <td><button @click="editTaskDialog({id: task.id, 
                                                        title: task.title, 
                                                        description: task.description,
                                                        priority: task.priority,
                                                        assignee: task.assignee,
                                                        created_at: task.created_at})">Edit</button></td>
            </tr>
        </table>

        
        <dialog>
            <h4>Task:</h4>
            <input type="text" v-model="title">
            <h4>Description:</h4>
            <input type="text" v-model="description">
            <h4>Priority</h4>
            <input type="text" v-model="priority">
            <h4>Assignee:</h4>
            <select v-model="assignee">
                <option v-for="(user, key) in users" 
                        :value="user.id"
                        :key="key">
                    {{user.first_name + " " + user.last_name}}
                </option>
            </select>
            <h4>Creation date:</h4>
            <input type="date" v-model="date">
            <button @click="addNewTask">Add</button>
            <button @click="closeTaskDialog">Close</button>
        </dialog>
        <dialog id="editDialog">
            <h4>Task:</h4>
            <input type="text" v-model="titleEdited">
            <h4>Description:</h4>
            <input type="text" v-model="descriptionEdited">
            <button @click="editTask">Update</button>
            <button @click="closeEditTaskDialog">Close</button>
        </dialog>
    </div>
</template>

<script>

export default{
    name: 'TaskList',
    props:["loggedInUserEmail", "newTask"],
    data(){
        return{
            users: [],
            tasks: [],
            loggedUserId: null,
            loggedInUserName: '',
            title: '',
            description: '',
            priority: '',
            assignee: null,
            date: '',
            addingTask: false,
            assigneeFilter: null,
            taskIDForUpdate: null,
            titleEdited: '',
            descriptionEdited: '',
            priorityOfTaskEdited: '',
            assigneeOfTaskEdited: null,
            creationDateOfTaskEdited: '',
            editDialog: false,
            priorityFilter: ''
        }
    },

    watch:{
        loggedInUserEmail:{
            handler(val){
                let user = this.users.find(user=>
                    user.email == val
                )

                this.loggedInUserName = user.first_name + " " + user.last_name
            }
        },
    },

    computed:{
        loggedInUserTasks(){

            this.users = [...this.$store.getters.users]
            this.tasks = [...this.$store.getters.tasks]

            let userTasks = []
            if(this.assigneeFilter){
                let loggedInUser = this.users.find((user)=>{
                    if(user.id == this.assigneeFilter){
                        return user
                    }
                });

                userTasks = this.tasks.filter((task)=>{
                    if(loggedInUser && (loggedInUser.id == task.assignee)){
                        return task
                    }
                })
            }

            else{
                let loggedInUser = this.users.find((user)=>{
                    if(user.email == this.loggedInUserEmail){
                        return user
                    }
                });

                if(this.priorityFilter != ''){
                    userTasks = this.tasks.filter((task)=>{
                            if(loggedInUser && (loggedInUser.id == task.assignee) && task.priority.includes(this.priorityFilter)){
                                return task
                            }
                        })
                }

                else{
                        userTasks = this.tasks.filter((task)=>{
                        if(loggedInUser && (loggedInUser.id == task.assignee)){
                            return task
                        }
                    })
                }
            }

            return userTasks
            
        }
    },

    methods: {
        addNewTask(){
            let task = {
                id: ""+ this.tasks.length + 2,
                title: this.title,
                description: this.description,
                priority: this.priority,
                assignee: this.assignee,
                created_at: this.date
            }

            this.newTask(task);
        
            this.$store.commit("addTask", task)

            const dialog = document.querySelector("dialog");
            dialog.close()
        },

        async deleteTask(id){
                const response = await fetch(`http://localhost:3000/tasks/${id}`, { 
                method: 'DELETE', 
                headers: { 
                    'Content-type': 'application/json'
                } 
            });

            console.log(await response.json())

            this.$store.commit("deleteTask", id)
        },

        editTaskDialog(data){
            this.taskIDForUpdate = data.id;
            this.titleEdited = data.title;
            this.descriptionEdited = data.description;
            this.priorityOfTaskEdited = data.priority;
            this.assigneeOfTaskEdited = data.assignee;
            this.creationDateOfTaskEdited = data.created_at
            const dialog = document.getElementById('editDialog')
            dialog.showModal()
        },

        closeEditTaskDialog(){
            const dialog = document.getElementById('editDialog')
            dialog.close()
        },

        async editTask(){

            let updatedData = {
                id: this.taskIDForUpdate,
                title: this.titleEdited,
                description: this.descriptionEdited,
                priority: this.priorityOfTaskEdited,
                assignee: this.assigneeOfTaskEdited,
                created_at: this.creationDateOfTaskEdited
            }

            const requestOptions = {
            method: "PUT",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify(updatedData)
            };

            // Make the request
            fetch(`http://localhost:3000/tasks/${this.taskIDForUpdate}`, requestOptions)

            this.$store.commit("editTask", {
                id: this.taskIDForUpdate,
                title: this.titleEdited,
                description: this.descriptionEdited
            })

            const dialog = document.getElementById('editDialog')
            dialog.close()
        },

        addTask(){
            const dialog = document.querySelector("dialog");
            dialog.showModal()
        },

        closeTaskDialog(){
            const dialog = document.querySelector("dialog");
            dialog.close()
        }
    },

    beforeCreate(){
        fetch('http://localhost:3000/users')
        .then(response => {
        if (!response.ok) {
            throw new Error('Network response was not ok');
        }
        return response.json();
        })
        .then(response => {
        this.$store.commit("initUsers", response)
        })
        .catch(error => {
        console.error('There was a problem with the fetch operation:', error);
        });



        fetch('http://localhost:3000/tasks')
        .then(response => {
        if (!response.ok) {
            throw new Error('Network response was not ok');
        }
        return response.json();
        })
        .then(response => {
        this.$store.commit("initTasks", response);
        })
        .catch(error => {
        console.error('There was a problem with the fetch operation:', error);
        });
    }
}

</script>