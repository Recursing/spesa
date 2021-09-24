<script lang="ts">
	import { onMount } from "svelte";
	import type { Base } from "airtable";
	const ids = {
		Hodor: "recWk9rTbd27JLtMX",
		Irene: "recSKUJw4gv9jkSY6",
		Isabel: "recnN8gbf45pTDpo1",
		Lucia: "recPLxhbvlPUXhfUx",
		Maggi: "recqaYv6OYBOPBn5X",
		Mannaggia: "recjBxi0d4Dv35wbp",
		Max: "recJpxbpgahtKbDHt",
		Minala: "recyFNg4GowXtq29s",
		Nevischio: "recUXTfN5cpNaIgsp",
		Pasti: "recYekZGdpfRVEt5a",
		Serena: "recXrNUH2cOoDZ8om",
		Serse: "recz2sUGCyC4LarTm",
	};
	const names: string[] = Object.keys(ids).sort();
	let name = null;
	let base: Base | null = null;
	let date = new Date(Date.now() - new Date().getTimezoneOffset() * 60000)
		.toISOString()
		.split("T")[0];
	let amount: number | null = null;
	let receivers: string[] = [];
	let causale = "";
	onMount(async () => {
		let Airtable = require("airtable");
		name = window.localStorage.getItem("userName") || "Maggi";
		receivers = JSON.parse(
			window.localStorage.getItem("receiversNames")
		) || [...names];
		base = new Airtable({ apiKey: "SUPER_SECRET" }).base(
			"appp30Zudw46QZbWI"
		);
		base("Calci")
			.select({
				view: "Grid view",
			})
			.firstPage(function (err, records) {
				if (err) {
					console.error(err);
					return;
				}
				if (
					!records.every(
						(r) => ids[r.get("Name") as string] == r.getId()
					)
				) {
					alert("Calci ids changed??");
				}
				if (records.length != names.length) {
					alert("Calci number changed??");
				}
			});
	});
	$: name && (() => window.localStorage.setItem("userName", name))();
	$: receivers.length &&
		(() =>
			window.localStorage.setItem(
				"receiversNames",
				JSON.stringify(receivers)
			))();
	function handleClick(event: MouseEvent) {
		const creditoreId = ids[name];
		const debitoriIds: string[] = receivers.map((n) => ids[n]);
		base("Spese").create(
			[
				{
					fields: {
						Creditore: [creditoreId],
						Debitore: debitoriIds,
						"€": amount,
						Data: date,
						Causa: causale,
					},
				},
			],
			(err, records) => {
				if (err) {
					alert(err);
					console.error(err);
					return;
				}
			}
		);
		amount = null;
	}
</script>

<main>
	<select name="name" id="name" bind:value={name}>
		{#each names as name}
			<option value={name}>{name}</option>
		{/each}
	</select>
	<input type="number" min="0" max="999" bind:value={amount} />€
	<input type="date" bind:value={date} />
	{#each names as name}
		<label class:text-selected={receivers.includes(name)}>
			<input
				type="checkbox"
				bind:group={receivers}
				name="receiver"
				value={name}
			/>
			<div class="flex-auto">
				{name}
			</div>
		</label>
	{/each}
	<input type="text" bind:value={causale} placeholder="Causale" />
	<input
		class="action"
		disabled={!amount}
		type="button"
		value="Salva"
		on:click={handleClick}
	/>
	<a href="https://airtable.com/shrFWC8S2p7zseohr">Elenco crediti</a>
	<a href="https://airtable.com/shr0VsZrKINXerDxR">Elenco spese</a>
</main>

<style>
	a {
		padding: 1em;
	}
	main {
		text-align: center;
		max-width: 400px;
		margin: 0 auto;
	}
	input[type="number"] {
		width: 9ch;
		text-align: right;
	}
	input[type="date"] {
		display: block;
		margin: auto;
		width: 100%;
		max-width: 300px;
	}
	input[type="checkbox"] {
		margin: 0;
		transform: scale(1.5);
	}
	label,
	input[type="text"] {
		display: flex;
		align-items: center;
		margin: 0 auto 0 auto;
		border-left: 1px solid #ccc;
		border-right: solid 1px #ccc;
		border-bottom: solid 1px #ccc;
		border-top: none;
		width: 100%;
		max-width: 300px;
		box-sizing: border-box;
		padding: min(0.5em, calc(4vh - 18px));
		padding-left: 2em;
		padding-right: 2em;
		margin: auto;
		transition: 0.1s;
	}
	input[type="text"] {
		padding: 0.4em;
	}
	.action {
		display: block;
		margin: auto;
		width: 100%;
		max-width: 300px;
		font-size: 1.5em;
		background-color: #cfdfff;
		border-radius: 5px;
	}
	.action[disabled=""] {
		background-color: #eee;
	}
	.flex-auto {
		flex: auto;
		transition: 0.5s;
	}
	.text-selected {
		background-color: #cfdfffa0;
	}
</style>
