<template>
    <div class="w-8/12 m-auto px-24 py-12 border border-amber-200 rounded-2xl">
        <h1 class="text-red-600 text-center text-xl">Hello Task manager build</h1>

        <form v-show="isShowForm">
            <label for="task-title">Task list</label><br>
            <input type="text" class="px-2 w-full mt-4 py-2 border border-amber-100 rounded-md ">

            <label for="task-priority">Task Priority</label>
            <select class="px-4 w-full mt-2 py-2 border border-gray-300 rounded-md" name="priotity" id="">Priority Task 
                <option value="High">High</option>
                <option value="Mediem">Mediem</option>
                <option value="Low">Low</option>
            </select>
            <button class="w-full py-2 bg-green-500 text-white rounded-md mt-4">Add agent task</button>
        </form>
        <div class="flex justify-between items-center mt-4">
            <p class="text-gray-500 text-md">All task below</p>
            <button class="py-2 px-4 bg-blue-500 text-black rounded-md" @click="openForm()">Create Task</button>
        </div>
    </div>
</template>

<script setup>
import { ref } from 'vue'
const isShowForm = ref(false);
const openForm = () => {
    isShowForm.value = !isShowForm.value;
}
</script>

<style lang="scss" scoped>

</style>