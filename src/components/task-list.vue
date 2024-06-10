<template>
    <div class="task-list-container" v-show="loggedInUserEmail != ''">
        <h3 v-show="!assigneeFilter" class="user-name">{{ loggedInUserName }}</h3>
        <div class="filter-controls">
            <button @click="addTask" class="btn">Add Task</button>
            <div class="filter-group">
                <h5>Filter list by assignee:</h5>
                <select v-model="assigneeFilter" class="select-field">
                    <option v-for="(user, key) in users" 
                            :value="user.id"
                            :key="key">
                        {{user.first_name + " " + user.last_name}}
                    </option>
                    <option :value="null"
                            :key="key">
                            No filter
                    </option>
                </select>
            </div>
            <div class="filter-group">
                <h5>Filter list by priority:</h5>
                <select v-model="priorityFilter" class="select-field">
                    <option value="low"
                            :key="key">
                            Low
                    </option>
                    <option value="high"
                            :key="key">
                            High
                    </option>
                    <option value="medium"
                            :key="key">
                            Medium
                    </option>
                    <option value=""
                            :key="key">
                            No filter
                    </option>
                </select>
            </div>   
        </div>
        <table class="task-table">
            <tr>
                <td>Tasks</td>
                <td>Description</td>
                <td>Creation date</td>
                <td>Priotity</td>
            </tr>
            <tr v-for="(task, index) in loggedInUserTasks">
                <td>{{ task.title }}</td>
                <td>{{ task.description }}</td>
                <td>{{ task.priority }}</td>
                <td>{{ task.created_at }}</td>
                <td><button class="btn" @click="deleteTask(task.id)">Delete</button></td>
                <td><button class="btn" @click="editTaskDialog({id: task.id, 
                                                        title: task.title, 
                                                        description: task.description,
                                                        priority: task.priority,
                                                        assignee: task.assignee,
                                                        created_at: task.created_at})">Edit</button></td>
            </tr>
        </table>

        
        <dialog ref="addDialog" class="dialog">
            <h4>Task:</h4>
            <input type="text" v-model="title" class="input-field">
            <h4>Description:</h4>
            <input type="text" v-model="description" class="input-field">
            <h4>Priority:</h4>
            <input type="text" v-model="priority" class="input-field">
            <h4>Assignee:</h4>
            <select v-model="assignee" class="select-field">
                <option v-for="(user, key) in users" 
                        :value="user.id"
                        :key="key">
                    {{user.first_name + " " + user.last_name}}
                </option>
            </select>
            <h4>Creation date:</h4>
            <input type="date" v-model="date" class="input-field">
            <div class="dialog-buttons">
                <button @click="addNewTask" class="btn">Add</button>
                <button @click="closeTaskDialog" class="btn">Close</button>
            </div>
        </dialog>
        <dialog ref="editDialog" id="editDialog" class="dialog">
            <h4>Task:</h4>
            <input type="text" v-model="titleEdited" class="input-field">
            <h4>Description:</h4>
            <input type="text" v-model="descriptionEdited" class="input-field">
            <div class="dialog-buttons">
                <button @click="editTask" class="btn">Update</button>
                <button @click="closeEditTaskDialog" class="btn">Close</button>
            </div>
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

                if(this.priorityFilter != ''){
                    userTasks = this.tasks.filter((task)=>{
                            if(loggedInUser && (loggedInUser.id == task.assignee) && task.priority.includes(this.priorityFilter)){
                                return task
                            }
                        })
                        
                    return userTasks;
                }

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

<style scoped>
* {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
}

body {
    font-family: Arial, sans-serif;
    background-color: #f4f4f9;
    color: #333;
}

.task-list-container {
    max-width: 800px;
    margin: 2rem auto;
    padding: 2rem;
    background: #fff;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.user-name {
    font-size: 1.5rem;
    text-align: center;
    margin-bottom: 1rem;
}

.filter-controls {
    display: flex;
    justify-content: space-around;
    align-items: flex-end;
    margin-bottom: 1rem;
}

.filter-group {
    display: flex;
    flex-direction: column;
    align-items: center;
    margin: 0 1rem;
}

.filter-group h5 {
    margin-bottom: 0.5rem;
    font-size: .9rem;
}

.btn {
    display: inline-block;
    padding: 0.5rem 1rem;
    margin: 0.5rem;
    border: none;
    border-radius: 4px;
    background: #4CAF50;
    color: white;
    cursor: pointer;
}

.btn:hover {
    background: #45a049;
}

.select-field {
    padding: 0.5rem;
    border: 1px solid #ddd;
    border-radius: 4px;
    margin-top: 0.5rem;
}

.task-table {
    width: 100%;
    border-collapse: collapse;
    margin-bottom: 1rem;
}

.task-table th, .task-table td {
    border: 1px solid #ddd;
    padding: 0.5rem;
    text-align: left;
}

.task-table th {
    background-color: #f4f4f9;
    font-weight: bold;
}

.dialog {
    padding: 2rem;
    border-radius: 8px;
    background: #fff;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    left: 505px;
    top: 70px;
}

.dialog-buttons {
    display: flex;
    justify-content: space-between;
    margin-top: 1rem;
}

.input-field {
    width: 100%;
    padding: 0.5rem;
    border: 1px solid #ddd;
    border-radius: 4px;
    margin-bottom: 1rem;
}
</style>