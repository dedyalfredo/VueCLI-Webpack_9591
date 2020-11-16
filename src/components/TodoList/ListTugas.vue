<template>
    <v-main class="list">
        <link href="https://cdn.jsdelivr.net/npm/@mdi/font@5.x/css/materialdesignicons.min.css" rel="stylesheet">
        <h3 class="text-h3 font-weight-medium mb-5">To Do List</h3>    
        <v-card>
            <v-card-title>
                <v-text-field
                    v-model="search"
                    append-icon="mdi-magnify"
                    label="Search"
                    single-line
                    hide-details
                ></v-text-field>
                <v-spacer></v-spacer>
                    <v-btn color="success" dark @click="dialog = true">
                        Tambah <v-icon dark right>mdi-checkbox-marked-circle</v-icon>
                    </v-btn>
                    <v-breadcrumbs></v-breadcrumbs>
                    <v-btn color="#FF3030" dark @click="deleteSelect">
                        Delete Selected <v-icon dark right>mdi-cancel</v-icon>
                    </v-btn>
            </v-card-title>

            <v-data-table 
                :headers="headers"
                :items="todos"
                :search="search"
                class="elevation-1">

                <template v-slot:[`item.priority`]="{ item }">
                    <v-chip v-if="item.priority == 'Penting'" color="red" label outlined>{{ item.priority }}</v-chip>
                    <v-chip v-if="item.priority == 'Biasa'" color="green" label outlined>{{ item.priority }}</v-chip>
                    <v-chip v-if="item.priority == 'Tidak penting'" color="blue" label outlined>{{ item.priority }}</v-chip>
                </template>

                <template v-slot:[`item.actions`]="{ item }">
                    <v-btn small class="mr-2" @click="editItem(item)">
                        edit
                    </v-btn>
                    <v-btn small @click="deleteItem(item)">
                        delete
                    </v-btn>
                </template>
                <template v-slot:[`item.check`]="{ item }">
                    <v-simple-checkbox 
                        v-model="item.check"
                    ></v-simple-checkbox>
                </template>
            </v-data-table>
        </v-card>

        <v-dialog v-model="dialog" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline">Form Todo</span>
                </v-card-title>
                <v-card-text>
                    <v-container>
                        <v-text-field
                            v-model="formTodo.task"
                            label="Task"
                            required
                        ></v-text-field>
                        <v-select
                            v-model="formTodo.priority"
                            :items="['Penting', 'Biasa', 'Tidak penting']"
                            label="Priority"
                            required
                        ></v-select>
                        <v-textarea
                            v-model="formTodo.note"
                            label="Note"
                            required
                        ></v-textarea>
                    </v-container>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="cancel">
                        Cancel
                    </v-btn>
                    <v-btn color="blue darken-1" text @click="save">
                       Save
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
        <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
            <v-card-title class="headline">Are you sure you want to delete this item?</v-card-title>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="closeDelete">Cancel</v-btn>
              <v-btn color="blue darken-1" text @click="deleteItemConfirm">OK</v-btn>
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
    </v-main>
</template>
<script>
export default {
    name: "List",
    data() {
        return {
            dialogDelete: false,
            search: null,
            dialog: false,
            completed: false,
            editedIndex: -1,
            headers: [
                {
                    text: "Task",
                    align: "start",
                    sortable: true,
                    value: "task",
                },
                { text: "Priority", value: "priority" },
                { text: "Note", value: "note" },
                { text: "Actions", value: "actions" },
                { text: 'Delete Selected', value: 'check' },
            ],
            todos: [
                {
                    task: "bernafas",
                    priority: "Penting",
                    note: "huffttt",
                    check: false,
                },
                {
                    task: "nongkrong",
                    priority: "Tidak penting",
                    note: "bersama tman2",
                    check: false,
                },
                {
                    task: "masak",
                    priority: "Biasa",
                    note: "masak air 500ml",
                    check: false,
                },
            ],
            filters: {
                notDone: function(todo){
                    return !todo.check;
                },
            },
            formTodo: {
                task: null,
                priority: null,
                note: null,
                check: false,
            },
        };
    },
    computed: {
      formTitle () {
        return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
      },
    },
    watch: {
        dialog (val) {
            val || this.close()
        },
        dialogDelete (val) {
            val || this.closeDelete()
        },
    },
        created () {
            this.initialize()
    },
    methods: {
        save() {
             if (this.editedIndex > -1) {
                 Object.assign(this.todos[this.editedIndex], this.formTodo)
                 this.resetForm();
                this.dialog = false;
                this.editedIndex=-1;
        } else {
            this.todos.push(this.formTodo);
            this.resetForm();
            this.dialog = false;
        }
        },
        cancel() {
            this.resetForm();
            this.dialog = false;
        },
        resetForm() {
            this.formTodo = {
                task: null,
                priority: null,
                note: null,
            };
        },
        editItem(item) {
            this.editedIndex = this.todos.indexOf(item)
            this.formTodo = Object.assign({}, item)
            this.dialog = true
        },
        deleteItem(item) {
            this.editedIndex = this.todos.indexOf(item)
            this.formTodo = Object.assign({}, item)
            this.dialogDelete = true
        },
        deleteItemConfirm() {
            this.todos.splice(this.editedIndex, 1)
            this.closeDelete()
        },
            
        deleteSelect: function () {
            this.todos = this.todos.filter(this.filters.notDone);
        },
        
        closeDelete() {
            this.dialogDelete = false
            this.$nextTick(() => {
                this.formTodo = Object.assign({}, this.defaultItem)
                this.editedIndex = -1
            })
        },
    },
};
</script>