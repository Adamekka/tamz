<script lang="ts">
	import { v4 as uuidv4 } from 'uuid';

	let htmlList: HTMLUListElement;
	let ionicList: HTMLElement;
	let itemName = '';

	function clearLists() {
		htmlList.innerHTML = '';
		ionicList.innerHTML = '';
	}

	function showHList(n: number) {
		clearLists();
		console.log('showHList');
		for (let i = 0; i < n; i++) {
			let node = document.createElement('li');
			node.innerText = i.toString();
			htmlList.appendChild(node);
		}
	}

	function showIList(n: number) {
		clearLists();
		console.log('showIList');
		for (let i = 0; i < n; i++) {
			let node = document.createElement('ion-item');
			node.setAttribute('button', 'true');
			node.innerHTML = `<ion-label>${i.toString()}</ion-label>`;
			ionicList.appendChild(node);
		}
	}

	function handleItemNameInput(event: Event) {
		const target = event.currentTarget as HTMLIonInputElement;
		itemName = String(target.value ?? '');
	}

	function addSimpleItem() {
		const name = itemName.trim();
		if (!name) return;

		localStorage.setItem(uuidv4(), name);
		itemName = '';
		console.log('localStorage.length: ' + localStorage.length);
	}

	function clearItems() {
		localStorage.clear();
		showItems();
	}

	function showItems() {
		clearLists();

		for (let i = 0; i < localStorage.length; i++) {
			let key = localStorage.key(i);
			if (!key) continue;

			let value = localStorage.getItem(key);
			console.log(value);

			let node = document.createElement('ion-item');

			let label = document.createElement('ion-label');
			label.innerText = value ?? '';

			let button = document.createElement('ion-button');
			button.setAttribute('slot', 'end');
			button.innerText = 'Delete';
			button.addEventListener('click', () => itemClick(key));

			node.appendChild(label);
			node.appendChild(button);
			ionicList.appendChild(node);
		}
	}

	function itemClick(id: string) {
		console.log('Item ID: ' + id);
		localStorage.removeItem(id);
		showItems();
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
	<ion-row>
		<ion-input placeholder="Item name" value={itemName} on:ionInput={handleItemNameInput}
		></ion-input>
		<ion-button id="button-list-3" on:click={addSimpleItem}>LS add</ion-button>
	</ion-row>

	<ion-button id="button-list-6" on:click={clearLists}>Clear lists</ion-button>
	<ion-button id="button-list-1" on:click={() => showHList(50)}>HTML List</ion-button>
	<ion-button id="button-list-2" on:click={() => showIList(50)}>Ionic List</ion-button>
	<ion-button id="button-list-4" on:click={showItems}>LS read</ion-button>
	<ion-button id="button-list-5" on:click={clearItems}>LS clear</ion-button>

	<ul id="html-list" bind:this={htmlList}></ul>
	<ion-list id="ionic-list" inset={true} bind:this={ionicList}></ion-list>
</ion-content>
