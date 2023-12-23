<template>
  <div class="max-w-3xl my-20 py-12 mx-60">
    <div class="items-center justify-between flex">
      <h1 class="text-3xl"><strong>List</strong></h1>
      <Button icon-left="plus">New List</Button>
    </div>

    <div class="mt-2 ">
      <Card  title="General">
        <ul>
          <li class="flex justify-between" v-for="action in actions.data" :key="action.title">

          <span>   {{ action.title }} </span>
            <Button @click="completeAction(action.name)" icon="check"></Button>
           
          
          </li>
          <Button @click="Dailogshown = true" icon-left="plus">New Task</Button>
          {{ actiontask.data }}
        </ul>
      </Card>
    </div>
 <Dialog
  :options="{
    title: 'Add new Task',
    size: 'xl',
    actions: [
      {
        label: 'Add Task',
        variant: 'solid',
        onClick: () => {
               addTask() },
      },
    ],
  }"
  v-model="Dailogshown"
>
<template #body-content>
    <div class="space-y-10">
      <Input type="text" label="Title" placeholder="enter your task" v-model="action.title" required />
      <Input type="select" :options="optionsss" label="category" v-model="action.category"/>
      <Input type="date" v-model="action.due_date" label="Due Date"/>
    </div>
  </template>
</Dialog>
  </div>
</template>

<script setup>
import { reactive } from 'vue';
import { ref, onMounted , computed } from 'vue'
import { createListResource, Card , Dialog, Input } from 'frappe-ui'

const Dailogshown = ref(false)

const actions = createListResource({
  doctype: 'Actions',
  fields: ["name","title", "status", "description", "date", "due_date","task"],
  filters: {
    status: "To Do"
  },
  limit: 10
})

//dynamically fetching doctypes and passing it to the select options
const categories = createListResource({
  doctype: 'Category',
  fields: ["title"],
  transform(categories){
    return categories.map((c) => ({label:c.title, value:c.name}))
  }
})

const optionsss =computed(() => {
  if(categories.list.loading || !categories.data){
    return []
  }
 return categories.data
})

//change the value of the status in the original doctypes 
const completeAction = (actionName) =>{
  actions.setValue.submit({
    name:actionName,
    status:'Done'

  })
}


//redirecting the add task menu
const addTask = () =>{
  actions.insert.submit(action)
}

const action = reactive({
  title:'',
  category:'General'
})

const actiontask = createListResource({
  doctype: 'Action task',
  fields: ["description","is_completed"],

  limit: 10
})


onMounted(() => {
  actions.reload()
  categories.reload()
  actiontask.reload()
})
</script>
