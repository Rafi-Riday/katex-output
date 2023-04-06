<script>
	import { tick } from "svelte";
	import Katex from "./Katex.svelte";

	let currentExpr = `\\frac{a^2}{b}`;
	let fieldStr;
	let resetBtn;

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

	// const keyUpFeatures = async (e) => {
	// 	if (e.key === "{") {
	// 		const { selectionStart, selectionEnd, value } = fieldStr;
	// 		if (selectionStart === selectionEnd) {
	// 			currentExpr =
	// 				value.slice(0, selectionStart) +
	// 				"}" +
	// 				value.slice(selectionStart, value.length);
	// 		}
	// 		await tick();
	// 		fieldStr.selectionStart = selectionStart;
	// 		fieldStr.selectionEnd = selectionEnd;
	// 		fieldStr.focus();
	// 	}
	// };

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
	{#if currentExpr !== ""}
		<div class="output">
			<Katex
				expression={currentExpr.replace(/\\\\/g, "\\")}
				displayMode
			/>
			<!-- .replace(/\\/g, "\\\\") -->
		</div>
	{/if}
	<div class="inp-cont">
		<textarea bind:this={fieldStr} bind:value={currentExpr} />
		<!-- on:keyup={(e) => keyUpFeatures(e)} -->
	</div>
	<div class="actions">
		<button
			style="border: 0; background: #74efff; border-radius: 10px; color: #000; padding:10px 20px; font-size: 18px"
			on:click={() => copy2Clip()}>Copy</button
		>
		<button
			style="border: 0; background: #74efff; border-radius: 10px; color: #000; padding: 7.5px 20px; font-size: 18px"
			on:click={() => copy2ClipForCode()}
			>Copy : <code>\</code>
			<Katex expression={"\\rightarrow"} />
			<code>\\</code></button
		>
		<button
			bind:this={resetBtn}
			style="border: 0; background: #ffc6b5; border-radius: 10px; color: #000; padding:10px 20px; font-size: 18px"
			on:click={() => (currentExpr = "")}>Reset</button
		>
	</div>
	<aside>
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
		/* font-size: 30px; */
		background-color: #fff;
		border-radius: 10px;
		padding: 5px;
		display: inline;
	}
	textarea {
		background-color: #fff;
		width: calc(100% - 16px);
		/* width: 100%; */
		height: 120px;
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
	aside {
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
