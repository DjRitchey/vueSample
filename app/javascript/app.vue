<!-- v-model => element or component having prop with same namespace i.e. v-model="email" has a prop: email: email.url -->
<!-- use v in tags for things such as loop or condition such as v-if="true" --> 
<!-- @ signifies a action to a func @change="changeMethod" -->
<!-- using v-click handles a listener to fire  --> 
<!-- props taken from somewhere to get values, such as a json list passed to it -->
<!-- used the window.store  to access persisted data, and to keep everything uniform across all my components-->
<!-- webpacker made this easy, but we use gulp? --> 
<!-- props pass down into children such as from app to list :list="list" -->


<template>
  <draggable :list="lists" :options="{group: 'lists'}" class='board dragArea' @end="listMoved">
    
    <list v-for="(list, index) in original_lists" :list="list" :key="list.name">
    </list>
      
        
        <div class='list'>
          <a v-if="!editing" v-on:click="startEditing">Add a list</a>
          <textarea v-model="message" ref="message" v-if="editing" class="form-control mb-1" ></textarea>
          <button v-on:click="submitMessage" v-if="editing" class="btn btn-primary">Save</button>
          <a v-if="editing" v-on:click="editing=false">cancel</a>
      </div>

  </draggable>
</template>

<script>
import draggable from 'vuedraggable'
import list from 'components/list'

export default {
  components: { draggable, list },
  props: ["original_lists"],
  data: function() {
    return {
      lists: this.original_lists,
      editing: false,
      message: ''
      }
    },

    methods: {
      startEditing: function(){
      this.editing = true
      this.$nextTick(() => {
        this.$refs.message.focus()
      })
    },
      listMoved: function(e){
        var data = new FormData 
        data.append("list[position]", e.newIndex + 1)
        Rails.ajax({
          url: `/lists/${this.lists[e.newIndex].id}/move`,
          type: "PATCH",
          data: data,
          dataType: "json"
        })
      },

      submitMessage: function(){
        
        var data = new FormData
        data.append("list[name]", this.message)
        

        Rails.ajax({
          url: "/lists",
          type: "POST",
          data: data,
          dataType: "json",
          success: (data) => {
            window.store.lists.push(data)
            this.message = ''
            this.editing = false
          }
        });
      },
      
    }
  }

</script>

<style scoped>
.dragArea {
  min-height: 50px;
}
p {
  font-size: 2em;
  text-align: center;
}

.board {
  white-space: nowrap;
  overflow-x: auto;
}

.list {
  background: #E2E4E5;
  width: 250px;
  margin-right: 15px;
  padding: 10px;
  vertical-align: top;
  display: inline-block;
  border-radius: 3px;
}
</style>
