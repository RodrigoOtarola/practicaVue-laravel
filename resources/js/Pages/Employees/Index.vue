<script setup>
import AuthenticatedLayout from '@/Layouts/AuthenticatedLayout.vue';
import InputError from '@/Components/InputError.vue';
import InputLabel from '@/Components/InputLabel.vue';
import PrimaryButton from '@/Components/PrimaryButton.vue';
import TextInput from '@/Components/TextInput.vue';
import DangerButton from '@/Components/DangerButton.vue';
import SelectInput from "@/Components/SelectInput.vue";
import WarningButton from "@/Components/WarningButton.vue";
import SecondaryButton from "@/Components/SecondaryButton.vue";
import Modal from "@/Components/Modal.vue";

import {Head, useForm} from '@inertiajs/vue3';
import {nextTick,ref} from 'vue';
import Swal from "sweetalert2";
import VueTailwindPaginationEs from "@ocrv/vue-tailwind-pagination";

/** DEFINICION DE CONSTANTES*/
const nameInput=ref(null);
const modal = ref(false);
const title = ref('');
const operation = ref(1);
const id = ref('');

const props = defineProps({
    employees: {type:Object},
    departments: {type: Object}
});

//Parametros del formulario
const form = useForm({
    name:'',email:'',phone:'',department_id:''
});

/** PAGE PARA LA PAGINACIÓN*/
const formPage = useForm({});
const onPageClick=(event)=>{
    formPage.get(route('employees.index',{page:event}))
}

/** VENTANA MODAL */
const openModal = (op,name,email,phone,department,employee)=>{
    modal.value = true;
    nextTick(() => nameInput.value.focus());
    operation.value = op;
    id.value = employee;
    if(op == 1){
        title.value = 'Create employee';
    }else {
        title.value = 'Edit employee';
        form.name = name;
        form.email = email;
        form.phone = phone;
        form.department_id = department;
    }
}
const closeModal = ()=>{
    modal.value = false;
    form.reset();
}

/** FUNCION GUARDAR*/
const save = ()=>{
    if(operation.value == 1){
        form.post(route('employees.store'),{
            onSuccess:()=>{ok('Employee created')}
        });
    }else{
        form.put(route('employees.update',id.value),{
            onSuccess:()=>{ok('Employee updated')}
        });
    }
}

const ok = (msj)=>{
    form.reset();
    closeModal();
    Swal.fire({
        title: msj,icon:'success'
    });
}

//Recibimos el parametro id y name
const deleteEmployee = (id,name) => {
    //Funcion de alerta con SweetAlert
    const alerta = Swal.mixin({
        buttonsStyling: true
    });
    alerta.fire({
        title: 'Estas de seguro de eliminar ' + name + ' ?',
        icon: 'question',
        showCancelButton: true,
        confirmButtonText: '<i class="fa-solid fa-check"></i> Si, eliminar',
        cancelButtonText: '<i class="fa-solid fa-ban"></i> Cancelar'
    }).then((result) => {
        if (result.isConfirmed) {
            form.delete(route('employees.destroy', id),{
                onSuccess:()=>{ok('Employee deleted')}
            });
        }
    })
}

</script>

