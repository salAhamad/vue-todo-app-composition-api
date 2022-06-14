<script setup>
	import { ref, watch, onMounted, onBeforeUpdate } from 'vue'
  
  const msg = ref('Welcome to Vue')
  const todos = ref([]);
  const priority = ref(false);
  const selectPriority = ref('');
  const editingItem = ref(false);
  const udatableItem = ref(null);
  
  const wishList = ref([]);
  
	const inputValue = ref('');
  
  const addNewItem = (el) => {  
    el.preventDefault();
   if(inputValue.value !== '') {
      todos.value.push({ 
        id: new Date().getTime(), 
        content: inputValue.value,
        priority: priority.value,
        status: false,
      },);
   } else {
     alert('Please, enter the text')
   }
    inputValue.value = '';
    priority.value = false;
  }  
  const deleteItem = (deletable_item_id) => {
    todos.value = todos.value.filter(item_id => item_id.id !== deletable_item_id)
  }
  
  const editItem = (item) => {
    udatableItem.value = item;
    inputValue.value = item.content;
    editingItem.value = true;
    priority.value = item.priority
  }
  const editCancel = () => {
    editingItem.value = false;
    udatableItem.value = null;    
    inputValue.value = '';
    priority.value = false;
  }
  const changeStatus = (item) => {
    item.status = !item.status
    priority.value = false;
  }
  const updateItem = () => {
    if(udatableItem.value !== null) {
    	udatableItem.value.content = inputValue.value;
      udatableItem.value.priority = priority.value;
      udatableItem.value.status = false;
    }    
    udatableItem.value = null;    
    inputValue.value = '';
    priority.value = false;
  }

  watch(todos, (newItem) => {
    localStorage.setItem('todoItemList', JSON.stringify(newItem))
  }, {deep: true})
  
  watch(wishList, (newItem) => {
    localStorage.setItem('wishlists', JSON.stringify(newItem))
  }, {deep: true})
  
  onMounted(() => {
    todos.value = JSON.parse(localStorage.getItem('todoItemList')) || [];
    wishList.value = JSON.parse(localStorage.getItem('wishlists')) || [];
	})
  onBeforeUpdate(() => {
    todos.value = todos.value.sort((a, b) => b.priority - a.priority)
    todos.value = todos.value.sort((a, b) => a.status - b.status)
  })
  
</script>

<template>
  <h1>{{ msg }}</h1>
  {{selectPriority}}
  <hr />
  <form style="display: flex;  justify-content: center">
    <input type="text" @:keyup.enter="addNewItem(e)" v-model.trim="inputValue" placeholder="Enter Text" style="width: 50%; max-width: max-content" />
    <div style="margin: 0 1rem;">
      <input type="checkbox" id="priority" v-model="priority" />
      <label for="priority">High Priority</label>
    </div>
    <select v-model="selectPriority" style="display: none;">
      <option value="" disabled> Select a category</option>
      <option value="high">High</option>
      <option value="low">Low</option>
    </select>
    
  	<button v-if="!editingItem" style="margin-left: 1rem" type="button" v-on:click.prevent="addNewItem">
    	Add New
    </button>
    <button v-if="editingItem" style="margin-left: .5rem" type="button" v-on:click.prevent="editCancel">
    	Cancel
    </button>
    <button v-if="editingItem" style="margin-left: .5rem" type="button" v-on:click.prevent="updateItem">
    	Update Item
    </button>
    
  </form>
  <hr />
  <div style="display: none; gap: 1rem; flex-wrap: wrap;">
    <div>
    	<input type="checkbox" id="mango" value="mango" v-model="wishList" />
      <label for="mango">Mango</label>
    </div>
    <div>
    	<input type="checkbox" id="orange" value="orange" v-model="wishList" />
      <label for="orange">orange</label>
    </div>
  </div>
	<br />
  <ul class="list-group">
    <li 
      class="list-item" 
      v-for="(item, index) in todos" 
      :key="item.id"
      :class="{priority: item.priority, completed: item.status}"
    >
      <span style="display: flex;">
        <span class="indexing" :class="{strikeout: item.status}">{{index + 1}}</span>
        <span class="content" :class="{strikeout: item.status}">{{item.content}}</span>
      </span>
      <span class="actions">
        <span class="status" :class="{completed: item.status}" @click="changeStatus(item)">
          {{!item.status ? 'Pending' : 'Completed'}}
        </span>
        <span class="edit" @click="editItem(item)">Edit</span>
        <span class="delete" @click="deleteItem(item.id)">Delete</span>
      </span>
    </li>
    <li class="empty" v-if="todos.length <= 0">Nothing Show </li>
  </ul>
</template>
<style>
  html {
    font-size: 20px;
    font-family: sans-serif;
  }
  h1 {
    text-align: center;
  }
  .list-group {
    width: 100%;
    max-width: 700px;
    margin: 0;
    margin-inline: auto;
    list-style: none;
    padding: 0;
  }
  .list-item {
    border-bottom: 1px solid #dddddd;
    padding: .75rem 0 .75rem 1.25rem;
    position: relative;
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 1rem;
    box-shadow: inset 6px 0 0 0 #e6e6e6;
  }
  .list-item:first-child { border-top: 1px solid #dddddd; }
  
  .list-item.priority { box-shadow: inset 6px 0 0 0 orange; }
  .list-item.completed {
    box-shadow: inset 6px 0 0 0 darkgreen;
    background-color: rgb(139 195 74 / 10%);
  }
  .list-item span { display: inline-block; }
  .list-item span.indexing {
    margin: 0 1rem 0 0;
  }
  .list-item span.actions {
    position: relative;
    right: 0;
    display: flex;
    gap: .5rem;
  }
  .list-item.completed span.actions {right: .785rem;}
  .list-item span.actions span {
    padding: .85rem 0.6rem;
    color: white;
    cursor: pointer;
    line-height: 0;
    border-radius: 4px;
    font-size: 16px;
	}
  .list-item span.actions span.edit { background: purple; }
  .list-item span.actions span.status { background: skyblue; }
  .list-item span.actions span.status.completed { background: darkgreen; }
  .list-item span.actions span.delete { background: red; }
  li.empty {
    background-color: #f5f5f5;
    padding: 1rem;
    text-align: center;
  }
  .strikeout {
    text-decoration: line-through;
    opacity: .5;
  }
  
</style>