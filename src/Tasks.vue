<template>
  <div>
    <div class="modal fade" id="modal" tabindex="-1" role="dialog">
      <div class="modal-dialog" role="document">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Редактирование</h5>
            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <div class="modal-body">
            <form @submit.stop.prevent="post">
              <div class="form-group">
                <label for="name">Задание</label>
                <input type="text" v-model="name" id="name" class="form-control">
              </div>
              <div class="form-group">
                <select v-model="status" class="form-control">
                  <option value="1">Завершен</option>
                  <option value="0">Не завершен</option>
                </select>
              </div>
              <button type="submit" class="btn btn-primary">Отправить</button>
            </form>
          </div>
        </div>
      </div>
    </div>
    <div class="container main-holder">
      <div class="col-md-12">
        <button type="button" class="btn btn-primary" data-toggle="modal" data-target="#modal" v-on:click="modal()">Добавить</button>
        <form>
          <div class="form-group">
            <input type="text" class="form-control" placeholder="По названию" v-model="filterByName">
          </div>
        </form>
        <table class="table table-hover">
          <thead>
            <tr>
              <th scope="col">#</th>
              <th scope="col" v-on:click = "sortBy('task')">Задача</th>
              <th scope="col" v-on:click = "sortBy('completed')">Статус</th>
              <th scope="col">Действия</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, index) in filterTasks" data-toggle="modal" data-target="#modal" v-on:click="modal(index)">
              <th scope="row">{{item.id}}</th>
              <td>{{item.task}}</td>
              <td v-html="$options.filters.normalize(item.completed)" ></td>
              <td>
                  <a href="#" v-on:click.stop.propogation="del(index)">Удалить</a><br/>
                  <a href="#" v-on:click.stop.propogation="changeStatus(index)">Изменить статус</a>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
export default {
    data(){
        return{
          tasks : [],
          tasksJSON : './src/data.json',
          name : "",
          status : 0,
          desc : false,
          filterByName : ''
        }
    },
    methods: {
      getTasks: function(){
        this.$http.get(this.tasksJSON).then(function(response) {
          this.tasks = JSON.parse(response.bodyText);
        }, function(error) {
            console.log(error);
        })
      },
      del: function(index){
        this.tasks.splice(index, 1);
      },
      changeStatus: function(index){
        switch (this.tasks[index].completed){
          case 1:
            this.tasks[index].completed = 0;
            break;
          case 0:
            this.tasks[index].completed = 1;
            break;
          default:
            break;
        }
      },
      modal : function(index){
        if(index !== undefined){
          this.name = this.tasks[index].task.toUpperCase();
          this.status = this.tasks[index].completed;
          this.index = index;
        }else{
          this.name = "";
          this.status = 0;
          this.index = undefined;
        }
      },
      post : function(){
        if(this.tasks[this.index]){
          this.tasks[this.index].task = this.name;
          this.tasks[this.index].completed = parseInt(this.status);
        }else{
          this.tasks.push({"id":(this.tasks.length + 1),"task":this.name.toUpperCase(),"completed":parseInt(this.status)});
        }
        $('#modal').modal('hide');
      },
      sortBy: function(param){
        this.desc = !this.desc;
        this.tasks = this.tasks.sort((a, b) => {
          if(this.desc)
            return a[param] < b[param]
          else
            return a[param] > b[param]
        })
      }

    },
    filters: {
      normalize: function (value) {
        switch (value){
          case 1:
            value = '<span class="badge badge-success">Завершено</span>';
            break;
          case 0:
            value = '<span class="badge badge-warning">Не завершено</span>';
            break;
          default:
            value = '<span class="badge badge-danger">Ошибка</span>';
        }
        return value;
      }
    },
    created: function() {
      this.getTasks();
    },
    computed:{
      filterTasks : function(){
        return this.tasks.filter((task) => {
          return task.task.match(this.filterByName.toUpperCase())
        })
      }
    }
}
</script>