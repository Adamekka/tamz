<script lang="ts">
	import { onMount } from 'svelte';

	let htmlList: HTMLUListElement;
	let ionicList: HTMLElement;

	onMount(() => {
		// showItems();
	});

	function showHList(n: number) {
		console.log('showHList');
		for (let i = 0; i < n; i++) {
			let node = document.createElement('li');
			node.innerText = i.toString();
			htmlList.appendChild(node);
		}
	}

	function showIList(n: number) {
		console.log('showIList');
		for (let i = 0; i < n; i++) {
			let node = document.createElement('ion-item');
			node.setAttribute('button', 'true');
			node.innerHTML = `<ion-label>${i.toString()}</ion-label>`;
			ionicList.appendChild(node);
		}
	}

	function addSimpleItem() {
		localStorage.setItem('key-1', 'AAA BBB CCC DDD');
		localStorage.setItem('key-2', 'BBB CCC DDD EEE');
		localStorage.setItem('key-3', 'CCC DDD FFF GGG');
		console.log('localStorage.length: ' + localStorage.length);
	}

	function showItems() {
		let el = htmlList;
		el.innerHTML = '';

		let value = localStorage.getItem('key-1');
		console.log(value);
		let node = document.createElement('li');
		node.innerText = value ?? '';
		htmlList.appendChild(node);
	}

	function itemClick(id: string) {
		console.log('Item ID: ' + id);
	}
</script>

<!-- =============================================================
	TODO 0: Zobrazení záznamů v ionic-list
	=============================================================
	Upravte funkci showItems() tak, aby se zobrazovaly správné
	názvy záznamu v ion-list (místo HTML list).
-->

<!-- =============================================================
	TODO 1: Smazání všech záznamů
	=============================================================
	Doplňte funkci, která odstraní všechny záznamy z úložiště
	a aktualizuje zobrazený seznam.
-->

<!-- =============================================================
	TODO 2: Přidání záznamu s vlastním názvem
	=============================================================
	Upravte přidávání tak, aby se jako název záznamu použil
	text zadaný uživatelem (ion-input).
	Co použít jako vhodný klíč?
-->

<!-- =============================================================
	TODO 3: Odstranění jednotlivého záznamu
	=============================================================
	Přidejte možnost odstranit konkrétní záznam ze seznamu
	(např. tlačítkem u každé položky).
-->

<!-- =============================================================
	Za splnění základních TODO 0-3 získáte 2 body.
	Pokud vypracujete i některá z následujících rozšíření (bonus),
	můžete získat další 1 bod navíc.
	=============================================================
	- Vytvořte objekt reprezentující záznam a do úložiště ukládejte
	  a načítejte tento objekt pomocí JSON.stringify / JSON.parse.
	  Např.: { name: "Nákup", priority: "high" }

	- Přidejte barvu/ikonu podle priority (ion-select + ion-icon)

	- Použijte ion-item-sliding pro swipe-to-delete
	  https://ionicframework.com/docs/api/item-sliding

	- Přidejte ion-alert pro potvrzení smazání
	  https://ionicframework.com/docs/api/alert

	- Přidejte ion-badge s počtem položek do headeru
	  https://ionicframework.com/docs/api/badge
-->

<ion-header>
	<ion-toolbar>
		<ion-title>Local Storage</ion-title>
	</ion-toolbar>
</ion-header>

<ion-content class="ion-padding">
	<ion-button id="button-list-1" on:click={() => showHList(50)}>HTML List</ion-button>
	<ion-button id="button-list-2" on:click={() => showIList(50)}>Ionic List</ion-button>
	<ion-button id="button-list-3" on:click={addSimpleItem}>LS add</ion-button>
	<ion-button id="button-list-4" on:click={showItems}>LS read</ion-button>

	<ul id="html-list" bind:this={htmlList}></ul>
	<ion-list id="ionic-list" inset={true} bind:this={ionicList}></ion-list>
</ion-content>
