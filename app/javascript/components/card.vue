<template>
  <div>
    <div @click="editing=true" class="card card-body mb-3">
      {{card.name}}
    </div>

  <div v-if="editing"  class="modal-backdrop show"></div>
    <div v-if="editing" @click="closeModal" class="modal show" style="display: block;">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h4 class="modal-header">{{card.name}}</h4>
          </div>
          <div class="modal-body">
            <input v-model="name" class="form-control">
          </div>
          <div class="modal-footer">
            <button @click="save" type="button" class="btn btn-primary">Save</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    props: ["card", "list"],
    data: function(){
      return {
        editing: false,
        name: this.card.name
      }
    },
    methods: {
      save: function(){
        var data = new FormData 
        data.append("card[name]", this.name)

        Rails.ajax({
          url: `/cards/${this.card.id}`,
          type: "PATCH",
          data: data,
          dataType: "json",
          success: (data) => {
            const listIndex = window.store.lists.findIndex((item) => {
              return item.id == this.list.id
            });
            const cardIndex = window.store.lists[listIndex].cards.findIndex((item) => {
              return item.id == this.card.id
            });

            window.store.lists[listIndex].cards.splice(cardIndex, 1, data)
            this.editing = false;
          }
        });
      },
      closeModal: (e) => {
        if(e.target.classList.contains("modal")){
          this.editing = false
        }
      }
    }
  }
</script>

<style scoped>

</style>
