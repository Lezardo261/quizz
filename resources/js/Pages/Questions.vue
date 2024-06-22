<script setup>
import Layout from '@/Shared/Layout.vue';
import NewQuestionModal from '@/Shared/NewQuestionModal.vue';
import {ref} from 'vue'
import {usePage} from '@inertiajs/vue3'
import { router } from '@inertiajs/vue3';
import {computed} from 'vue'
const page = usePage()
const success = computed(() => page.props.flash.success)
const showNewQuestionModal = ref(false)
const createdQuestions = ref(null)
const newAnswers = ref([])
const selectedAnswer = ref(null)
let answersId = 1


function  createQuestion() {
    showNewQuestionModal.value = true
}
function  destroyModal() {
    showNewQuestionModal.value = false
}


function addNewAnswers() {
  // Create a new answer object with unique ID, empty answer text, and initial correctness state
  const newAnswer = {
    id: answersId++,
    answer: '', // Initialize answer text to an empty string
    correct_answer: 0 // Initially not marked as correct
  };

  // Push the new answer object to the `newAnswers` array
  newAnswers.value.push(newAnswer);
}

function handleRadioToggle(Id){
    selectedAnswer.value = Id
    newAnswers.value.forEach(answer =>    {
        if(answer.id === Id){
            answer.correct_answer = 1
        }else{
            answer.correct_answer = 0 
        }
    });
}
function validateAnswer(){
    for(const answer of newAnswers.value){
        if(answer.answer.trim() === ''){
            return false
        }
    }
    return true
}
function answerCount(){
    if(newAnswers.length<4){
        alert('Four Answer Required');
    }else if(newAnswers.length===4){
        return true
    }
    return false
}

function submitQuestion(){
    if(!createdQuestions.value){
        alert('Cannot be empty ');
        return false
    }
    if(!validateAnswer() && !answerCount()){
        alert('Fill All Input');
        return false
    }
    router.post('/questions',{
        question:createdQuestions.value,
        answer:newAnswers.value
    })
    router.on('success', () => {
        // showNewQuestionModal.value = false
        createdQuestions.value = null,
        newAnswers.value = []
    })
}
</script>
<template>
    <Layout>
        <button @click="createQuestion" class="btn btn-success">Create</button>
        <table class="table table-striped">
            <thead>
                <tr>
                    <th scope="col">#</th>
                    <th scope="col">First</th>
                    <th scope="col">Last</th>
                    <th scope="col">Handle</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <th scope="row">1</th>
                    <td>Mark</td>
                    <td>Otto</td>
                    <td>@mdo</td>
                </tr>
            </tbody>
        </table>
        <Teleport to="body">
            <NewQuestionModal :show="showNewQuestionModal" @close="destroyModal">
                <template #header>
                    <h5>Add New Question</h5>
                </template>
                <template #success>
                    <div v-if="success" class="alert alert-success">{{ success }}</div>
                </template>
                <template #body>
                    <form>
                        <div class="mb-3">
                            <label for="exampleInputEmail1" class="form-label">Question</label>
                            <input type="text" v-model="createdQuestions" class="form-control" id="exampleInputEmail1" aria-describedby="emailHelp">
                        </div>
                    <table class="table">
                        <thead>
                            <tr>
                            <th scope="col">#</th>
                            <th scope="col">Answers</th>
                            <th scope="col">Correct</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="(answer, index) in newAnswers">
                                <th scope="row">{{answer.id}}</th>
                                <td>

                                    <input type="text" v-model="answer.answer" class="form-control" id="exampleInputEmail1" aria-describedby="emailHelp">
                                </td>
                                <td>
                                    <input :checked="answer.correct_answer == 1" class="form-check-input" :value="answer.id" @change="handleRadioToggle(answer.id)" type="radio">
                                </td>
                            </tr>
                        </tbody>
                    </table>
                    </form>

                </template>
                <template #footer>
                    <span @click="addNewAnswers"  v-if="newAnswers.length<4" ><h3>+</h3></span>
                    <button @click="destroyModal" class="btn btn-danger">Close</button>
                    <button v-if="newAnswers.length>3" @click="submitQuestion" class="btn btn-success">Submit</button>
                </template>
            </NewQuestionModal>
        </Teleport>
    </Layout>
</template>