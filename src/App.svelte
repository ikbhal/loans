<script>
import {onMount} from 'svelte';
import {writable} from 'svelte/store';
import {db} from './firebase';
let todayDateString = new Date().toISOString().slice(0,10);
export let date= todayDateString;
let amount = 0;
let category = "";
let categoryList = ["Familiy & Friends", "Gold Loan", "Credit Card", "Other"];
let notes = "";
export let loanList=writable([]);
async function loadLoanList(){
	let loansRef = db.collection('loans');
  	let allLoans= await loansRef.get();
	let loanListCopy = [];
	for(const doc of allLoans.docs){
		let docData = doc.data();
		let loanCopy = {
			id:doc.id, 
			date:docData.date, 
			amount: docData.amount,
			category: docData.category,
			notes: docData.notes
		};
		loanListCopy.push(loanCopy);
   		console.log(doc.id, '=>', doc.data());
  	}
	loanList.set(loanListCopy);
}
onMount(async () => {
	loadLoanList();
});
function handleOnSubmit() {
	console.log("form submitted");
	db.collection("loans").add({
		date,
		amount,
		category,
		notes	
	})
	.then(() => {
		console.log("Document successfully written!");
		loadLoanList();
	})
	.catch((error) => {
		console.error("Error writing document: ", error);
	});
}
function deleteLoan(id){
	console.log("remove loan id:", id);
	db.collection("loans").doc(id).delete().then(() => {
		console.log("Document successfully deleted!");
		loadLoanList();
	}).catch((error) => {
		console.error("Error removing document: ", error);
	});
}
</script>

<main>
	<!-- <EnvTest></EnvTest> -->
<h1>Loan</h1>
<p>People given loan, gold loan, credit card loan crud, search</p>

<form on:submit|preventDefault={handleOnSubmit}>
	
	<h3>Add Loan</h3>
	<label>Date:<input type="date" bind:value={date}/></label> <br/>
	<label>Amount:<input type="number" bind:value={amount}/></label> <br/>
	<label>category
		<select bind:value={category}>
			{#each categoryList as category}
			<option value={category}>{category}</option>
			{/each}
		</select>
	</label>
	<div>
		Selected category: {category}
	</div>
	<br/>
	<label>
		Notes:
		<input type="text" bind:value={notes}/>
	</label>
	<br/>
	<button type="submit">Save</button>
</form>

<hr/>
<h3>List Loans</h3>
<div class="loanList">
	{#each $loanList as loan}
		<div>
			{loan.date} <br/>
			Amount: {loan.amount}  
			Category {loan.category}
			Notes {loan.notes}
			<button on:click={deleteLoan(loan.id)}>Delete</button>
		</div>
	{/each}
</div>
</main>

<style>
</style>