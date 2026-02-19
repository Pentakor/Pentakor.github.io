<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
<meta name="theme-color" content="#020409" />
<title>Production Gate</title>

<style>
:root {
	--bg: #020409;
	--text: #a7f3d0;
	--accent: #22d3ee;
}

* {
	box-sizing: border-box;
}

html, body {
	margin: 0;
	padding: 0;
	width: 100%;
	height: 100%;
	background: #020409;
	overflow-x: hidden;
}

body {
	min-height: 100svh;
	display: flex;
	justify-content: center;
	align-items: flex-start;
	padding:
		max(4vh, env(safe-area-inset-top))
		max(4vw, env(safe-area-inset-right))
		max(6vh, env(safe-area-inset-bottom))
		max(4vw, env(safe-area-inset-left));
	background:
		radial-gradient(1200px 600px at 50% -10%, #0b1f1a 0%, transparent 60%),
		linear-gradient(180deg, #020409, #010308 70%);
	color: var(--text);
	font-family: "Courier New", Courier, monospace;
}

.terminal-body {
	width: min(92vw, 720px);
	padding: 2rem;
}

.log-line {
	margin-bottom: 0.4rem;
}

button {
	padding: 0.8rem 1.2rem;
	border-radius: 8px;
	border: 1px solid rgba(34,211,238,0.5);
	background: rgba(7,32,38,0.9);
	color: var(--text);
	cursor: pointer;
}

input {
	padding: 0.8rem;
	border-radius: 8px;
	border: 1px solid rgba(34,211,238,0.4);
	background: rgba(2,10,12,0.9);
	color: var(--text);
	width: 100%;
}

.form {
	margin-top: 1rem;
	opacity: 0;
	pointer-events: none;
	transition: opacity 0.4s ease;
}

.form.visible {
	opacity: 1;
	pointer-events: auto;
}

.hint-button {
	margin-top: 1rem;
	visibility: hidden;
}

.hint-button.visible {
	visibility: visible;
}
</style>
</head>

<body>
<main class="terminal-body">

	<div id="systemLog"></div>

	<button id="showHintButton" class="hint-button">Show hint</button>

	<h1 id="titleText"></h1>
	<p id="hintText"></p>

	<form id="gateForm" class="form">
		<label>Passcode</label>
		<input id="passcode" maxlength="10" placeholder="DD/MM/YYYY" />
		<button type="submit">Unlock</button>
	</form>

	<div id="message"></div>

</main>

<script>
const form = document.getElementById("gateForm");
const passcodeInput = document.getElementById("passcode");
const message = document.getElementById("message");
const systemLog = document.getElementById("systemLog");
const showHintButton = document.getElementById("showHintButton");
const titleText = document.getElementById("titleText");
const hintText = document.getElementById("hintText");

const requiredCode = "20/02/2001";

const logLines = [
	"Boot sequence... OK",
	"Encryption matrix synced",
	"Awaiting operator credentials"
];

const titleLine = "your Production date";
const hintLine = "Enter the passcode exactly as shown in the required format.";

const typingSpeed = 50;
const lineDelay = 600;

function typeLine(text, done) {
	const line = document.createElement("div");
	line.className = "log-line";
	systemLog.appendChild(line);

	let i = 0;
	const timer = setInterval(() => {
		i++;
		line.textContent = "> " + text.slice(0, i);
		if (i >= text.length) {
			clearInterval(timer);
			done();
		}
	}, typingSpeed);
}

function playLog(index = 0) {
	if (index >= logLines.length) {
		showHintButton.classList.add("visible");
		return;
	}
	typeLine(logLines[index], () => {
		setTimeout(() => playLog(index + 1), lineDelay);
	});
}

playLog();

showHintButton.addEventListener("click", () => {
	showHintButton.disabled = true;
	typeLine(titleLine, () => {
		typeLine(hintLine, () => {
			form.classList.add("visible");
		});
	});
});

form.addEventListener("submit", (e) => {
	e.preventDefault();
	if (passcodeInput.value.trim() === requiredCode) {
		message.textContent = "Access granted.";
		message.style.color = "#a7f3d0";
	} else {
		message.textContent = "Incorrect passcode. Try again.";
		message.style.color = "#fca5a5";
	}
});


// ✅ FIXED DATE INPUT FORMATTER
passcodeInput.addEventListener("input", () => {
	let value = passcodeInput.value;

	// Allow digits and slashes only
	value = value.replace(/[^\d/]/g, "");

	// Extract digits only for formatting
	const digits = value.replace(/\D/g, "").slice(0, 8);

	let formatted = "";

	if (digits.length > 0) {
		formatted += digits.slice(0, 2);
	}
	if (digits.length >= 3) {
		formatted += "/" + digits.slice(2, 4);
	}
	if (digits.length >= 5) {
		formatted += "/" + digits.slice(4, 8);
	}

	passcodeInput.value = formatted;
});
</script>

</body>
</html>
