<template>
  <div>
    <form @submit.prevent="addColaborador">
      <input v-model="nuevoColaborador.nombre" placeholder="Ingrese Nombre" />
      <input v-model="nuevoColaborador.correo" placeholder="Ingrese Correo" />
      <button type="submit">Agregar</button>
    </form>
    <ul>
      <li v-for="colaborador in colaboradores" :key="colaborador.id">
        {{ colaborador.id }} - {{ colaborador.nombre }} - {{ colaborador.correo }}
        <button @click="deleteColaborador(colaborador.id)">Eliminar</button>
        <button @click="startEditColaborador(colaborador)">Editar</button>
      </li>
    </ul>

    <div v-if="isEditing">
      <h3>Editar Colaborador</h3>
      <form @submit.prevent="editColaborador">
        <input v-model="colaboradorToEdit.nombre" placeholder="Ingrese Nombre" />
        <input v-model="colaboradorToEdit.correo" placeholder="Ingrese Correo" />
        <button type="submit">Guardar Cambios</button>
        <button type="button" @click="cancelEdit">Cancelar</button>
      </form>
    </div>
  </div>
</template>

<script>
import { getFirestore, collection, onSnapshot, addDoc, doc, deleteDoc, updateDoc } from 'firebase/firestore';
import App from '../config/FirebaseConfig';

export default {
  data() {
    return {
      nuevoColaborador: { nombre: '', correo: '' },
      colaboradores: [],
      isEditing: false,
      colaboradorToEdit: null
    };
  },
  mounted() {
    const db = getFirestore(App);
    const colaboradoresRef = collection(db, 'colaboradores');
    onSnapshot(colaboradoresRef, (snapshot) => {
      this.colaboradores = snapshot.docs.map((doc) => ({ id: doc.id, ...doc.data() }));
    });
  },
  methods: {
    async addColaborador() {
      if (this.nuevoColaborador.nombre.trim() === '' || this.nuevoColaborador.correo.trim() === '') return;
      const db = getFirestore(App);
      const colaboradoresRef = collection(db, 'colaboradores');
      await addDoc(colaboradoresRef, { nombre: this.nuevoColaborador.nombre, correo: this.nuevoColaborador.correo });
      this.nuevoColaborador = { nombre: '', correo: '' }; // Limpiar campos después de agregar
    },
    async deleteColaborador(colaboradorId) {
      const db = getFirestore(App);
      const colaboradorRef = doc(db, 'colaboradores', colaboradorId);
      await deleteDoc(colaboradorRef);
    },
    startEditColaborador(colaborador) {
      this.colaboradorToEdit = { ...colaborador };
      this.isEditing = true;
    },
    async editColaborador() {
      const db = getFirestore(App);
      const colaboradorRef = doc(db, 'colaboradores', this.colaboradorToEdit.id);
      await updateDoc(colaboradorRef, {
        nombre: this.colaboradorToEdit.nombre,
        correo: this.colaboradorToEdit.correo
      });
      this.cancelEdit();
    },
    cancelEdit() {
      this.isEditing = false;
      this.colaboradorToEdit = null;
    }
  }
};
</script>

<style scoped>
img {
  height: 150px;
  width: 200px;
  object-fit: contain;
}
</style>