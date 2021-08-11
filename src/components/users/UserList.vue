<template>
  <base-container>
    <h2>Active Users</h2>
    <base-search @search="updateSearch" :search-term="enteredSearchTerm"></base-search>
    <div>
      <button @click="sort('asc')" :class="{selected: sorting === 'asc'}">Sort Ascending</button>
      <button @click="sort('desc')" :class="{selected: sorting === 'desc'}">Sort Descending</button>
    </div>
    <ul>
      <user-item
        v-for="user in displayedUsers"
        :key="user.id"
        :user-name="user.fullName"
        :id="user.id"
        @list-projects="$emit('list-projects', $event)"
      ></user-item>
    </ul>
  </base-container>
</template>

<script>
    import UserItem from './UserItem.vue';
    import {
        ref,
        computed,
        watch
    } from 'vue';
    export default {
        components: {
            UserItem,
        },
        // users gets provided via props for the computed function
        props: ['users'],
        emits: ['list-projects'],
        // We want to pass props to the setup() function as an argument
        // In this case props will have a user key 
        setup(props) {
            /////////////// Search related ///////////////
            const enteredSearchTerm = ref('');
            // Search term is empty to store user input
            const activeSearchTerm = ref('');

            const availableUsers = computed(function() {
                let users = [];
                if (activeSearchTerm.value) {
                    // props instead of value is used because props is being returned 
                    // We pass in props before any place we reference users
                    users = props.users.filter((usr) =>
                        usr.fullName.toLowerCase().includes(activeSearchTerm.value.toLowerCase())
                    );
                } else if (props.users) {
                    users = props.users;
                }
                return users;
            });

            // For the watch we pass in the dependency it should watch for
            // In this case it is the "enteredSearchTerm"
            watch(enteredSearchTerm, function(newValue) {
                setTimeout(() => {
                    if (newValue === enteredSearchTerm.value) {
                        activeSearchTerm.value = newValue;
                    }
                }, 300);
            });

            // At the end of our search logic we return the updated search value 
            // val is the value chosen by the user
            function updateSearch(val) {
                enteredSearchTerm.value = val;
            }


            /////////////// Sorting related ///////////////
            const sorting = ref(null);

            const displayedUsers = computed(function() {
                if (!sorting.value) {
                    // we use .value here because computed properties are stored as read only refs (as in the ref above)
                    return availableUsers.value;
                }
                // Slice is used to return a new sorted array so we don't sort the existing one
                return availableUsers.value.slice().sort((u1, u2) => {
                    if (sorting.value === 'asc' && u1.fullName > u2.fullName) {
                        return 1;
                    } else if (sorting.value === 'asc') {
                        return -1;
                    } else if (sorting.value === 'desc' && u1.fullName > u2.fullName) {
                        return -1;
                    } else {
                        return 1;
                    }
                });
            });

            function sort(mode) {
                sorting.value = mode;
            }

            return {
                enteredSearchTerm,
                updateSearch,
                displayedUsers,
                sorting,
                sort
            };
        },
        /////////////////////// Original Data 
        // data() {
        //     return {
        //         enteredSearchTerm: '',
        //         activeSearchTerm: '',
        //         sorting: null,
        //     };
        // },
        /////////////////////// Original Computed
        // computed: {
        //     availableUsers() {
        //         let users = [];
        //         if (this.activeSearchTerm) {
        //             users = this.users.filter((usr) =>
        //                 usr.fullName.includes(this.activeSearchTerm)
        //             );
        //         } else if (this.users) {
        //             users = this.users;
        //         }
        //         return users;
        //     },
        //     displayedUsers() {
        //         if (!this.sorting) {
        //             return this.availableUsers;
        //         }
        //         return this.availableUsers.slice().sort((u1, u2) => {
        //             if (this.sorting === 'asc' && u1.fullName > u2.fullName) {
        //                 return 1;
        //             } else if (this.sorting === 'asc') {
        //                 return -1;
        //             } else if (this.sorting === 'desc' && u1.fullName > u2.fullName) {
        //                 return -1;
        //             } else {
        //                 return 1;
        //             }
        //         });
        //     },
        // },
        //////////////////////// Original methods
        // methods: {
        //     updateSearch(val) {
        //         this.enteredSearchTerm = val;
        //     },
        //     sort(mode) {
        //         this.sorting = mode;
        //     },
        // },
        // watch: {
        //     enteredSearchTerm(val) {
        //         setTimeout(() => {
        //             if (val === this.enteredSearchTerm) {
        //                 this.activeSearchTerm = val;
        //             }
        //         }, 300);
        //     }
        // },
    };
</script>

<style scoped>
    ul {
        list-style: none;
        margin: 0;
        padding: 0;
    }
</style>