<template>
    <Head title="Employees"/>

    <AuthenticatedLayout>
        <template #header>
            <h2 class="font-semibold text-xl text-gray-800 leading-tight">Employees</h2>
        </template>

        <div class="py-12">
            <div class="bg-white grid v-screen place-items-center">
                <div class="mt-3 mb-3 flex">
                    <!--Le pasa como parametro 1, para que abra el formulario vacio-->
                    <PrimaryButton @click="$event=>openModal(1)">
                        <i class="fa-solid fa-plus-circle"></i> Add
                    </PrimaryButton>
                </div>
            </div>
            <div class="bg-white grid v-screen place-items-center overflow-x-auto">
                <div class="mt-3 mb-3 flex">
                    <table class="table-auto border border-gray-400">
                        <thead>
                        <tr class="bg-gray-100">
                            <th class="px-2 py-2">#</th>
                            <th class="px-2 py-2">Name</th>
                            <th class="px-2 py-2">Email</th>
                            <th class="px-2 py-2">Phone</th>
                            <th class="px-2 py-2">Department</th>
                            <th class="px-2 py-2"></th>
                            <th class="px-2 py-2"></th>
                        </tr>
                        </thead>
                        <tbody>
                        <tr v-for="emp, i in employees.data" :key="emp.id">
                            <td class="border border-gray-400 px-2 py-2">{{ (i + 1) }}</td>
                            <td class="border border-gray-400 px-2 py-2">{{ emp.name }}</td>
                            <td class="border border-gray-400 px-2 py-2">{{ emp.email }}</td>
                            <td class="border border-gray-400 px-2 py-2">{{ emp.phone }}</td>
                            <td class="border border-gray-400 px-2 py-2">{{ emp.department }}</td>
                            <td class="border border-gray-400 px-2 py-2">
                               <WarningButton
                                   @click="$event=>openModal(2,emp.name,emp.email,emp.phone,emp.department_id,emp.id)">
                                   <i class="fa-solid fa-edit"></i>
                               </WarningButton>
                            </td>
                            <td class="border border-gray-400 px-2 py-2">
                                <DangerButton @click="$event=>deleteEmployee(emp.id,emp.name)">
                                    <i class="fa-solid fa-trash"></i>
                                </DangerButton>
                            </td>
                        </tr>
                        </tbody>
                    </table>
                </div>
            </div>
            <!--PAGINACION-->
            <div class="bg-white grid v-screen place-items-center">
                <VueTailwindPaginationEs
                    :current="employees.currentPage"
                    :total="employees.total"
                    :per-page="employees.perPage"
                    @page-changed="$event => onPageClick($event)">
                </VueTailwindPaginationEs>
            </div>
            <!--MODAL-->
            <Modal :show="modal" @close="closeModal">
                <h2 class="p-3 text-lg font-medium text-gray-900">{{ title }}</h2>
                <div class="p-3">
                    <InputLabel for="name" value="Name:"></InputLabel>
                    <TextInput type="text" id="name" ref="nameInput" v-model="form.name" class="mt-1 block w-3/4" placeholder="Name"></TextInput>
                    <InputError :message="form.errors.name"></InputError>
                </div>
                <div class="p-3">
                    <InputLabel for="email" value="email:"></InputLabel>
                    <TextInput type="text" id="email" v-model="form.email" class="mt-1 block w-3/4" placeholder="email"></TextInput>
                    <InputError :message="form.errors.email"></InputError>
                </div>
                <div class="p-3">
                    <InputLabel for="phone" value="phone:"></InputLabel>
                    <TextInput type="text" id="phone" v-model="form.phone" class="mt-1 block w-3/4" placeholder="phone"></TextInput>
                    <InputError :message="form.errors.phone"></InputError>
                </div>
                <div class="p-3">
                    <InputLabel for="department_id" value="Department:"></InputLabel>
                    <!--Trae options desde el componentes-->
                    <SelectInput id="department_id" :options="departments" v-model="form.department_id"
                                 type="text" class="mt-1 block w-3/4"></SelectInput>
                    <InputError :message="form.errors.department_id"></InputError>
                </div>
                <div class="p-3 mt-6">
                    <PrimaryButton :disabled="form.processing" @click="save">
                        <i class="fa-solid fa-save"></i> Save
                    </PrimaryButton>
                </div>
                <div class="p-3 mt-6 flex justify-end">
                    <SecondaryButton class="ml-3" :disabled="form.processing" @click="closeModal">
                        Cancel
                    </SecondaryButton>
                </div>
            </Modal>
        </div>
    </AuthenticatedLayout>
</template>
