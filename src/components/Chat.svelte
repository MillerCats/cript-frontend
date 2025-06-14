<script>
  import { slide } from "svelte/transition";

  const userId = crypto.randomUUID();
  const socket = new WebSocket("ws://127.0.0.1:8000/ws/chat");
  let messages = $state([]); //{text: "", sent:true/false}

  socket.onopen = () => {
    console.log("Conectado al WebSocket");
  };

  socket.onmessage = (e) => {
    const data = JSON.parse(e.data);
    if (data.id !== userId) {
      messages = [...messages, { text: data.text, sent: false }];
    }
  };

  function sendMessages(msg) {
    const data = JSON.stringify({ id: userId, text: msg });
    socket.send(data);
  }
  socket.onerror = (err) => {
    console.error("WebSocket error", err);
  };

  let expand = $state(false);
  let showStates = $state([]);
  let message = $state("");
  let getText = $derived(expand ? "Ocultar" : "Obtener");
  let keys = $state({ e: 0, n: 0, d: 0 });

  async function getClaves() {
    const res = await fetch("http://127.0.0.1:8000/clave", {
      method: "GET",
      headers: {
        "Content-Type": "application/json",
      },
    }).then((response) => response.json());
    keys = res;
    expand = true;
  }

  async function sendMessage() {
    const res = await fetch("http://127.0.0.1:8000/cifrar", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        mensaje: message,
        e: keys.e,
        n: keys.n,
      }),
    }).then((response) => response.json());
    messages = [...messages, { text: res.cifrado, sent: true }];
    showStates = [...showStates, false];
    sendMessages(res.cifrado);
    message = "";
  }

  let messageDecrypt = $state([]);

  async function getMessage(index) {
    const res = await fetch("http://127.0.0.1:8000/descifrar", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        cifrado: messages[index].text,
        d: keys.d,
        n: keys.n,
      }),
    }).then((response) => response.json());
    messageDecrypt[index] = res.mensaje;
    showStates[index] = true;
  }
</script>

<div class="container mx-auto content-center max-w-3xl">
  <h1 class="text-3xl font-bold mb-2 text-center">Chat con algoritmo RSA</h1>
  <div class="bg-radial to-green-300/80 from-green-200/20 rounded-2xl p-4 my-4">
    <div class="flex flex-row justify-center items-center m-4">
      <button
        class="bg-green-100 p-2 rounded-xl shadow-sm cursor-pointer mx-5"
        onclick={() => {
          !expand ? getClaves() : (expand = false);
        }}>🔑 {getText} clave pública</button
      >
      {#if expand}
        <span id="e" class="font-semibold mx-2" transition:slide
          >e: {keys.e}</span
        >
        <span id="n" class="font-semibold mx-2" transition:slide
          >n: {keys.n}</span
        >
      {/if}
    </div>

    {#if messages.length}
      <div
        class="bg-green-100/50 p-2 rounded-lg shadow-lg mb-4 flex flex-col"
        transition:slide
      >
        {#each messages as msg, index}
          <div
            class="flex flex-col max-w-fit my-2 transition-all
            {msg.sent ? 'self-end' : 'self-start'} "
          >
            <button
              class="bg-amber-50 p-2 cursor-pointer hover:scale-105 font-semibold
              {showStates[index] ? 'rounded-t-md' : 'rounded-md'}"
              onclick={() => {
                !showStates[index]
                  ? getMessage(index)
                  : (showStates[index] = false);
              }}
            >
              {msg.text}
            </button>
            {#if showStates[index]}
              <p
                class="bg-gray-50 p-1 rounded-b-md text-sm font-bold text-end"
                transition:slide
              >
                {messageDecrypt[index]}
              </p>
            {/if}
          </div>
        {/each}
      </div>
    {/if}

    <div class="flex flex-row gap-4 items-center">
      <input
        type="text"
        id="message"
        bind:value={message}
        autocomplete="off"
        placeholder="Hola ..."
        class="bg-green-50 border-2 border-green-300 focus:outline-green-500 rounded-xl w-full p-2"
      />
      <button
        id="sendMessage"
        disabled={!expand || message.trim() === ""}
        class="rounded-md px-3 py-1 font-semibold shadow-md
        {expand && message.trim() !== ''
          ? 'bg-green-400 cursor-pointer hover:scale-110 transition-all'
          : 'bg-green-200'} "
        onclick={() => sendMessage()}>Enviar</button
      >
    </div>
  </div>
</div>
