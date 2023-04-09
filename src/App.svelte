<script>
	import { tick } from "svelte";
	import Katex from "./Katex.svelte";

	let currentExpr = "";
	let fieldStr;
	let resetBtn;
	let loaded = false;
	setTimeout(() => {
		loaded = true;
	}, 1000);

	const copy2Clip = async () => {
		currentExpr = currentExpr.replaceAll("\\\\", "\\");
		await tick();
		fieldStr.select();
		document.execCommand("copy");
		resetBtn.focus();
	};

	const copy2ClipForCode = async () => {
		currentExpr = currentExpr.replaceAll("\\\\", "\\");
		currentExpr = currentExpr.replaceAll("\\", "\\\\");
		await tick();
		fieldStr.select();
		document.execCommand("copy");
		currentExpr = currentExpr.replaceAll("\\\\", "\\");
		resetBtn.focus();
	};

	const featureKey = async (key, cursor = 1) => {
		const { selectionStart, selectionEnd, value } = fieldStr;
		if (selectionStart === selectionEnd) {
			currentExpr =
				value.slice(0, selectionStart) +
				key +
				value.slice(selectionStart, value.length);
		}
		await tick();
		fieldStr.selectionStart = selectionStart + cursor;
		fieldStr.selectionEnd = selectionStart + cursor;
		fieldStr.focus();
	};

	let necessaryBtn = ["{", "}", "^", "\\", "="];
	let preExpression = [
		{ key: "{}", cursor: 1 },
		{ key: "\\times ", cursor: 7 },
		{ key: "\\div ", cursor: 5 },
		{ key: "\\frac{}{}", cursor: 6 },
		{ key: "\\sqrt{}", cursor: 6 },
	];
	console.log(preExpression[0].key);
	let demo = [
		"a^2",
		"a^{xyz}",
		"\\times\\div",
		"\\frac{a}{b}",
		"\\sqrt{ab}",
		"2x^2+11x+12",
		"\\frac{-b\\pm\\sqrt{b^2-4ac}}{2a}",
		"\\frac{b}{2}\\sqrt{a^2-\\frac{b^2}{4}}",
	];
</script>

<main>
	{#if currentExpr.trim() !== ""}
		<center class="output">
			<Katex expression={currentExpr} />
			<!-- displayMode -->
			<!-- .replace(/\\/g, "\\\\") -->
		</center>
	{/if}
	<div class="inp-cont">
		<textarea
			placeholder="Write your expression here..."
			bind:this={fieldStr}
			bind:value={currentExpr}
		/>
	</div>
	<aside class="feature-key">
		{#each necessaryBtn as nb}
			<button
				class="feature-key-btn btn"
				style="padding: 10px 20px;"
				on:click={() => {
					featureKey(nb);
				}}
			>
				{nb}
			</button>
		{/each}
	</aside>
	<aside class="feature-key">
		{#each preExpression as expData}
			<button
				class="feature-key-btn btn"
				style="padding: 10px 10px;"
				on:click={() => {
					featureKey(expData.key, expData.cursor);
				}}
			>
				{expData.key}
			</button>
		{/each}
	</aside>
	<div class="actions">
		<button
			class="btn"
			style="background: #74efff;"
			on:click={() => copy2Clip()}>Copy</button
		>
		<button
			style="border: 0; background: #74efff; border-radius: 10px; color: #000; padding: 7.5px 16px; font-size: 18px; display: flex; align-items: center; justify-content: space-between;"
			on:click={() => copy2ClipForCode()}
			>Copy :&nbsp;<code>\</code>
			<div style="transform: translate(0, -2px);">→</div>
			<code>\\</code></button
		>
		<button
			bind:this={resetBtn}
			class="btn"
			style="background: #ffc6b5;"
			on:click={() => {
				currentExpr = "";
				fieldStr.focus();
			}
			}>Reset</button
		>
	</div>
	{#if loaded}
		<aside class="demo">
			<b style="margin: 5px 0;font-size: 20px;">Demo</b>
			{#each demo as d}
				<div>
					<button class="code" on:click={() => (currentExpr = d)}
						>{d}</button
					>
					=
					<Katex expression={d} />
				</div>
			{/each}
			<p style="display: flex; flex-direction: column; gap: 12px">
				<a
					href="https://katex.org/docs/supported.html"
					target="_blank"
					rel="noreferrer"
				>
					<b>More Demo...</b>
				</a>
				<a
					href="https://detexify.kirelabs.org/classify.html"
					target="_blank"
					rel="noreferrer"
				>
					<b>Detect a Symbol by Drawing...</b>
				</a>
			</p>
		</aside>
	{/if}
</main>

<style>
	:global(html, body) {
		height: 100%;
		width: 100%;
		color: #000;
		background-color: #323232;
		margin: 0;
		padding: 0;
	}
	main {
		padding: 10px;
		display: flex;
		flex-direction: column;
		align-items: stretch;
		justify-content: center;
		flex-wrap: wrap;
		gap: 10px;
	}
	.output {
		font-size: 18px;
		background-color: #fff;
		border-radius: 10px;
		padding: 5px;
		display: inline;
	}
	.inp-cont {
		border-bottom: 2px solid #fffb;
	}
	textarea {
		background-color: #fff;
		width: calc(100% - 16px);
		/* width: 100%; */
		height: 75px;
		/* border: 2px dashed #444; */
		border: 2px solid #ff3300;
		/* margin: 0; */
		border-radius: 10px;
		padding: 5px;
		outline: 0;
	}
	.actions {
		display: flex;
		flex-direction: row;
		justify-content: center;
		align-items: center;
		gap: 10px;
	}
	.btn {
		border: 0;
		border-radius: 10px;
		color: #000;
		padding: 10px 16px;
		font-size: 18px;
	}
	.feature-key {
		/* background-color: #fff; */
		display: flex;
		flex-direction: row;
		flex-wrap: wrap;
		justify-content: center;
		align-items: center;
		gap: 10px;
		border-radius: 10px;
	}
	.feature-key-btn {
		font-size: 18px;
		padding: 3px 5px;
	}
	.demo {
		background-color: #fff;
		padding: 10px;
		border-radius: 10px;
		display: flex;
		flex-direction: column;
		gap: 5px;
	}
	.code {
		display: inline-block;
		color: #ff3300;
		font-weight: 600;
		background-color: #ff3e0010;
		border: 2px solid #ff3e0020;
		padding: 2px 5px;
		border-radius: 10px;
	}
	a {
		color: #0058aa;
	}
	code {
		background: #fffb;
		border-radius: 5px;
		border: 1.5px solid #ff3e00aa;
		letter-spacing: -7px;
		padding-right: 7px;
	}
</style>
