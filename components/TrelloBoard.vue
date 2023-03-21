<script setup lang="ts">
import type { Column, Task } from '~~/types';
import draggable from "vuedraggable"
import { nanoid } from "nanoid";
const columns = useLocalStorage<Column[]>("trelloBoard",[
    {
        id: nanoid(),
        title: "Backlog",
        tasks: [
            {id: nanoid(), title: "Create marketing landing page", createdAt: new Date()},
            {id: nanoid(), title: "Bug on nav page", createdAt: new Date()},
            {id: nanoid(), title: "Cool new landing page", createdAt: new Date()},
        ],

    },
    { id: nanoid(),title: "Selected for Dev",tasks: []},
    { id: nanoid(),title: "In Progress",tasks: []},
    { id: nanoid(),title: "QA",tasks: []},
    { id: nanoid(),title: "Complete",tasks: []},
]);
/*watch(columns, ()=>{
    //ajax request
}, {
    deep: true,
}*/

const alt = useKeyModifier("Alt")

function createColumn(){
    const column: Column = {
        id: nanoid(),
        title: "",
        tasks:[],
    };
    columns.value.push(column);
    nextTick(() => {
        (
            document.querySelector(
                ".column:last-of-type .tilte-input"
            )as HTMLInputElement
        ).focus();
    });
    
}
</script>
<template>
    <div class="flex items-start overflow-x-auto gap-4">
        <draggable
            v-model="columns"
            group="columns"
            :animation="150"
            handel=".drag-handle"
            item-key="id"
            class=" flex gap-4 items-start"
        >
            <template #item="{element: column}: {element: Column}">
                <div class="column bg-gray-200 p-5 rounded min-w-[250px]">
                    <header class="front-bold mb-4">
                        <DragHandle/>
                        <input
                            class="tilte-input bg-transparent focus:bg-white rounded px-1 w-4/5"
                            @keyup.enter="($event.target as HTMLInputElement).blur()"
                            @keydown.backspace="column.title == '' 
                                ? (columns = columns.filter((e) => e.id != column.id)) 
                                : null
                            "
                            type="text"
                            v-model="column.title"
                        />
                    </header>
                    <draggable
                        v-model="column.tasks"
                        :group="{ name: 'tasks', pull: alt ? 'clone': true}"
                        handle=".drag-handle"
                        :animation="150"
                        item-key="id"
                    >
                        <template #item="{element: task}: {element: Task}">
                            <div>
                                <TrelloBoardTask :task="task"
                                    @delete="
                                    column.tasks = column.tasks.filter((t) => t.id != $event)
                                    "
                                 />
                            </div>
                        </template>
                    </draggable>
                    <footer>
                        <NewTask @add="column.tasks.push($event)"/>
                    </footer>
                </div>
            </template>
        </draggable>
        <button
            @click="createColumn"
            class=" bg-gray-200 whitespace-nowrap p-2 rounded opacity-50"
        >
            + Add Another Column
        </button>
    </div>
</template>
