<script setup>
import { ref, onMounted, computed } from "vue";

const showModal = ref(false);
const newNote = ref("");
const newPrice = ref(0);
const notes = ref([]);
const errorMessage = ref("");
const editModal = ref("");

onMounted(async () => {
	fetchNotes();
});

const fetchNotes = async () => {
	try {
		const response = await fetch(
			"https://gastossushi-default-rtdb.firebaseio.com/sushi.json"
		);
		const data = await response.json();
		if (response.ok) {
			const fetchedItems = [];
			for (const key in data) {
				fetchedItems.push({ id: key, ...data[key] });
			}
			notes.value = fetchedItems;
		} else {
			console.error("Error fetching items:", response.statusText);
		}
	} catch (error) {
		console.error("Error fetching data:", error);
	}
};

function randomColor() {
	return "hsl(" + Math.random() * 360 + ", 100%, 75%)";
}

const addNote = async () => {
	if (newNote.value.length <= 1) {
		return (errorMessage.value = "Ingresa una descripcion");
	}
	if (newPrice.value === 0) {
		return (errorMessage.value = "Ingrese un monto");
	}

	const newFullNote = {
		text: newNote.value,
		price: newPrice.value,
		date: new Date(),
		backgroundColor: randomColor(),
	};

	notes.value.push(newFullNote);

	try {
		if (editModal.value === "") {
			const response = await fetch(
				"https://gastossushi-default-rtdb.firebaseio.com/sushi.json",
				{
					method: "POST",
					headers: {
						"Content-Type": "application/json",
						"Access-Control-Allow-Origin": "*",
					},
					body: JSON.stringify(newFullNote),
				}
			);
			const data = await response.json();
			console.log("Data posted successfully:", data);
		} else {
			const response = await fetch(
				`https://gastossushi-default-rtdb.firebaseio.com/sushi/${editModal.id}.json`,
				{
					method: "PUT",
					headers: {
						"Content-Type": "application/json",
					},
					body: JSON.stringify(newFullNote),
				}
			);
			const data = await response.json();
			fetchNotes();

			if (index !== -1) {
				notes.value.splice(index, 1, newFullNote);
			}
			console.log("Data posted successfully:", newFullNote);
		}
	} catch (error) {
		console.error("Error posting data:", error);
	}

	showModal.value = false;
	newNote.value = "";
	errorMessage.value = "";
	editModal.value = "";
};

const totalPrice = computed(() => {
	return notes.value.reduce((total, item) => total + item.price, 0);
});

const totalLost = computed(() => {
	return notes.value
		.filter((item) => item.price < 0)
		.reduce((total, item) => total + item.price, 0);
});
const totalWin = computed(() => {
	return notes.value
		.filter((item) => item.price > 0)
		.reduce((total, item) => total + item.price, 0);
});

const handleDelete = async (item) => {
	try {
		const response = await fetch(
			`https://gastossushi-default-rtdb.firebaseio.com/sushi/${item.id}.json`,
			{
				method: "DELETE",
			}
		);

		if (response.ok) {
			notes.value = notes.value.filter((note) => note.id !== item.id);
			console.log("Nota eliminada", item);
		} else {
			console.error("Error deleting item:", response.statusText);
		}
	} catch (error) {
		console.error("Error deleting item:", error);
	}
};

const handleUpdate = (item) => {
	console.log(item.id);
	newPrice.value = item.price;
	newNote.value = item.text;
	showModal.value = true;
	editModal.value = item.id;
};
</script>

<template>
	<main>
		<div v-if="showModal" class="overlay">
			<div class="modal">
				<h3>Gastos Sushi</h3>
				<textarea
					v-model.trim="newNote"
					name="note"
					id="note"
					cols="30"
					rows="5"
				></textarea>
				<input type="number" v-model="newPrice" name="price" id="price" />
				<p v-if="errorMessage">{{ errorMessage }}</p>
				<button @click="addNote">Crear item</button>
				<button @click="showModal = false" class="close">Close</button>
			</div>
		</div>
		<div class="container">
			<header>
				<h1>Compras</h1>
				<button @click="showModal = true">+</button>
			</header>
			<div class="total">
				<p :style="{ color: 'blue' }">Ingresos: {{ totalWin }}</p>
				<p :style="{ color: 'red' }">Egresos: {{ totalLost }}</p>
				<p>Total: {{ totalPrice }}</p>
			</div>
			<div class="cards-container">
				<div
					:key="note.id"
					class="card"
					v-for="note in notes"
					:style="{ backgroundColor: note.backgroundColor }"
				>
					<p class="main-text">
						{{ note.text }}
					</p>
					<p class="main-price" :style="{ color: note.price < 0 ? 'red' : 'blue' }">
						Gastos: {{ note.price }}
					</p>
					<p class="date">{{ note.date }}</p>
					<button
						:style="{ color: 'red', border: 'none', borderRadius: '20%' }"
						@click="handleDelete(note)"
					>
						Eliminar
					</button>
					<button
						:style="{ color: 'green', border: 'none', borderRadius: '20%' }"
						@click="handleUpdate(note)"
					>
						Editar
					</button>
				</div>
			</div>
		</div>
	</main>
</template>

<style scoped>
main {
	height: 100vh;
	width: 100vw;
}
.container {
	max-width: 1000px;
	padding: 10px;
	margin: 0 auto;
}

header {
	display: flex;
	justify-content: space-between;
	align-items: center;
}

h1 {
	font-weight: bold;
	margin-bottom: 25px;
	font-size: 75px;
}

header button {
	border: none;
	padding: 10px;
	width: 50px;
	height: 50px;
	cursor: pointer;
	background-color: rgb(21, 20, 20);
	border-radius: 100%;
	color: white;
	font-size: 20px;
}

.cards-container {
	display: flex;
	flex-wrap: wrap;
}

.card {
	width: 225px;
	height: 225px;
	background-color: rgb(200, 182, 44);
	padding: 10px;
	border-radius: 15px;
	display: flex;
	flex-direction: column;
	justify-content: space-between;
	margin-right: 20px;
	margin-bottom: 20px;
}

.date {
	font-size: 12.5px;
	font-weight: bold;
}

.overlay {
	position: absolute;
	width: 100%;
	height: 100%;
	background-color: rgba(0, 0, 0, 0.77);
	z-index: 10;
	display: flex;
	align-items: center;
	justify-content: center;
}

.modal {
	width: 750px;
	background-color: white;
	border-radius: 10px;
	padding: 30px;
	position: relative;
	display: flex;
	flex-direction: column;
}

.modal button {
	padding: 10px 20px;
	font-size: 20px;
	width: 100%;
	background-color: blueviolet;
	border: none;
	color: white;
	cursor: pointer;
	margin-top: 50px;
}

.modal .close {
	background-color: red;
	margin-top: 7px;
}

.modal p {
	color: red;
}
</style>
