<template>
    <div>
        <el-input v-model="name" placeholder="Имя события" @change="updateNameEvent"></el-input>
        
    </div>
</template>

<script lang="ts">
import { ReduxType } from "@/views/Home.vue";
import { Vue, Options } from "vue-class-component";

@Options({
props: {
    name: String,
    type: String,
    payload: Array,
}
})
export default class Event extends Vue {
    name!: string
    type!: string
    payload!: ReduxType[]

    updateNameEvent(name: string) {
        console.log(name);
        
    }
}
</script>