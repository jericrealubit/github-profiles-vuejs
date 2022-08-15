

<template>
  <h2>Github Profiles</h2>
  <input type="text" placeholder="Github Username" v-model="username" @keyup.enter="getUser">
  <button @click="getUser">Search</button>
  
  <div class="row">
    <div class="column" style="background-color:#aaa;">
      <section v-if="loading">
        <img src="./assets/ajax.gif" alt="">
      </section>
      <section v-else>
        <img :src="avatar" alt="" width="150" id="avatar">
        <div>
          <ul class="profile-info">
            <li>Name: {{ name }}</li>
            <li>Username: {{ username }}</li>
            <li>Followers: {{ followers }}</li>
            <li>Repositories count: {{ public_repos }}</li>
          </ul>
        </div>   
      </section>         
    </div>
    <div class="column" style="background-color:#bbb;">
      <h2>Newest Repositories:</h2>
      <ul>
        <li v-for="repo in repos">
          <a :href="`${repo.html_url}`" target="_blank">{{ repo.name }}</a>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        username: '',
        avatar_url: '',
        name: '',
        followers: 0,
        public_repos: 0,
        repos: [],
        loading: false
      }
    },
    computed: {
        avatar() {
          return this.avatar_url || "./src/assets/profile-placeholder.png";
        }
    },
    methods: {
      getUser() {
        const apiUsers = "https://api.github.com/users/" + this.username;
        fetch(apiUsers)
        .then((response) => response.json())
        .then((data) => {
          if (data.message !== undefined) {
              alert(`Username: ${this.username} is ${data.message}`)
          } else {
            this.avatar_url = data.avatar_url;
            this.name = data.name
            this.followers = data.followers
            this.public_repos = data.public_repos

            this.loading = true;
            const apiRepos = "https://api.github.com/users/"+ this.username +"/repos";
            fetch(apiRepos)
              .then((response) => response.json())
              .then((data) => {
                this.loading = false;
                // reverse sort so we have the newest first
                data.sort(function(a, b) { 
                    return - ((a.id - b.id) || a.name.localeCompare(b.name));
                });

                this.repos = data.slice(0,4) // get the first 4 records
              }).catch((error) => {
                console.error('Error:', error);
              });
          }
        })
        .catch((error) => {
          console.error('Error:', error);
        });
      }
    }
  }
</script>

<style scoped>
li {
  text-align: left;
  font-weight: bold;
  list-style: none;
}
#avatar {
  width: 150px;
  height: 150px;
}
.profile-info li {
  font-weight: bold;
  list-style: none;
}
.row {
  width: 760px;
  padding-top: 10px;
}

.row > div {
  box-sizing: border-box;
  padding: 35px;
}

* {
  box-sizing: border-box;
}

/* Create two equal columns that floats next to each other */
.column {
  float: left;
  width: 50%;
  padding: 10px;
  height: 360px; 
}

/* Clear floats after the columns */
.row:after {
  content: "";
  display: table;
  clear: both;
}
</style>
