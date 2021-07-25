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
	let amount = 0;
	let receivers = [...names];
	let causale = "";
	onMount(async () => {
		let Airtable = require("airtable");
		name = window.localStorage.getItem("userName") || "Maggi";
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
				window.location.href =
					"https://airtable.com/shrFWC8S2p7zseohr/tblG4EdvD3vtzO175";
			}
		);
		amount = 0;
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
		<label>
			<input
				type="checkbox"
				bind:group={receivers}
				name="receiver"
				value={name}
			/>
			{name}
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
</main>

<style>
	main {
		text-align: center;
		max-width: 400px;
		margin: 0 auto;
	}
	input[type="number"] {
		width: 9ch;
	}
	input[type="date"] {
		display: block;
		margin: auto;
		width: 240px;
	}
	label {
		display: block;
		border-radius: 10px;
		margin: 0.5em auto 0.5em auto;
		box-shadow: #666 0px 0px 3px 1px;
		max-width: 240px;
	}
	.action {
		width: 90%;
		font-size: 1.5em;
		background-color: #cfdfff;
		border-radius: 5px;
	}
	.action[disabled=""] {
		background-color: #eee;
	}
</style>
