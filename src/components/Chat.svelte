<script>
  import { slide } from "svelte/transition";

  let expand = $state(false);
  let getText = $derived(expand ? "Ocultar" : "Obtener");
  let data = $state({ e: 0, n: 0 });

  async function getClaves() {
    const res = await fetch("http://127.0.0.1:8000/clave", {
      method: "GET",
      headers: {
        "Content-Type": "application/json",
      },
    }).then((response) => response.json());
    data = res;
    expand = !expand;
  }

  let messageCrypt = $state({});

  async function sendMessage() {
    const res = await fetch("http://127.0.0.1:8000/cifrar", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ mensaje: message.value, e: data.e, n: data.n }),
    }).then((response) => response.json());
    messageCrypt = res.cifrado;
  }
</script>

<div class="container mx-auto content-center max-w-3xl">
  <h1 class="text-3xl font-bold mb-2 text-center">Chat con algoritmo RSA</h1>
  <div class="bg-radial to-green-300/80 from-green-200/20 rounded-2xl p-4 my-4">
    <div class="flex flex-row justify-center items-center m-4">
      <button
        class="bg-green-100 p-2 rounded-xl shadow-sm cursor-pointer mx-5"
        onclick={() => getClaves()}>🔑 {getText} clave pública</button
      >
      {#if expand}
        <span id="e" class="font-semibold mx-2" transition:slide
          >e: {data.e}</span
        >
        <span id="n" class="font-semibold mx-2" transition:slide
          >n: {data.n}</span
        >
      {/if}
    </div>

    {#if messageCrypt.length}
      <div class="bg-green-100/50 p-2 rounded-lg shadow-lg mb-4">
        <p class="rounded-md bg-amber-50 inline p-2">
          {messageCrypt}
        </p>
      </div>
    {/if}

    <div class="flex flex-row gap-4 items-center">
      <input
        type="text"
        id="message"
        placeholder="Hola ..."
        class="bg-green-50 border-2 border-green-300 focus:outline-green-500 rounded-xl w-full p-2"
      />
      <button
        id="sendMessage"
        disabled={!expand}
        class="rounded-md px-3 py-1 font-semibold shadow-md {expand
          ? 'bg-green-400 cursor-pointer hover:scale-110 transition-all'
          : 'bg-green-200'} "
        onclick={() => sendMessage()}>Enviar</button
      >
    </div>
  </div>
</div>
