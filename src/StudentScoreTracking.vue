<template>
    <div>
        <h1 class=" text-4xl font-bold text-blue-500">Student Score Tracking</h1>
        <h1 class="text-xl font-bold text-green-400">Total Score is:{{ totalScore }}</h1>
        <h1 class="text-xl font-bold text-blue-400">Score average is:{{ average }}</h1>
        <h1 class="text-xl font-bold text-black">The student name {{ studentName }} is {{ examStatus }}</h1>
    </div>
</template>

<script setup>

import { ref, computed, watch} from 'vue'

const studentName = ref("Mr, Ohknha")

const scores = ref([
    {subject: "English", score: 100},
    {subject: "Vue js", score: 10},
    {subject: "Laravel", score: 100}
]);

const passMark = ref(50);
const totalScore = computed(() => {
    let total = 0
    scores.value.forEach(item => {
        total += item.score
    });
    return total;
})
const average = computed(() => {
    return totalScore.value === 0 ? 0 : totalScore.value / scores.value.length;
});

const examStatus = computed(() => {
    if (passMark.value < average.value) {
        return "Passed"
    }
    return "Failed"
})

watch(scores, (newScore) => {
    localStorage.setItem("scores", JSON.stringify(newScore))
}, { deep: true })

</script>
