<script setup>
import Layout from '@/Shared/Layout.vue';
import NewQuestionModal from '@/Shared/NewQuestionModal.vue';
import {ref} from 'vue'
import {usePage} from '@inertiajs/vue3'
import { router } from '@inertiajs/vue3';
import {computed} from 'vue'
const page = usePage()
const success = computed(() => page.props.flash.success || '');
const showNewQuestionModal = ref(false)
const showViewQuestionModal = ref(false)
const createdQuestions = ref(null)
const newAnswers = ref([])
const answers = ref([])
const selectedAnswer = ref(null)
const selectedQuestions = ref(null)

let answersId = 1



function  createQuestion() {
    showNewQuestionModal.value = true
}
function  destroyModal() {
    showNewQuestionModal.value = false
    showViewQuestionModal.value = false
    window.sessionStorage.removeItem('success'); 
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

const props = defineProps({
    questions: Object,
    errors: Object,
});
function viewQuestion(index){
    showViewQuestionModal.value=true;
    selectedQuestions.value = props.questions[index];
    answers.value = props.questions[index].answer;
}
// handle radio and submit
const selectedEditAnswer = ref(null)
function handleRadioChange(Id){
    selectedEditAnswer.value =Id

    answers.value.forEach(answer => {
        if(answer.id == Id){
            answer.correct_answer = 1
        }else{
            answer.correct_answer = 0
        }
    })
}
function updateAnswers(){
    router.put('/answers',
        answers.value
    )
}
function CloseModals(){
    router.get('/questions',
        answers.value
    )
};
//edits questions
const editquestionModal=ref(false)
const questionsForEdit=ref(null)

function editQuestion(index){
    questionsForEdit.value=props.questions[index];

}
function updateQuestion(){
    router.put('/questions',
        questionsForEdit.value
    )
}

function DeleteQuizz(id) {
  const confirmDelete = confirm('Are you sure you want to delete?')

  if (confirmDelete) {
    router.delete('/questions/' + id) // Perform deletion
  }
}


</script>
<template>
    <Layout>
        <button @click="createQuestion" class="btn btn-success">Create</button>
        <table class="table table-striped">
            <thead>
                <tr>
                    <th scope="col" class="text-center">#</th>
                    <th scope="col">Questions</th>
                    <th scope="col" class="text-center">Actions</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(question, index) in questions">
                    <th scope="row" class="text-center">{{ index+1 }}</th>
                    <td>{{ question.question  }}</td>
                    <td class="d-flex gap-4 justify-content-center">
                        <button @click="viewQuestion(index)" class="btn btn-primary">View</button>
                        <button  @click="editquestionModal=true, editQuestion(index)" class="btn btn-warning">Edit</button>
                        <button @click="DeleteQuizz(question.id)" class="btn btn-danger">Delete</button>
                    </td>
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
                                    <input :checked="answer.correct_answer === 1" class="form-check-input" :value="answer.id" @change="handleRadioToggle(answer.id)" type="radio">
                                </td>
                            </tr>
                        </tbody>
                    </table>
                    </form>

                </template>
                <template #footer>
                    <span @click="addNewAnswers"  v-if="newAnswers.length<4" ><h3>+</h3></span>
                    <button @click="CloseModals" class="btn btn-danger">Close</button>
                    <button v-if="newAnswers.length>3" @click="submitQuestion" class="btn btn-success">Submit</button>
                </template>
            </NewQuestionModal>
            
            <NewQuestionModal :show="showViewQuestionModal" @close="destroyModal">
                <template #header>
                    <h5>View Questions/Answers</h5>
                </template>
                <template #success>
                    <div v-if="success" transition="fade" class="alert alert-success">{{ success }}</div>
                </template>
                <template #body>
                     <p><strong>Q. {{ selectedQuestions.question }}</strong></p>
                     
                     <table class="table">
                        <thead>
                            <tr>
                            <th scope="col">#</th>
                            <th scope="col">Answers</th>
                            <th scope="col">Correct</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="(answer, index) in answers">
                                <th scope="row">{{index+1}}</th>
                                <td>

                                    <input type="text" v-model="answer.answer" class="form-control" id="exampleInputEmail1" aria-describedby="emailHelp">
                                </td>
                                <td>
                                    <input :checked="answer.correct_answer === 1" class="form-check-input" :value="answer.id" @change="handleRadioChange(answer.id)" type="radio">
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </template>
                <template #footer>
                    <button @click="CloseModals" class="btn btn-danger">Close</button>
                    <button  @click="updateAnswers" class="btn btn-success">Submit</button>
                </template>
            </NewQuestionModal>

            <NewQuestionModal :show="editquestionModal"  >
                <template #header>
                    <h5>Edit Questions</h5>
                </template>
                <template #success>
                    <div v-if="success" transition="fade" class="alert alert-success">{{ success }}</div>
                </template>
                <template #body>
                    <div class="mb-3">
                        <label for="exampleInputEmail1" class="form-label">Questions</label>
                        <input v-model="questionsForEdit.question" type="text" class="form-control" id="exampleInputEmail1" aria-describedby="emailHelp">
                    </div>
                </template>
                <template #footer>
                    <button @click="CloseModals" class="btn btn-danger">Close</button>
                    <button  @click="updateQuestion" class="btn btn-success">Submit</button>
                </template>
            </NewQuestionModal>
        </Teleport>
    </Layout>
</template>