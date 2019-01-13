<template>
  <div class="list">
      <h4>{{list.name}}</h4>

      <draggable v-model="list.cards" :options="{group: 'cards'}" @change="cardMoved" class="dragArea">
        <card v-for="card in list.cards" :card="card" :list="list" ></card>
      </draggable>

        <a v-if="!editing" v-on:click="startEditing">Add a card</a>
        <textarea v-model="message" ref="message" v-if="editing" class="form-control mb-1" ></textarea>
        <button v-on:click="submitMessage" v-if="editing" class="btn btn-primary">Save</button>
        <a v-if="editing" v-on:click="editing=false">cancel</a>
  </div>
</template>

<script>
import draggable from 'vuedraggable'
import card from 'components/card'
export default {
  components: { draggable, card },
  props: ["list"],
  data: function() {
    return { 
      message: "",
      editing: false,
    }
  },

  methods: {
    startEditing: function(){
      this.editing = true
      this.$nextTick(() => {
        this.$refs.message.focus()
      })
    },
    cardMoved: function(e){
        const evt = e.added || e.moved
        if(evt == undefined){
          return
        }

        const element = evt.element
        
        const list_index = window.store.lists.findIndex(list => {
          return list.cards.find(card => {
            return card.id == element.id
          });
        });
         var data = new FormData 
         data.append('card[list_id]', window.store.lists[list_index].id)
        data.append("card[position]", evt.newIndex + 1) 

        Rails.ajax({
          url: `/cards/${element.id}/move`,
          type: "PATCH",
          data: data,
          dataType: 'json'
        })
      },

    submitMessage: function(){
        
        var data = new FormData
        data.append("card[list_id]", this.list.id)
        data.append('card[name]', this.message)

        Rails.ajax({
          url: "/cards",
          type: "POST",
          data: data,
          dataType: "json",
          success: (data) => {
            const index = window.store.lists.findIndex(item => item.id == this.list.id)
            window.store.lists[index].cards.push(data)
            this.message = ''
          }
        })
      }
  }
}

</script>

<style scoped>
  
.dragArea {
  min-height: 50px;
}

</style>

